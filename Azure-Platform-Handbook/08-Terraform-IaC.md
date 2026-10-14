# 08 — Terraform and infrastructure as code

> **Evidence convention:** `[Live]` is the Azure CLI inventory collected 2026-10-13; `[Code]` is
> repository/HCL/YAML inspection; `[Chat]` is historical troubleshooting evidence; `[Doc]` is the
> architecture or governance document. HCL is desired state—not proof of live configuration. Secret
> values and credentials are deliberately excluded.

## TL;DR

- `[Code]` Three workload roots implement Terraform: `EC_Shared_Resources_IaC/src/01-iac`,
`EC_OrderAutomation_IaC/src/01-iac`, and `EC_AssistantsFactory_IaC/src/01-iac`.
`EC_Terraform_Modules` holds reusable modules. Shared and Order Automation roots declare no module
sources; Assistant composes local and Git-sourced modules.
- `[Live]` The active subscription inventory includes separate shared and Order Automation resource
groups for dev, qa, and prod, plus Assistant Factory dev/test. The environment suffix does **not**
mean separate Azure subscriptions: all CAIRO environments were found in `BASF_SC_CAIRO_PI_PROD`.
- `[Code]` Remote state is Azure Blob, initialized by each repo's `src/tf.ps1`; storage is composed
as `tf${Prefix}${EnvironmentName}`, container `tf-state`. Root keys are `ec-shared.tfstate`,
`ec-orderauto.tfstate`, and `ec-assistfactory.tfstate`.
- `[Chat]` Highest-risk IaC lessons: import/adopt rather than recreate, read plans carefully,
reconcile subnet/policy constraints atomically, do not assume private endpoint means private-only,
and protect secrets in state. Portal changes can be lost where Terraform owns a setting, or become
invisible drift where lifecycle ignores it.

## 1. Repositories and ownership boundaries

| Repository | Root / code location | What the Terraform graph owns | What it does not establish |
|---|---|---|---|
| `EC_Shared_Resources_IaC` | `src/01-iac` | Shared VNet/NSGs/private DNS, APIM/API Center, shared KV, Front Door/WAF, ML workspace supporting resources and endpoints. | Active ADLS/Databricks resources are commented out; no app APIs/policies, no app-specific databases. `[Code]` |
| `EC_OrderAutomation_IaC` | `src/01-iac` | App RG, shared-VNet app subnet, App Services/plan, Function, Logic App shells, Event Grid, storage, SQL/PostgreSQL, KV, AFD/WAF, LAW/diagnostics. | Does not deploy container images or ARM workflow definitions; CD/repo/manual work does that. `[Code]` |
| `EC_Terraform_Modules` | One module per `<module>/<module>` | Reusable individual Azure resource patterns, not an integrated environment root. | Neither shared nor Order Automation root imports these modules. Assistant Git module refs are not pinned. `[Code]` |
| `EC_AssistantsFactory_IaC` | `src/01-iac` | Assistant API/MCP App Services, VNet integrations/subnets, Front Door/WAF, Cosmos, KV, ADLS, Speech, PostgreSQL, App Insights and PEs. | Current root graph omits several declared AI/Bot/APIM/ACR/Search/Databricks inputs. `[Code]` |

### 1.1 Naming, tags, and environment inputs

- Shared and Order Automation use `local.name_prefix = lower("${var.workload}-${var.environment}")`;
compact forms remove punctuation and truncate. Assistant uses its own service naming values.
`[Code]`
- Shared and Order Automation common tags include `Environment`, `Owner`, `CostCenter`, and
`ManagedBy = Terraform`; resource-specific tags are merged. `[Code]`
- Dev examples: `rg-sharedresources-dev`, `vnet-sharedresources-dev`, `kv-sharedresources-dev`,
`apim-sharedresources-dev`, `mlw-sharedresources-dev`; Order Automation RG
`rg-order_automation-dev`. `[Code][Live]`
- Shared VNet is `10.0.0.0/16`; shared subnets include APIM `10.0.0.0/24`, MLOps `10.0.1.0/24`,
Order Automation `10.0.2.0/24`, and private endpoints `10.0.3.0/24`. `[Code]`
- Assistant dev uses `vnet-sharedresources-dev` and dedicated ranges including App `10.0.5.0/24`,
MCP `10.0.6.0/24`, and PE subnets in `10.0.7.0/24`. QA tfvars point to the **dev** shared VNet;
validate before apply. `[Code]`
- Only dev tfvars were present in the checked Shared and Order Automation roots, while their QA/PROD
pipeline YAML refers to `vars_qa.tfvars` and `vars_prod.tfvars`. Assistant has all three. Confirm
exact branch/artifact contents before dispatch. `[Code]`
- The Order Automation subnet resource hardcodes `snet-order_automation-dev` / `10.0.2.0/24`; this
is not an environment parameter despite multiple env pipelines. Shared code already documents the
range as occupied by that subnet. Do not let two independent states claim it. `[Code][Live]`

