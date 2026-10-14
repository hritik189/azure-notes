# 11 — Manual work and operational runbooks

> **Safety:** commands below are diagnostic/read-only unless explicitly marked as a controlled
> change. Use exact environment, subscription and resource IDs; do not paste secrets, keys,
> connection strings, SAS URLs or output JSON into tickets. `[Live]` = inventory evidence; `[Code]`
> = repo config; `[Chat]` = historical incident; `[Doc]` = official overview.

## TL;DR

Terraform is only one operator in this platform. Important manual/post-deployment tasks include role
grants and service-connection authorization, Key Vault secret values/firewall handling, Office 365
connector consent/mailbox delegation, PostgreSQL extension allowlisting, private-endpoint approval,
Databricks Git integration, ADLS directory ACLs, app CORS/identity settings, and ADO Environment
approvals. A successful Terraform or ARM deployment does not mean these external dependencies are
configured. `[Code][Chat]`

Before touching an environment: capture target subscription/context, identify owning state/pipeline,
inspect resource Activity Log, check current plan/runtime, and confirm whether the value is
IaC-owned, app-CD-owned, or manual. Never use `terraform state rm` as a shortcut for deleting cloud
resources or erasing security gaps.

## 1. Manual and post-deployment task register

| Manual activity | Why / evidence | Required verification |
|---|---|---|
| ADO service connections, variable groups, pipeline authorization | Not represented in application Terraform; live ADO list denied. `[Code][Live]` | Correct CAIRO project, connection subscription and permissions; group access/pipeline permissions; no value disclosure. |
| ADO Environments/approvals | IaC YAML references `terraform_<region>_<env>` and app environments but does not define approver/check list. `[Code]` | Required reviewers, branch control, exclusive lock, bypass role. |
| Human Entra RBAC/PIM | Portal grants to people/groups and subscription roles; Contributor is not KV data-plane access. `[Chat]` | Effective role, scope, active PIM and audit entry. |
| Service principal role assignments | Pipeline identity, managed identity, app identity and human identity differ. `[Code][Chat]` | Correct principal object, minimum role, exact resource scope and data plane. |
| Key Vault secret population/rotation | Secret values are manually maintained or sourced by Terraform lookups; never in tfvars. `[Code][Chat]` | Correct secret name/version/expiry and application reference refresh. |
| KV network/IP exception for build agents | Hosted agents may lack PE routing/private DNS; history changed public flag and IP exceptions. `[Code][Chat]` | Agent egress is current and limited; close exception after use where possible. |
| O365 API connection consent and mailbox delegation | Workflow shell/ARM does not grant Exchange mailbox rights. `[Code][Chat]` | Connection authorization status, intended service identity, mailbox access, successful trigger/send test. |
| PostgreSQL extension allowlist | `pgcrypto`, `pg_trgm`, `fuzzystrmatch` per-server manual config. `[Doc][Chat]` | `SHOW azure.extensions`; create extension test/migration succeeds, each environment. |
| PE approval | Assistant PEs use manual approval; other PE connection status also requires verification. `[Code]` | Target service connection is Approved and DNS zone group/A record exists. |
| Databricks Git source setup | Git folder sync requires workspace-level Git credential, repo, ACL, entitlement; pipeline does not create all of this. `[Chat]` | Databricks Repo latest commit, correct workspace identity, folder ACL, run job separately. |
| Databricks job/notebook migration | Current source repo branch only proves sync pipeline, not job execution or notebook content. `[Code][Chat]` | Notebook/job is present, permissions/schedule/compute, a successful run and data outputs. |
| ADLS folder creation/POSIX ACL | `src/03-iac/masterscripts/AdlsPermissions.ps1` mutates filesystems and ACL recursively using key auth. `[Code]` | Correct account/filesystem/path/principal IDs, test with login-based ACL listing, change approval. |
| CORS and App Service port/startup changes | History contains portal changes for CORS, `WEBSITES_PORT`, MI config and app settings. `[Chat]` | Compare HCL/CD and live config; redeploy/health test; make intended owner durable. |
| Front Door custom routing / APIM APIs | Terraform defines infrastructure edge pieces; no complete API product/policy inventory in root. `[Code]` | Routes, WAF, APIM API/revision/subscription/policies confirmed in portal/source. |
| DNS records / stale private records | KV recreation left stale DNS; PE record can point at old IP. `[Chat]` | Resolver from workload subnet returns current private address; avoid deleting shared zone records blindly. |
| Azure Policy / Defender / SOC | Azure Policy code is commented; Defender changes historically reverted. `[Code][Chat]` | Subscription Policy assignments, Defender plans, secure score, alert receiver/routing. |
| App DB user roles | `system_admin`, `system_user`, `power_user` are application DB role assignments, not Azure RBAC. `[Code][Chat]` | User row/role/country scope and audit history in authorized app UI/DB process. |
| SAP CPI registration and callback | App config and SAP callback may be split across systems/branches. `[Code][Chat]` | Client registration, audience, certificates/credential rotation, callback auth, idempotency and SAP test. |