## 2. Shared platform root: file-by-file

Root: `EC_Shared_Resources_IaC/src/01-iac`; orchestration: `src/tf.ps1`.

| File | Root-level purpose / active resources |
|---|---|
| `resourcegroup.tf` | `azurerm_resource_group.RG`, `rg-${local.name_prefix}`. |
| `locals.tf` | Prefix/compact naming, tags, private-DNS map, PE subnet ID. |
| `variables.tf` | Environment/location/workload/owner/cost; VNet/subnets; APIM/API Center; MLOps; Databricks/ADLS/mount; AFD; KV RBAC/IP-list toggles. |
| `backend.tf` | AzureRM backend key `ec-shared.tfstate`; backend account/RG/container supplied by wrapper. |
| `providers.tf` | Terraform constraints, AzureRM/AzAPI/Random, client config. Databricks provider block commented. |
| `vnet.tf` | VNet, NSGs, common inbound deny, APIM rules/moved addresses, subnet AzAPI resources. |
| `privatedns.tf` | Private DNS zones and VNet links. |
| `pvtendpoint.tf` | Comments documenting placement/conventions only; active PEs are service-specific files. |
| `apim.tf` | Optional API Management, system MI, optional Key Vault RBAC assignment. |
| `apicenter.tf` | Preview API Center via AzAPI, system MI. |
| `frontdoor.tf` | AFD profile, endpoint, origin group/origin/route, WAF and security policy. |
| `azurepolicy.tf` | CIS/ISO assignment code is commented; no active policy assignment in this root. |
| `keyvault.tf` | RBAC KV, KV PE and DNS-zone association. No secret resources. |
| `mlworkspace.tf` | Optional ML workspace, supporting storage/ACR/App Insights, PEs, optional KV grant. |
| `adlsstorage.tf` | ADLS account, Gen2 filesystem, DFS PE commented out. |
| `adlsmount.tf` | Databricks ADLS mount commented out. |
| `databricks.tf` | Workspace/config/groups/SP/users/membership blocks commented out. |
| `databrickscluster.tf` | Cluster and permissions commented out. |
| `imports.tf` | One-time import of APIM/MLOps subnets and removal of legacy Terraform addresses without Azure destroy. |
| `outputs.tf` | RG/VNet/subnets/DNS/KV/ML/APIM/API Center/AFD. ADLS/Databricks outputs commented. |

### 2.1 Shared environment/service details

- `[Code]` Terraform `>=1.5.0`, AzureRM `~>3.100`, AzAPI `~>2.0`, Random `~>3.6`; Databricks
`~>1.35` is declared but its provider is commented. The dev shared root enables APIM Developer_1
External, API Center, MLOps resources, KV, Front Door.
- `[Live]` Subscription resource group `rg-sharedresources-dev` contains `apim-sharedresources-dev`,
`kv-sharedresources-dev`, `mlw-sharedresources-dev`, `acrsharedresourced2slfk`,
`aisearch-sharedresources-dev`, AI Hub/project resources and shared endpoints. Live API Center is
`apic-mcp-dev`; compare HCL/API Center naming before claiming the same owner.
- `[Code]` Shared KV is RBAC-authorized, purge-protected, soft delete 90 days, public network
disabled, default-deny with AzureServices bypass and private endpoint. No secrets are created
here.
- `[Code]` Shared ML workspace public access is disabled in dev; supporting storage public access
disabled; ACR public network enabled despite private endpoint. Endpoint is additive, not a
firewall switch.
- `[Code]` APIM external VNet mode exposes gateway 443 from Internet in dev; management public
access is configured true in tfvars. NSG includes 3443 from ApiManagement, 6390 from
AzureLoadBalancer, and outbound platform dependencies. API products/operations/policies are not
declared in this root.
- `[Code]` The shared root's ADLS/Databricks blocks and Azure Policy assignments are inactive. A
commented sample of intended Databricks resources is not evidence of provisioned resources.
`[Live]` Inventory shows AI workspace(s), storage, and Search, but source ownership must be
checked independently.

### 2.2 Shared import/adoption and drift record

`imports.tf` imports `snet-apim` (`10.0.0.0/24`) and `snet-mlops` (`10.0.1.0/24`) under
`azapi_resource.subnet[...]`. `removed` blocks forget previous `azurerm_subnet.subnet` and
subnet/NSG association state addresses with `destroy = false`. Source comments describe it as
one-time; validate that plan has no subnet destroy/replace, then remove transient import
declarations as appropriate. `[Code]`

- `[Chat][Code]` APIM went through adopt/import/recreate iterations. Recent Shared history includes
adoption, public recreation, import and subnet reconciliation commits. The correct response to an
existing service is an explicit import/state ownership plan, not a casual rename or replacement.
- `[Chat]` APIM activation stalled in Internal mode; NSG service-management/probe/dependency rules
and reachable VNet/DNS were investigated. Current checked HCL says External—historical intent and
current configuration differ; confirm live mode.
- `[Chat]` Terraform subnet policy failures showed the NSG must be attached in the atomic subnet
create/update payload; adding NSG later is too late under the policy. Use AzAPI subnet resource
carrying NSG ID.
- `[Chat]` Concurrent APIM/VNet changes generated 409 subnet update conflicts. Sequence
infrastructure operations and avoid parallel mutation of one VNet/subnet.

## 3. Order Automation root: file-by-file

Root: `EC_OrderAutomation_IaC/src/01-iac`; wrapper `src/tf.ps1`; separate `src/03-iac` scripts.

| File | Root-level purpose / active resources |
|---|---|
| `resourcegroup.tf` | Workload resource group. |
| `locals.tf` | Naming, tags, DNS map, PE subnet ID. |
| `variables.tf` | Environment/network/DNS, apps, Function, SQL/PostgreSQL, LAW/Monitor, storage/ADLS, Event Grid, KV, Logic Apps, App Insights, static site/AFD inputs. |
| `backend.tf` | AzureRM key `ec-orderauto.tfstate`. |
| `providers.tf` | Terraform/AzureRM/AzAPI/client-config; Databricks provider commented. |
| `subnet.tf` | Reads shared VNet and optional existing NSG; creates hard-coded app subnet via AzAPI. |
| `privatedns.tf` | App DNS zones and optional VNet links (default false). |
| `adlsstorage.tf` | ADLS account/filesystems and DFS PE; network-ACL block commented. |
| `blobstorage.tf` | Optional StorageV2 account, private containers, blob PE. |
| `staticstorage.tf` | Optional StorageV2 static-website account/configuration. |
| `appserviceplan.tf` | Optional Linux plan. |
| `extractionappservice.tf` | Extraction container App Service, settings, system/user identity, conditional FD source restriction. |
| `orchestratorapiappservice.tf` | Orchestrator container App Service. |
| `rulesengineappservice.tf` | Rules container App Service. |
| `apimintegrationtestbackendappservice.tf` | Test backend App Service. |
| `appserviceidentity.tf` | ACR lookup, UAMI, AcrPull grants and App Service MI-pull AzAPI config. |
| `functionapp.tf` | Random runtime-storage suffix, Linux EP plan and Function; optional `sites` PE. |
| `eventgrid.tf` | Optional Event Grid topic and optional topic PE. |
| `logicapp.tf` | Main workflow shell, DEV test HTTP trigger, email-to-bug shell and App Insights. Workflow definitions/connections deployed separately. |
| `keyvault.tf` | Optional RBAC KV/PE, secret-name stubs, SQL/PostgreSQL password data lookups. |
| `postgresqlserver.tf` | Flexible Server/database/PE. |
| `sqldb.tf` | SQL Server/database, Azure-services firewall exception, optional SQL PE. |
| `loganalytics.tf` | Optional Log Analytics workspace. |
| `monitor.tf` | Optional email action group and diagnostic settings for selected App Service, Function, SQL sources. |
| `frontdoor.tf` | AFD profile/endpoint, static/extraction/orchestrator/rules origins/routes, CORS response headers, WAF/security policy. |
| `outputs.tf` | RG/network/KV/storage/apps/SQL/LAW/AFD hostnames and selected IDs; no Logic App/Event Grid/PostgreSQL IDs. |

### 3.1 Order Automation design and limits