### 1.1 ADLS `src/03-iac` scripts—high impact

`[Code]` `AdlsPermissions.ps1` uses `az storage fs` with account-key auth to create filesystem
`data`, set root ACL, create `ai_suite`, set default/access ACLs and recursively set ACLs (reader
`r-x`, writer `rwx`). Environment `Wrapper.ps1` passes storage name and group/object IDs;
`execute.ps1` is an indirection that assumes repository-root working directory. Shared/Order
Automation copies contain Assistant-like names; target account ownership mismatch must be resolved
before execution.

**Do not run by default.** Preflight: inspect actual script and YAML in the exact branch; verify
account, subscription and filesystem; review all principal IDs with owner; confirm key auth
approved; use a test directory or `--auth-mode login` diagnostic if possible. The pipeline has no
Terraform plan/approval stage in reviewed folder YAMLs. `[Code]`

## 2. Safe context and inventory checks

Use PowerShell/Azure CLI with explicit subscription. These commands only read current
context/resources:

```powershell
az account show --query "{subscription:id,name:name,tenant:tenantId,user:user.name}" -o json
az account list --query "[].{name:name,id:id,isDefault:isDefault}" -o table
az group show --name rg-order_automation-dev --subscription <SUBSCRIPTION_ID> --query "{name:name,location:location}" -o json
az resource list --resource-group rg-order_automation-dev --subscription <SUBSCRIPTION_ID> --query "[].{name:name,type:type,location:location}" -o table
az monitor activity-log list --resource-group rg-order_automation-dev --subscription <SUBSCRIPTION_ID> --offset 7d --query "[].{time:eventTimestamp,operation:operationName.value,status:status.value,caller:caller}" -o table
```

Avoid printing app settings wholesale: settings may contain secrets. Prefer safe projections to
names only. `az webapp config appsettings list` can disclose values if queried without a names-only
projection. `[Live][Code]`

## 3. Runbook: prepare/deploy a new environment