- `[Code]` Provider constraints align with Shared: AzureRM `~>3.100`, AzAPI `~>2.0`, Random `~>3.6`;
Databricks provider is disabled. No reusable external modules are called.
- `[Code][Live]` Primary app names include `extract-orderautomatio`,
`orchestrator-api-appservice-dev`, `rules-engine-appservice-dev`; plans include
`appserviceplan-dev`; app UAMI `uami-order_automation-dev` is used for ACR pull. Main app services
also have SystemAssigned identities.
- `[Live]` `rg-order_automation-dev` has five PEs: ADLS DFS, blob, Event Grid, Key Vault and
PostgreSQL. The presence of a PE does not prove public access has been turned off.
- `[Code]` The dev app plan SKU is B3; Function plan EP1; PostgreSQL Flexible Server 15 /
GP_Standard_D2s_v3 / 65,536 MB with public access true in dev; SQL Database S0, up to 5 GB. Admin
passwords are data-read from Key Vault (`sql-admin-password`, `postgresqlpassword`), not tfvars
plaintext.
- `[Code]` ADLS `dlsecorderautodev` filesystems `land`, `processing`, `output`, `report`; blob
containers are same names and private. DFS PE is active but ADLS firewall network_rules are
commented. SQL's `AllowAzureServices` 0.0.0.0 rule is a public Azure-services path; SQL PE is
optional and false in dev.
- `[Code]` Diagnostics route extraction App Service, Function, SQL server and SQL database
categories/metrics to LAW. Orchestrator, rules, Logic Apps, KV, PostgreSQL, Event Grid, Storage,
Front Door/WAF diagnostics are not declared in `monitor.tf`.
- `[Code]` AFD Standard, WAF Prevention, routes `/*` static, `/extraction/*`, `/api/*`, `/rules/*`;
HTTPS redirect and HTTPS origin. CORS overwrites `Access-Control-Allow-Origin` to the AFD default
hostname and allows configured methods/headers, no credentials; not all routes use the rule set.
- `[Code]` KV is RBAC, purge protection, 90-day soft-delete; public access enabled with default-deny
ACL/IP allowlist and optional PE. Terraform ignores KV `network_acls`; manual firewall allowlist
changes may persist as unmanaged state.
- `[Code]` Lifecycle ignores app settings and some app restrictions/image/identity settings because
app CD owns them. If app CD does not update the same properties, this becomes unmanaged
configuration.

## 4. Reusable modules: `EC_Terraform_Modules`

`[Code]` Module repo layout is `<Module>/<Module>/`; modules do not form a deployment root. No fixed
`?ref=` version pin appears in Assistant Git module URLs. Typographic filenames `varaibles.tf` and
singular `variable.tf` are present in source.

| Module directory | Primary purpose / implementation note |
|---|---|
| `APIManagement/APIManagement` | API Management. |
| `ApplicationInsights/ApplicationInsights`, `ApplicationInsightsKey/ApplicationInsightsKey` | App Insights and API-key resource (sensitive value excluded). |
| `AppService/AppService`, `AppServicePlan/AppServicePlan`, `AppServiceVnetIntegration/AppServiceVnetIntegration` | Legacy App Service container resource, plan and Swift connection; app module supports system MI and optional AFD service-tag/header restriction, ignores settings. |
| `CognitiveSearch/CognitiveSearch`, `CommunicationService/CommunicationService` | Search service and Communication resource. |
| `CosmosDBAccount/CosmosDBAccount`, `CosmosDBMongoDatabase/CosmosDBMongoDatabase` | Cosmos Mongo account/database. Account has static IP range configuration and ignores its drift; validate effective network filters. |
| `Databricks/Databricks`, `DatabricksCluster/DatabricksCluster`, `DatabricksMount/DatabricksMount` | Workspace/users/groups, cluster/permissions and ADLS Gen2 mount. |
| `Datalakestorageg2/Datalakestorageg2` | StorageV2/HNS/filesystem; network rules commented. |
| `DNS/DNS`, `DNSVNETLink/DNSVNETLink`, `Privatelink/Privatelink` | Private zones, VNet links and PEs/optional DNS group. |
| `FormRecognizer/FormRecognizer`, `SpeechService/SpeechService`, `TranslationService/TranslationService` | Cognitive accounts by modality. |
| `KeyVault/KeyVault`, `KV/KV`, `KeyVaultSecrets/KeyVaultSecrets` | Legacy access-policy Key Vault modules (secret code commented) and secret lookup module. RBAC/access policy semantics must be checked before reuse. |
| `MLStudio/MLStudio` | Azure ML workspace. |
| `PostgresqlFlexibleServer/PostgresqlFlexibleServer`, `PostgresqlFlexibleServerDatabase/PostgresqlFlexibleServerDatabase` | Flexible Server/AD administrator and database. Password supplied by input; random password appears unused. |
| `Resourcegroup/Resourcegroup` | Resource group. |
| `SQLDB/SQLDB`, `SQLServer/SQLServer` | Legacy `azurerm_sql_database` and MSSQL server. |
| `Subnet/Subnet`, `Virtualnetwork/Virtualnetwork`, `Vnet/Vnet` | Subnet, network with DBR subnet/NSGs/peerings, or VNet only. Module subnet pattern is distinct from Assistant's local AzAPI subnet module. |

Assistant uses remote source URLs pointing to `basf4EMEA/CAIRO/_git/EC_Terraform_Modules//...`
without commit/tag refs, so a default-branch change can affect later plans. `[Code]`

## 5. Assistant Factory Terraform root

- `[Code]` Files: `backend.tf`, `provider.tf`, `main.tf`, `variables.tf`, `vars_dev.tfvars`,
`vars_qa.tfvars`, `vars_prod.tfvars`. Local modules: `FrontDoor`, `NetworkSecurityGroup`, `Subnet`
(AzAPI), `WebApplicationFirewall`. No root `imports.tf` or `outputs.tf` was found.
- `[Code]` Provider requirements: AzureRM `~>4.0`, AzAPI `~>2.0`, Random `~>2.1.2`;
`resource_provider_registrations = "none"`. This AzureRM 4.x root is distinct from the other
roots' 3.x baseline; legacy shared modules may not be compatible without test plans.
- `[Code]` Root module graph: resource group, main/MCP Linux plans and App Services, two Swift
integrations, local Front Door/WAF, shared NSG, Cosmos account/Mongo DB, Key Vault and secret
lookups, DNS zones/links, Cosmos/KV/ADLS/Speech/Postgres PEs, ADLS account, Speech account, App
Insights/key, PostgreSQL server/database.
- `[Code]` PE connections use manual approval. Confirm each connection from the target service
Networking/Private endpoint connections blade; Terraform request success does not mean service
owner approved it.
- `[Code]` Current graph does not call modules for Bot, OpenAI, AI Hub/Foundry, ACR creation, APIM,
Search, Form Recognizer, Translator, Databricks, SQL, or general `Virtualnetwork`, despite related
declared variables. Do not infer provisioning from a tfvars entry.
- `[Code]` Assistant Storage/ADLS module has no active network_rules; its shared KV module receives
RBAC input while still declaring access policies. Speech root pins West Europe even if tfvars
location says North Europe; compare live region.
- `[Code][Live]` Dev RG `rg-assistant_factory-dev`; test RG `rg-assistant_factory-test`. The
inventory's permission-scope limitations and lack of platform-created root outputs matter when
reconciling pipeline outputs.

## 6. State, plan and execution mechanics

### 6.1 Backend map

| Root | Backend key | Runtime backend storage naming | Container |
|---|---|---|---|
| Shared | `ec-shared.tfstate` | `tf${Prefix}${EnvironmentName}` | `tf-state` |
| Order Automation | `ec-orderauto.tfstate` | `tf${Prefix}${EnvironmentName}` | `tf-state` |
| Assistant Factory | `ec-assistfactory.tfstate` | `tf${Prefix}${EnvironmentName}` | `tf-state` |

`[Code]` `backend.tf` supplies the key; `tf.ps1` supplies backend resource group, account and
container dynamically. The live inventory saw utility groups such as `tf_state_shared_dev`,
`tf_state_orderautomation_qa`, `tf_state_shared_prod`, and `tf_state_orderautomation_prod`. Do not
assume an account name or key from a group name; inspect actual pipeline parameters and storage
account. `[Live][Code]`

### 6.2 `src/tf.ps1` wrapper

`[Code]` Wrapper switches include `Init`, `Plan`, `Apply`, `Output`, `Destroy`, `Validate`,
`Version`, `DownloadTerraform`, `UseExistingTerraformPlan`, `Force`, `ReplaceDBRProvider`, and
firewall controls. It can download/verify Terraform, map Azure CLI identity to provider environment,
initialize backend, run commands in selected Terraform root and remove local `.terraform` artifacts.

1. Resolve prefix, environment, region, utility RG, Terraform source path and tfvars from pipeline
variables.
2. Use Azure CLI context/service connection to ensure state RG/account/container; state storage uses
Standard_LRS / BlobStorage / Hot, HTTPS-only, TLS 1.2, private blob access disabled in wrapper
logic.
3. Retrieve backend authentication material for `terraform init` without surfacing its value in this
handbook.
4. Create `terraform.tfplan` in plan mode; upload plan artifact; apply can consume that exact plan
using `-UseExistingTerraformPlan`.
5. Output mode emits `output.json` from `terraform output -json`; treat as restricted because output
payload/state can contain sensitive material.
6. Optional provider replacement rewrites state source address from `databrickslabs/databricks` to
`databricks/databricks`; entry YAML passes this option even though current shared/app Databricks
blocks are commented.