1. **Confirm the actual isolation target.** DEV/QA/PROD groups all occur in subscription
`BASF_SC_CAIRO_PI_PROD`; separate suffixes do not equal separate subscriptions. Confirm
tenant/subscription, policy, cost owner and region with platform governance. `[Live]`
2. **Inventory current shared network.** Check VNet address space, existing subnet allocations, NSG,
route tables, private DNS zones/links and PE addresses. Do not assume free CIDR from tfvars.
`[Live][Code]`
3. **Check ownership boundaries.** Identify which root owns shared VNet/subnet/zone, which root
creates workload resources, and whether another state manages the same subnet. Order app root has
hardcoded dev subnet name/range; Assistant QA points at dev shared VNet. `[Code]`
4. **Prepare environment inputs.** Confirm `vars_<env>.tfvars` exists in selected branch, all
required inputs resolve, no `TODO-existing-*` placeholders remain, and service names meet global
uniqueness. Shared/Order checked roots lacked QA/PROD files in reviewed checkout. `[Code]`
5. **Prepare RBAC.** Verify service connection permissions for resource creation and role
assignment; arrange `AcrPull`, KV secret read, storage data roles and app MI roles separately.
Avoid granting subscription Owner as a convenience. `[Code][Chat]`
6. **Prepare backend.** Verify state RG/account/container, access and key. Use the root-specific
state key (`ec-shared.tfstate`, `ec-orderauto.tfstate`, `ec-assistfactory.tfstate`). Back up
state via approved process before address/import surgery. `[Code]`
7. **Prepare network dependencies.** Create subnets atomically with required NSG to satisfy deny
policy; verify PE network policy/delegation/service endpoints; plan private zones/VNet links and
inbound/outbound paths. `[Chat][Code]`
8. **Run pipeline plan only.** Confirm source branch/artifact and full plan for
replacement/deletes/imports; treat output/state/plan artifacts as sensitive. Do not bypass
approval. `[Code]`
9. **Apply through correct Environment gate.** Use exact artifact and service connection. Monitor
deployment, state write and each resource provisioning state. `[Code]`
10. **Complete manual gates:** approve PEs, populate KV secrets, authorize O365 connection,
allowlist DB extensions, create ADLS ACLs, configure Databricks Git/Jobs, complete app
deployment. `[Code][Doc]`
11. **Post-deploy acceptance:** DNS resolution from runtime subnet, private/public reachability,
managed identity effective roles, DB migrations/extensions, health endpoints, AFD origin health,
real mail trigger/upload/SAP test. `[Code][Chat]`
12. **Record ownership:** capture resource IDs, source branch/commit, Terraform state address,
manual deviation and support owner. Add new documented step to IaC or runbook; avoid long-lived
undocumented portal-only config.

## 4. Runbook: add an App Service safely

1. Decide whether it is workload-owned or shared platform; confirm app name, region/plan, VNet
subnet, AFD route/origin, public ingress and managed identity design.
2. Add HCL resource under owning repo and inputs for env-specific image/config; do not duplicate
service declarations in shared and app roots.
3. Use UAMI or system identity according to dependency; grant only `AcrPull` on registry and
data-plane roles at required service scope.
4. Confirm ACR network path from **build agent** and App Service image-pull path separately;
hosted-agent access and runtime MI pull are independent.
5. Add diagnostics for App Service, dependencies and AFD/WAF if not already in monitor graph; avoid
secret values in app settings.
6. Add pipeline branch/path trigger, variable group and service connection authorization. Decide ADO
vs GitHub authority.
7. Build a versioned image and scan; deploy one environment and inspect container
startup/port/health before promotion.
8. Configure AFD origin/route/WAF and direct-origin restriction; validate health probe and routing,
then verify direct app hostname is appropriately denied.
9. If database schema changes, ensure migration files are inside artifact/image and migration
identity has only required DB privileges.
10. Add resource/auth/diagnostic ownership to the resource register and runbook.

## 5. Runbook: rotate a Key Vault secret

1. Identify secret name, consumers, auth mechanism, expiry and rotation owner; check which apps use
direct SDK, Key Vault reference, Terraform lookup or pipeline variable.
2. Confirm target vault/env and access path. Example names: DEV `kv-orderautomation`, QA
`kv-orderautomation-qa`, PROD `kv-orderautomation-prod`. `[Live]`
3. Verify human is authorized to set secret and app MI/service principal has required secret read.
Human Contributor/Key Vault role is not automatically pipeline identity role. `[Chat]`
4. Generate new value in approved secret manager; never place it in shell history, ticket, tfvars,
output artifact or screenshot. Use portal secret creation or approved secure CLI input process.
5. Create a **new version** under same secret name; preserve previous version for rollback until
consumers verified. Avoid deleting/recreating the vault (purge-protection/global-name issue).
`[Chat]`
6. Validate network path and DNS independently from RBAC. If vault is private, consumer must reach
PE and correct private DNS; role alone will not fix timeout. `[Chat]`
7. Refresh app configuration/reference according to exact mechanism; restart only if required and
approved. Inspect Key Vault reference status without dumping values.
8. Run health/business transaction; verify DB login/API call. Check Activity Log and Key Vault
diagnostic/audit events.
9. Revoke/expire old credential only after all consumers transition; update credential rotation
inventory and incident rollback plan.
10. If Terraform data source reads secret, expect value in state. Do not add secret to output or
publish state. `[Code]`