`[Code][Chat]` Terraform `sensitive` affects display, not whether secret material enters state. The
two Key Vault password data lookups can persist values in remote state. Restrict backend access,
artifact access and retention; plan/output artifacts are security-sensitive.

### 6.3 Import, drift and safe reconciliation

- Before import: verify subscription, resource ID, owning stack/state, provider address, config
equivalence and backup state. Import changes Terraform tracking, not Azure configuration.
- Run `terraform plan` and inspect for forced replacement, `destroy`, dependency ordering, policy
failures, and sensitive values before approve.
- State `rm` removes Terraform ownership but **does not** remove or secure the Azure resource; use
only as audited ownership handoff, with named new owner.
- `[Chat]` APIM has repeated import/adoption/recreation. Confirm live SKU, VNet mode/subnet/public
access, state and other APIs before any apply.
- `[Chat]` Shared subnet imports reconcile AzAPI and legacy azurerm addresses; do not leave both
providers managing the same subnet.
- `[Chat]` Key Vault re-creation collided with purge-protection/soft-delete and globally unique name
retention. Check active and soft-deleted vaults before recreating; do not use a new name casually
when consumers/DNS depend on the old name.
- `[Chat]` ACR pull drift happened when app CD wrote registry username/password and reset
`acrUseManagedIdentityCreds`. Decide one owner (prefer MI + AcrPull), then ensure both Terraform
and application pipeline preserve that model.
- `[Chat]` Secrets moved out of plaintext tfvars into Key Vault data references. Keep values out of
Git, but remember Terraform state still contains data-source read values.
- `[Code]` Lifecycle `ignore_changes` is common on app settings, KV ACLs, app restrictions, WAF
custom rules and static Cosmos IP filters. It suppresses reconciliation; document who owns each
ignored setting and verify live config.

## 7. What is intentionally outside Terraform

| Activity | Evidence / implications |
|---|---|
| App image build/push and runtime image release | Separate BE CD builds and pushes to ACR then configures Web Apps. Terraform does not produce the image. `[Code]` |
| Logic App workflows and connector authorization | Terraform creates workflow shells; ARM JSON is deployed by app pipeline; Office 365 connections/mailbox grants are authorized manually. `[Code][Chat]` |
| ADLS directories/ACLs | `src/03-iac` wrappers invoke `AdlsPermissions.ps1` with Azure CLI key auth to create paths and recursively set POSIX ACLs. These mutate data-plane permissions outside Terraform. `[Code]` |
| PostgreSQL extension allowlisting | `pgcrypto`, `pg_trgm`, `fuzzystrmatch` require per-server configuration, repeated for each environment. `[Doc][Chat]` |
| Key Vault secret values/firewall exceptions | Secret values are manually provisioned/rotated; firewall exceptions have been adjusted for hosted-agent egress. `[Code][Chat]` |
| Private endpoint approvals | Assistant PEs use manual approval; target resource owner action required. `[Code]` |
| ADO role assignments / service connections / variable groups / Environment gates | Project permissions and protected variable-group values live in ADO settings, not the Terraform source. `[Code]` |
| Databricks workspace Git integration, repo-folder, ACL and job/notebook migration | Git sync pipeline updates a workspace repo; credentials/entitlement/folder setup is manual. It does not create notebook jobs or prove execution. `[Code][Chat]` |
| DNS changes, frontend CORS fixes and selected app settings | Some were applied via Portal/CLI and have later drifted; verify exact IaC ownership/ignore rules. `[Chat]` |
| Azure Policy/Defender subscription governance | Shared Azure Policy assignments are commented; policy recommendations are advisory, and Defender changes were reverted historically. Check subscription directly. `[Code][Doc][Chat]` |

## Where to look in Azure Portal / ADO

- Azure Portal → **Subscriptions** → `BASF_SC_CAIRO_PI_PROD` → Resource groups
(`rg-sharedresources-*`, `rg-order_automation-*`, `rg-assistant_factory-*`, `tf_state_*`).
- Azure Portal → **Storage accounts** → Terraform state account → Containers → `tf-state`; restrict
access and never download state casually.
- Azure Portal → VNet `vnet-sharedresources-dev` → Subnets / NSGs / Private DNS zones / Private
endpoints; compare address allocation, links and approvals.
- Azure Portal → Resource → **Activity log**; identify portal/CD actor for unexpected property
changes.
- Azure DevOps `basf4EMEA` → CAIRO → Repos → each IaC repo `src/01-iac`, `src/tf.ps1`,
`.azuredevops`, `src/03-iac`.
- Azure DevOps → Pipelines → Run → plan artifact, apply log and output artifact; ADO Library →
Variable groups; Project settings → Service connections / Agent pools / Environments.
- Azure Portal → Key Vault → Networking, Access configuration and IAM; do not confuse control-plane
Contributor with secret data-plane access.