## 6. Runbook: pipeline Key Vault firewall or 403

**Symptom separation:** `403` may be data-plane role denial, network ACL denial, private DNS/routing
failure, or wrong vault/identity. Diagnose the failure layer before changing policy.

```powershell
az keyvault show --name <VAULT> --resource-group <RG> --subscription <SUBSCRIPTION_ID> --query "{id:id,publicNetworkAccess:properties.publicNetworkAccess,networkAcls:properties.networkAcls,enableRbacAuthorization:properties.enableRbacAuthorization}" -o json
az role assignment list --scope <VAULT_RESOURCE_ID> --include-inherited --query "[].{role:roleDefinitionName,principal:principalName,scope:scope}" -o table
az network private-endpoint-connection list --name <VAULT> --resource-group <RG> --type Microsoft.KeyVault/vaults --subscription <SUBSCRIPTION_ID> -o table
```

1. Identify exact pipeline run, service connection, service principal/workload identity and vault in
that environment.
2. Read error: `ForbiddenByFirewall`/network-related status → agent egress/DNS; `Caller is not authorized`
→ RBAC/access model; timeout/name resolution → route/private DNS; `SecretNotFound` → wrong
name/version/vault.
3. Check vault RBAC vs access-policy setting, role scope, principal object ID and whether grant has
propagated. Pipeline identity is not the approving human.
4. Check hosted agent egress. Hosted pool may be unable to resolve/route private endpoint. Prefer
approved VNet-connected agent; if temporary public/IP exception is approved, scope narrowly and
remove after run.
5. Inspect DNS resolution from the actual build agent/runtime; PE DNS link to a workload VNet does
not help a Microsoft-hosted public agent.
6. Order Auto KV HCL ignores `network_acls`; reconcile manual change and intended IaC owner
deliberately. Do not assume next Terraform apply restores the desired rule.
7. Re-run plan with no secrets printed; execute only after environment approval. Verify read access
without echoing value; remove temporary firewall exception.

## 7. Runbook: Terraform drift or unexpected replacement

1. Stop concurrent applies; capture pipeline run, branch, Terraform version, provider lock/config
and state key.
2. Run read-only Azure inventory and Activity Log to establish what changed/when/by whom.
3. Check `ignore_changes`, `prevent_destroy`, moved/removed/import blocks, module ref, provider
major version and current state address.
4. Confirm single state owner. Do not import into another stack until original owner approves
handoff.
5. Produce new plan in correct pipeline. For adopted resources, `terraform import`/`import` blocks
update state; compare post-import plan to live before apply.
6. For transient `imports.tf`, preserve state and review destroy/replace lines. Remove temporary
imports only after verified apply, per source instructions.
7. If `state rm` already occurred, record that Azure resource remains and assign explicit owner; do
not interpret it as resource deletion/security repair.
8. For KV recreation, check deleted-vault recovery/name reservation first. For APIM, check
SKU/mode/subnet and expected provisioning latency. For subnet, check policy requiring NSG in
initial request.
9. Apply only a plan whose resource IDs and target environment were reviewed; verify Azure and state
agree afterwards.

## 8. Runbook: onboard user and diagnose 401/403

1. **User sign-in failure:** inspect FE MSAL tenant/client/redirect URI, browser devtools and Entra
App Registration Authentication. Ensure API base URL/environment was baked at correct FE build.
2. **401 on BE:** verify bearer token is present, not expired, `aud` matches BE accepted application
ID/URI and issuer tenant; FE source sends ID token, so ensure BE middleware contract
intentionally accepts it. Do not paste tokens into tickets.
3. **403 on BE:** distinguish valid Entra authentication from application authorization. Check user
record, `system_admin`/`system_user`/`power_user`, country/region scope and backend authorization
dependency. Azure RBAC role will not grant an application DB role.
4. **Service-to-service 401/403:** identify caller (SystemAssigned vs UAMI), request token for
recipient's expected audience/scope, confirm receiver app registration allowed-client list/JWKS;
check whether config says `mi` or `none`.
5. **Key Vault 403:** check data-plane KV role/access model **and** network/DNS; App Service MI may
be correct but no runtime VNet path.
6. **Storage 403:** confirm Blob data-plane role and auth mode. Connection string key auth differs
from MI; app `Contributor` on account does not equal blob data access.
7. **ACR 403/ImagePullUnauthorized:** check app UAMI has AcrPull on correct registry, App Service
selects that UAMI and registry allows network route. For agent docker push check AcrPush +
build-agent network separately.
8. **ADO service connection 403:** check project authorization, subscription scope, tenant,
federated credential and role grant. Do not use a human token as substitute.
9. Use Entra sign-in logs, Azure Activity Log, KV/Storage data-plane diagnostics (if enabled), App
Service logs and API request correlation ID. Never collect auth header/token.

## 9. Runbook: Logic App is not triggering / mail is not delivered

1. Open correct workflow: DEV `logicapp-orderautomatio`, QA `logicapp-order_automation-qa`;
production `logicapp-order_automation-prod` has historically been reported as an empty
shell—inspect current workflow definition. `[Live][Chat]`
2. Portal → Logic App → **Overview / Trigger history / Runs history**. No trigger history often
means trigger disabled/connector/recurrence issue, not backend HTTP failure.
3. Confirm Office 365 connection resource (`office365`, `office365-1`, `office365-support-dev`)
matches intended workflow; check connection status/reauthorization and mailbox delegation/service
account access.
4. Confirm trigger folder/shared mailbox and one-minute recurrence; review split-on and
sender/resend filters. Ensure test mail meets filters and attachment size/type constraints.
5. On failed run, inspect action and status only; redact email content, attachment names and signed
URLs before sharing. Check connection parameter `$connections` binding in deployed ARM
definition.
6. If trigger succeeded but HTTP failed, check target orchestrator URL, Managed Identity auth
audience/receiver role, App Service availability, request schema and response code.
7. If HTTP succeeded but no case, verify DB/Blob writes and case correlation ID.
8. For notification, confirm HTTP trigger access policy and Office365 `Send an email (V2)`
connection; test using approved test recipient, not a production customer.
9. After ARM deploy, manually trigger a test and verify actual mail received. Deployment success
alone is not verification. `[Chat]`

## 10. Runbook: PostgreSQL extension missing or database startup fails

1. Confirm exact server/database/env: DEV `pgsql-orderautomatio`, QA `pgsql-orderautomation-qa`,
PROD `pgsql-orderautomation-prod`. `[Live]`
2. Confirm runtime DNS/network and whether app connects through PE or public host; verify server
firewall and private zone from App Service integration subnet.
3. Inspect PostgreSQL server parameters/allowlist for required `pgcrypto`, `pg_trgm`,
`fuzzystrmatch` in the environment. Repeat per server; database migration will not automatically
allow them at Flexible Server level. `[Doc][Chat]`
4. With authorized DB client, use safe read-only checks:

```sql
SHOW azure.extensions;
SELECT extname FROM pg_extension ORDER BY extname;
SELECT current_database(), current_user;
```