## Interview talking points

- Explain the difference among configuration, state, actual Azure resource and pipeline artifact;
none alone proves end-to-end correctness.
- Describe import as adopting a resource into state without recreating it, followed by a reviewed
reconciliation plan.
- Explain atomic subnet + NSG creation under deny policy, and why a post-create NSG attachment
failed.
- Explain why PE and public network access are independent; inspect DNS, route, target subresource,
approval and public flag.
- Explain Azure Blob backend key isolation and why secrets read with a data source still become
state risk.
- Explain provider-major mismatch and unpinned module refs as reproducibility risk.
- Distinguish HCL provisioning from manual workflow authorization, ADLS ACLs, DB extensions and app
image deployment.
- Explain `ignore_changes` as explicit ownership delegation, not a cure for drift.

## Evidence & confidence

- **High:** file inventories, provider constraints, active/commented Terraform resources, backend
keys, plan wrapper flow—`raw_iac.md`, checked-out code. `[Code]`
- **High for collection timestamp/scope:** resource group/type inventory from `raw_azure.md`,
collected 2026-10-13. `[Live]`
- **Medium:** recurring incident history from `raw_chats.md`; historical changes may later be
reverted. `[Chat]`
- **Medium:** architecture overview states Terraform provisions all infra; specific HCL inspection
proves that statement is broader than current checked active graph. `[Doc][Code]`
- Azure DevOps direct listing was denied (`TF400813`); pipeline details are YAML-derived, not a live
registration inventory. `[Live][Code]`

## Further reading

- [03 Networking](03-Networking.md) · [04 Identity, security and
governance](04-Identity-Security-Governance.md) · [09 CI/CD](09-CICD-Azure-DevOps.md) · [11 Manual
steps and runbooks](11-Manual-Steps-and-Runbooks.md) · [14 Resource register and
glossary](14-Resource-Register-and-Glossary.md)

## Appendix A — Root-level ownership and configuration review worksheet

Use this worksheet before planning a change. It is intentionally framed around reviewed sources, not
assumptions.

### A.1 Shared platform review

- Confirm root path: `EC_Shared_Resources_IaC/src/01-iac`.
- Confirm backend key: `ec-shared.tfstate`.
- Confirm provider major version: AzureRM 3.x (`~>3.100`).
- Confirm current Azure CLI subscription and deployment principal.
- Check shared VNet owner before changing subnet/DNS resources.
- Check import declarations before planning subnet changes.
- Compare `vars_dev.tfvars` with current live values.
- Do not reactivate commented Databricks/ADLS resources using TODO placeholders.
- Check APIM live Virtual Network mode before changing `virtual_network_type`.
- Check APIM SKU and service lifetime/timeout because provisioning may be lengthy.
- Check APIM subnet NSG dependencies and public management-plane access separately.
- Check shared Key Vault data-plane RBAC, public network access, ACL bypass and PE.
- Confirm optional `manage_kv_rbac` is false/true intentionally and principal may assign role.
- Check MLOps storage and ACR network flags independently from private endpoints.
- Confirm ML workspace role grants and diagnostics; no complete diagnostic graph is declared.
- Review Front Door origin hostname—the dev source uses TODO hostname placeholder.
- Review WAF policy association and custom rule drift behavior.
- Check active `azurepolicy.tf` resources; current blocks are comments.
- Read `outputs.tf` and avoid publishing sensitive/resource data broadly.

### A.2 Order Automation review

- Confirm root path: `EC_OrderAutomation_IaC/src/01-iac`.
- Confirm backend key: `ec-orderauto.tfstate`.
- Confirm AzureRM 3.x and AzAPI version constraints.
- Confirm the shared VNet ID and owner state before subnet plan.
- Confirm `snet-order_automation-dev` name and `10.0.2.0/24` are not duplicated.
- Check `create_private_dns_vnet_links`; default false can leave zones unlinked.
- Verify KV `network_acls` is ignored; change requires an explicit owner/reconcile action.
- Confirm `manage_secrets` is disabled/enabled intentionally; secret stubs aren't secret values.
- Verify SQL/PostgreSQL KV secret data lookups and state access policy.
- Check PostgreSQL public access and PE are independent flags.
- Check SQL Azure-services firewall exception scope and whether PE is active.
- Confirm ADLS public firewall; DFS PE is additive where network rules are commented.
- Confirm Event Grid public network policy and topic PE separately.
- Confirm `sites` PE for Function is enabled if required; dev input says false.
- Review app-settings ignore list and designate app pipeline owner.
- Verify ACR MI ID and grant on every distinct registry used in image settings.
- Compare app container port with Dockerfile/service listener for each service.
- Check WAF `prevent_destroy` and ignored custom-rule change behavior.
- Confirm CORS origin list matches deployed custom/default AFD domain and methods.
- Verify diagnostic setting sources beyond the subset declared in `monitor.tf`.