5. Request approved server configuration change to include required extensions; then apply/create
extensions through Alembic or DBA-approved migration. Do not grant broad DB admin to the pipeline
by default.
6. Inspect App Service log stream and migration logs; verify image includes Alembic migration
directory and `RUN_MIGRATIONS_ON_STARTUP`/equivalent is intentionally set.
7. Verify expected tables/seed data (`regions`, `countries`, roles) using least-privilege read-only
query and test `/api/regions` plus authenticated FE flow.
8. Avoid emergency `AllowAll` firewall as permanent fix; if temporarily approved, record
owner/expiry and replace with correct private network path.

## 11. Runbook: APIM or Front Door 5xx

1. Check AFD endpoint/profile provisioning and route state; current DEV endpoint was
`order-automation-dev-d2end8fhgaemfncd.a01.azurefd.net`.
2. AFD → Origin groups → Origin health: confirm origin host header, HTTPS certificate validation,
probe path `/`, and correct `*.azurewebsites.net` site.
3. Check App Service itself with health endpoint/log stream before changing CORS/WAF. AFD static
site 200 doesn't prove backend API healthy.
4. Inspect WAF policy logs/action, AFD rule-set route, paths (`/api/*`, `/extraction/*`,
`/rules/*`), HTTPS forwarding and custom domain binding.
5. If APIM is involved, check APIM **Overview / Network / APIs / Diagnostics / Subscriptions**,
gateway URL, service status and VNet reachability. Dev APIM config has External mode and Internet
443 NSG; historical Internal activation stalled.
6. Verify APIM API backend URL/auth policy and test with authorized subscription/application token;
lack of public reachability may be intended for Internal mode.
7. Check NSG service rules and route/DNS to backend; APIM provisioning can take substantial time,
but long `Creating` alone does not prove eventual health.
8. Check app logs/DB/auth/dependency failures. Historical CORS symptom included empty DB/missing
migrations and ACR pull errors; correct these roots before adding headers.
9. Validate via AFD URL and direct origin policy; capture request/correlation ID and timestamps;
redact tokens.

## 12. Runbook: DNS/private endpoint not resolving

1. Find the PE target/subresource/connection approval and assigned private IP.
2. Inspect DNS zone exact suffix, zone group, VNet link and A record. Watch for Assistant KV
`.azure.com` vs shared/app `.azure.net` mismatch. `[Code][Live]`
3. Run `Resolve-DnsName <service-hostname>` from the runtime VNet/agent that needs access. Local
workstation lookup is not proof of App Service/VNet DNS.
4. Verify App Service VNet integration and route-all setting; inspected AIOA app service did not
show VNet integration for some current surfaces. `[Live]`
5. Test TCP/service health from an authorized network; if only public path works, identify whether
service public access is enabled and whether data policy allows it.
6. Check for stale DNS A records after resource recreation; remove only the stale record after
identifying zone owner and consumers.

## 13. Where to look in Azure Portal / ADO

- Portal → **Resource groups / Activity log**: ownership, recent mutations, failed deployments.
- Portal → **Entra ID → Enterprise applications / App registrations / Sign-in logs / PIM**: caller
identity, token audience registration and human role activation.
- Portal → **Key Vault → Networking / Access configuration / IAM / Secrets**; **Storage → Networking
/ Containers / Access control**.
- Portal → **App Service → Identity / Configuration / Networking / Deployment Center / Log stream /
Diagnose and solve problems / Application Insights**.
- Portal → **Azure Database for PostgreSQL → Server parameters / Networking / Databases / Logs**;
**SQL Server → Networking / Microsoft Entra admin / Auditing**.
- Portal → **Logic Apps → Trigger history / Runs history / API connections**; Microsoft 365/Exchange
admin for mailbox permissions.
- Portal → **Private DNS zones / Private endpoints / VNet → Subnets/NSG**; Front Door and CDN →
Origin health / routes / WAF.
- ADO → **CAIRO → Pipelines / Library / Environments / Project settings → Service connections, Agent
pools**.
- Databricks workspace → **Settings / Identity / Linked accounts / Repos / Jobs / Permissions**.