### A.3 Assistant root review

- Confirm root path and `ec-assistfactory.tfstate` key.
- Confirm AzureRM 4.x compatibility for every remote module and provider lock.
- Confirm Git module URLs resolve to reviewed immutable refs; current links lack `?ref=`.
- Check whether remote module repo access works with the pipeline identity.
- Verify tfvars subnet ranges against the actual selected shared VNet.
- Confirm QA VNet values do not inadvertently target DEV.
- Check Key Vault zone suffix and link against live resolver behavior.
- Check KV RBAC flag versus module's legacy access-policy resources.
- Check all manual PE connection states after Terraform requests them.
- Confirm shared DNS data-source lookup uses intended environment resource group.
- Verify Speech resource region; root module calls West Europe.
- Confirm AFD/WAF FDID header matches live endpoint and backend restriction.
- Check MCP service is not assumed to be an AFD origin; current graph omits it.
- Treat unused AI/Bot variables as inert until a module call is in root graph.
- Check `terraform output` expectations; child module outputs do not become root outputs.

## Appendix B — Import and drift decision tree

1. **Is Azure object already present?** Find by resource ID and inspect activity/deployment history.
2. **Does a Terraform state already own it?** Check exact backend/container/key and address with
approved state process.
3. **Is another repo/root the owner?** Stop if the resource has multiple owners; ask platform owner
for handoff.
4. **Does provider config support import?** Verify resource type and imported ID shape; test in a
controlled worktree.
5. **Would any immutable property force replacement?** Compare name, region, SKU, subnet, public
access and service-level mode.
6. **Does org policy require atomic properties?** For subnet, include NSG in initial request.
7. **Can dependent consumers tolerate replacement?** Include DNS, clients, secret references, AFD
origins and private endpoints.
8. **Import/adopt.** Make state change in the approved workflow; do not confuse state import with
Azure create/update.
9. **Plan.** Review all `create`, `update`, `delete`, `replace`, `moved`, and `removed` actions.
10. **Apply.** Use exact approved plan and target environment.
11. **Verify.** Check state address plus live resource, DNS, health and consumer path.
12. **Close temporary migration code.** Remove one-time imports only when source comments/process
call for it and state is stable.

## Appendix C — Variables and pipeline inputs that need preflight

`[Code]` The wrapper receives infrastructure inputs from ADO pipeline/variable groups. Names can
differ by repository and case; a successful parameter-shape check does not prove that values point
to correct resource IDs.

- `environmentName`: compare with selected pipeline YAML and tfvars.
- `prefix`: verify naming composition and Azure global name uniqueness.
- `region` / `location`: compare against existing resource region and service availability.
- `tfPath` / source path: ensure intended `src/01-iac` root is selected.
- `tfvarsFile`: verify file exists in selected branch and is the proper environment file.
- `utilResourceGroupName`: verify state backend placement and access.
- `AzureServiceConnectionName_<env>`: verify ADO project connection and subscription assignment.
- Key Vault IP/firewall list: compare with agent model and do not expose addresses unnecessarily.
- Shared DNS/VNet IDs: check subscription/resource-group/name path and environment.
- AFD prefix/origin/FDID: compare live endpoint/origin, not TODO or stale branch values.
- ACR registry and image names: separate pipeline `AcrPush` from runtime identity `AcrPull`.
- Identity object/client IDs: verify correct principal and audience without printing secret values.

## Appendix D — State security checklist

- Apply least-privilege access at storage account/container scope where supported.
- Enable/enforce storage encryption and secure transport; verify controls in live account.
- Protect backend keys and leases; use approved identity authentication where supported.
- Limit state downloads to IaC maintainers and pipeline service principal.
- Restrict plan/output artifacts; set retention and build access controls.
- Never attach `terraform.tfstate`, `output.json`, plan JSON or raw app settings to general tickets.
- Treat data-source secrets as material in state even when CLI marks output sensitive.
- Rotate any credential exposed historically and audit old state versions/backups.
- Review logs for secret interpolation/command echo.
- Document recovery/restore process for state and lock ownership.