## Interview talking points

- State clearly which steps are automation and which require human/cloud operations work.
- Give a least-privilege answer that separates role, network path, DNS and service auth.
- Explain safe Terraform drift response and why `state rm` does not repair live cloud resources.
- Explain why Logic App API connection consent and mailbox access are outside ARM deployment.
- Explain per-environment Postgres extension allowlisting and migration verification.
- Explain that the same-subscription dev/qa/prod separation is logical, not subscription isolation.

## Evidence & confidence

- Manual procedures are grounded in `raw_iac.md` post-step scripts and active HCL; historical
runbook outcomes from `raw_chats.md`; architecture extension requirement from `doc_aioa.txt`.
`[Code][Chat][Doc]`
- Current resource names from `raw_azure.md`; actual RBAC/service-connection/variable-group settings
were not queried due permission limits. `[Live]`
- Commands here are safe diagnostics / suggested process, not commands executed during evidence
collection.

## Further reading

- [04 Identity/Security/Governance](04-Identity-Security-Governance.md) · [08
Terraform/IaC](08-Terraform-IaC.md) · [09 CI/CD](09-CICD-Azure-DevOps.md) · [12
Incidents](12-Incidents-and-Lessons-Learned.md)

## 14. Runbook: distinguish CORS from API/backend failure

1. Record the FE URL, API URL, request method, timestamp and correlation ID; do not capture an
   Authorization header.
2. In browser DevTools, inspect whether the failing request is the `OPTIONS` preflight or the real
   API call.
3. If preflight fails, compare exact `Origin`, requested headers and method with App Service CORS
   and AFD response-header behavior.
4. If the real API returns 5xx, inspect App Service log stream and App Insights exception/dependency
   traces first.
5. Check whether the container started and whether health probe is passing; review `WEBSITES_PORT`
   against image listener.
6. Check ACR pull status and deployed image tag; an old image can make source-level assumptions
   misleading.
7. Confirm database connection, schema/migrations and required seed/reference data. A blank table
   can be application failure with a CORS-looking browser message.
8. Confirm backend logs include the request and the expected environment/DB host; avoid printing
   connection strings.
9. Once backend returns correct response, configure only the exact FE origin and methods needed.
10. Retest from the FE through AFD with Entra authentication and verify API payload, not only
    response headers.
11. Verify no unintended direct App Service route bypasses WAF/AFD. The inspected DEV extraction
    config allowed all inbound requests.
12. Capture the final result and owner in incident notes; do not widen CORS to `*` as a blanket
    recovery.

## 15. Runbook: validate Front Door to origin protection

1. Portal → Front Door profile → Endpoint → Routes: verify path patterns, protocol, redirect and
   selected origin group.
2. For each route, inspect the origin hostname and origin host header; verify TLS certificate-name
   check.
3. Inspect Origin group health probe path/protocol and sample settings; confirm a 200 response from
   intended endpoint.
4. Check the AFD WAF policy is associated with the endpoint/pattern and is in intended
   Prevention/Detection mode.
5. Review WAF logs/diagnostic settings for rule match before modifying rules.
6. Inspect backend App Service IP restriction list and Front Door ID/header. HCL only applies
   restrictions when the FDID input is supplied.
7. Send a controlled request through AFD and confirm expected response; use browser/API client with
   a non-sensitive test account.
8. Test direct `*.azurewebsites.net` path from an approved source and confirm it is blocked or
   explicitly accepted by design.
9. Validate CORS policy on the final public hostname, not the origin hostname.
10. Review Activity Log and deployment pipeline for a later config writer that may have removed
    restrictions.
11. Record any route/origin exception and owning team; do not claim AFD is the sole entry point
    without direct-origin verification.
