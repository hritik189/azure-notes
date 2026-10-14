# 14 — Resource register and glossary

## TL;DR

- Use this index to locate actual resources before navigating Azure. [Live]
- RG/environment/type identify scope; name alone is not ownership proof.
- The condensed register includes principal workload/platform/support resources.
- Runtime relationships are source-derived, not proved for every current transaction. [Code]
- The glossary distinguishes infrastructure, identity, data, delivery and SAP terms.

## High-level: using the register

Search an exact resource name and follow its type/RG to the corresponding chapter.
Inventory includes a real TODO-named Front Door and multiple callback resources.
A similarly named resource is not automatically duplicate or safe to delete.
If a name is missing, check live snapshot coverage/permission boundaries before declaring absence.

| Need | Chapter |
|---|---|
| Locate environment/RG | 02 |
| Resolve route/subnet/PE/DNS | 03 |
| Identify permission/credential ownership | 04 |
| Identify executing code/image/settings | 05 |
| Identify data/model/event/SAP integration | 06 |
| Identify incoming route/edge/gateway | 07 |
| Identify Terraform state/resource owner | 08 |
| Identify YAML/group/service connection | 09 |
| Identify functional transaction | 10 |
| Identify manual prerequisite | 11 |
| Identify prior failure | 12 |
| Practice explanation | 13 |

## Low-level: condensed resource register

Name/type/RG/environment are captured live metadata.
Purpose/connections are interpretation from resource type and source responsibility.
A “connected to” entry is not proof of successful current network traffic.
Detailed fields remain in domain chapters.
Full inventory was 340 visible resources; this register omits repetitive NIC/system-topic/managed alert objects.

| Name [Live] | Type | RG | Env | Purpose | Connected to / evidence caveat |
| --- | --- | --- | --- | --- | --- |
| BASF_VN_PI_BASF_SC_CAIRO_PI_PROD | virtualNetworks | BASF_RG_Network_BASF_SC_CAIRO_PI_PROD | prod | Network boundary | Subnets, NSGs, PEs/DNS; QA shared DEV |
| BASF_VN_PI_BASF_SC_CAIRO_PI_PROD_Bastion | virtualNetworks | BASF_RG_Network_BASF_SC_CAIRO_PI_PROD | prod | Network boundary | Subnets, NSGs, PEs/DNS; QA shared DEV |
| vnet-sharedresources-dev | virtualNetworks | rg-sharedresources-dev | dev | Network boundary | Subnets, NSGs, PEs/DNS; QA shared DEV |
| stsharedresourced2slfk | storageAccounts | rg-sharedresources-dev | dev | ML/AI workspace storage | ML/Hub/Project dependencies |
| kv-sharedresources-dev | vaults | rg-sharedresources-dev | dev | Secret/key management | Pipeline/runtime identity; firewall + RBAC |
| acrsharedresourced2slfk | registries | rg-sharedresources-dev | dev | Container image registry | App CD push / MI pull |
| managed-appi-mlops-sharedresources-dev-ws | workspaces | ai_appi-mlops-sharedresources-dev_6c149add-cabf-483c-b4dc-a4390f1991b3_managed | dev | Central log workspace | App Insights/resource diagnostics |
| aihubsharedres8407751719 | accounts | rg-sharedresources-dev | dev | AI model/project/DI/Speech | Extraction/Assistant/AI project; actual selected deployment verify |
| aihubsharedres7811513971 | storageAccounts | rg-sharedresources-dev | dev | ML/AI workspace storage | ML/Hub/Project dependencies |
| aihub-sharedresources-dev | workspaces | rg-sharedresources-dev | dev | ML/AI Hub/Project workspace | Vault, storage, ACR, telemetry |
| aiproject-sharedresources-dev | workspaces | rg-sharedresources-dev | dev | ML/AI Hub/Project workspace | Vault, storage, ACR, telemetry |
| bingsearch-sharedresources-dev | accounts | rg-sharedresources-dev | dev | Supporting platform service | Owner/active consumer verify |
| aihubsharedres8407751719/aifoundry-project-sharedresources-dev | projects | rg-sharedresources-dev | dev | AI model/project/DI/Speech | Extraction/Assistant/AI project; actual selected deployment verify |
| aisearch-sharedresources-dev | searchServices | rg-sharedresources-dev | dev | AI Search index engine | Intended embeddings/retrieval; actual OA caller verify |
| appserviceplan-dev | serverFarms | rg-order_automation-dev | dev | Managed app/Function compute | Corresponding environment sites |
| functionappservice-order_automation-dev | serverFarms | rg-order_automation-dev | dev | Managed app/Function compute | Corresponding environment sites |
| egt-orderautomatio | topics | rg-order_automation-dev | dev | Extraction-completion topic | Extraction publisher → orchestrator webhook/dead letter |
| bloborderautomatio | storageAccounts | rg-order_automation-dev | dev | Document/dead-letter Blob | Orchestrator/Event Grid |
| pgsql-orderautomatio | flexibleServers | rg-order_automation-dev | dev | Relational state | OA workflow or Assistant DB |
| storderautomatioc3cj2s | storageAccounts | rg-order_automation-dev | dev | Function/deployment storage | Function host/deployment assets |
| sqlserver-orderautomatio | servers | rg-order_automation-dev | dev | SQL logical server | Extraction SQL DB/auth/firewall |
| logicapp-orderautomatio | workflows | rg-order_automation-dev | dev | Mail/notification/incident workflow | Office 365/API connections/backend/job |
| law-orderautomatio | workspaces | rg-order_automation-dev | dev | Central log workspace | App Insights/resource diagnostics |
| staticorderautomatio | storageAccounts | rg-order_automation-dev | dev | React static website | Front Door StaticOrigin |
| managed-appi-orderautomatio-ws | workspaces | ai_appi-orderautomatio_262abf35-58f1-4a36-bf43-38ade07a896a_managed | dev | Central log workspace | App Insights/resource diagnostics |
| extract-orderautomatio | sites | rg-order_automation-dev | dev | Async extraction API | SQL, Document Intelligence, LLM router, Event Grid |
| orchestrator-api-appservice-dev | sites | rg-order_automation-dev | dev | Workflow API | PostgreSQL, Blob, extraction/rules, Databricks SQL, SAP CPI |
| rules-engine-appservice-dev | sites | rg-order_automation-dev | dev | Business decision API | Orchestrator/extraction, rule/reference DB |
| functionapp-orderautomatio | sites | rg-order_automation-dev | dev | Test app / Function shell | Plan/runtime storage; functional purpose verify |
| sqlserver-orderautomatio/master | databases | rg-order_automation-dev | dev | SQL database | Extraction state or system master DB |
| sqlserver-orderautomatio/sqldb-order_automation-dev | databases | rg-order_automation-dev | dev | SQL database | Extraction state or system master DB |
| dlsecorderautodev | storageAccounts | rg-order_automation-dev | dev | ADLS hierarchy/data zones | Lake/ACL scripts; runtime jobs verify |
| documentintel-orderautomation-dev | accounts | rg-order_automation-dev | dev | AI model/project/DI/Speech | Extraction/Assistant/AI project; actual selected deployment verify |
| order-automation-fd-dev | profiles | rg-order_automation-dev | dev | Global edge routing | Static website / App Service origins |
| orderautomationwafdev | frontdoorWebApplicationFirewallPolicies | rg-order_automation-dev | dev | Front Door WAF policy | AFD security association; actual rule coverage verify |
| uami-order_automation-dev | userAssignedIdentities | rg-order_automation-dev | dev | Independent managed identity | ACR pull or Recovery Services permissions |
| rgorderautomationde94c5 | storageAccounts | rg-order_automation-dev | dev | Function/deployment storage | Function host/deployment assets |
| ASP-rgorderautomationdev-ad27 | serverFarms | rg-order_automation-dev | dev | Managed app/Function compute | Corresponding environment sites |
| salesorderstatus | sites | rg-order_automation-dev | dev | SAP callback Function | SAP result → orchestrator (active deployment verify) |
| TODO-existing-frontdoor-prefix-fd-dev | profiles | rg-sharedresources-dev | dev | Global edge routing | Static website / App Service origins |
| TODOexistingfrontdoorprefixwafdev | frontdoorWebApplicationFirewallPolicies | rg-sharedresources-dev | dev | Front Door WAF policy | AFD security association; actual rule coverage verify |
| mlw-sharedresources-dev | workspaces | rg-sharedresources-dev | dev | ML/AI Hub/Project workspace | Vault, storage, ACR, telemetry |
| apim-integration-test-backend-dev | sites | rg-order_automation-dev | dev | Test app / Function shell | Plan/runtime storage; functional purpose verify |
| apim-sharedresources-dev | service | rg-sharedresources-dev | dev | API gateway instance | snet-apim; no captured APIs |
| law-orderautomation-qa | workspaces | rg-order_automation-qa | qa | Central log workspace | App Insights/resource diagnostics |
| order-automation-fd-qa | profiles | rg-order_automation-qa | qa | Global edge routing | Static website / App Service origins |
| appserviceplan-order_automation-qa | serverFarms | rg-order_automation-qa | qa | Managed app/Function compute | Corresponding environment sites |
| logicapp-order_automation-qa | workflows | rg-order_automation-qa | qa | Mail/notification/incident workflow | Office 365/API connections/backend/job |
| ststaticorderautoqa | storageAccounts | rg-order_automation-qa | qa | React static website | Front Door StaticOrigin |
| uami-order_automation-qa | userAssignedIdentities | rg-order_automation-qa | qa | Independent managed identity | ACR pull or Recovery Services permissions |
| stfuncorderautoqamw1rh7 | storageAccounts | rg-order_automation-qa | qa | Function/deployment storage | Function host/deployment assets |
| kv-orderautomation-qa | vaults | rg-order_automation-qa | qa | Secret/key management | Pipeline/runtime identity; firewall + RBAC |
| functionappservice-order_automation-qa | serverFarms | rg-order_automation-qa | qa | Managed app/Function compute | Corresponding environment sites |
| stbloborderautoqa | storageAccounts | rg-order_automation-qa | qa | Document/dead-letter Blob | Orchestrator/Event Grid |
| stdlsorderautoqa | storageAccounts | rg-order_automation-qa | qa | ADLS hierarchy/data zones | Lake/ACL scripts; runtime jobs verify |
| sqlserver-orderautomation-qa | servers | rg-order_automation-qa | qa | SQL logical server | Extraction SQL DB/auth/firewall |
| egt-orderautomation-qa | topics | rg-order_automation-qa | qa | Extraction-completion topic | Extraction publisher → orchestrator webhook/dead letter |
| pgsql-orderautomation-qa | flexibleServers | rg-order_automation-qa | qa | Relational state | OA workflow or Assistant DB |
| orderautomationwafqa | frontdoorWebApplicationFirewallPolicies | rg-order_automation-qa | qa | Front Door WAF policy | AFD security association; actual rule coverage verify |
| orchestrator-api-appservice-qa | sites | rg-order_automation-qa | qa | Workflow API | PostgreSQL, Blob, extraction/rules, Databricks SQL, SAP CPI |
| rules-engine-appservice-qa | sites | rg-order_automation-qa | qa | Business decision API | Orchestrator/extraction, rule/reference DB |
| extract-orderautomation-qa | sites | rg-order_automation-qa | qa | Async extraction API | SQL, Document Intelligence, LLM router, Event Grid |
| functionapp-orderautomation-qa | sites | rg-order_automation-qa | qa | Test app / Function shell | Plan/runtime storage; functional purpose verify |
| sqlserver-orderautomation-qa/master | databases | rg-order_automation-qa | qa | SQL database | Extraction state or system master DB |
| sqlserver-orderautomation-qa/sqldb-order_automation-qa | databases | rg-order_automation-qa | qa | SQL database | Extraction state or system master DB |
| rgorderautomationqabbce | storageAccounts | rg-order_automation-qa | qa | Function/deployment storage | Function host/deployment assets |
| ASP-rgorderautomationqa-ad96 | serverFarms | rg-order_automation-qa | qa | Managed app/Function compute | Corresponding environment sites |
| salesorderstatus-qa | sites | rg-order_automation-qa | qa | SAP callback Function | SAP result → orchestrator (active deployment verify) |
| vnet-sharedresources-prod | virtualNetworks | rg-sharedresources-prod | prod | Network boundary | Subnets, NSGs, PEs/DNS; QA shared DEV |
| stsharedresourceezuzal | storageAccounts | rg-sharedresources-prod | prod | ML/AI workspace storage | ML/Hub/Project dependencies |
| acrsharedresourceezuzal | registries | rg-sharedresources-prod | prod | Container image registry | App CD push / MI pull |
| managed-appi-mlops-sharedresources-prod-ws | workspaces | ai_appi-mlops-sharedresources-prod_f195cd14-8b1d-43f9-817b-b5912aa8b6ca_managed | prod | Central log workspace | App Insights/resource diagnostics |
| apim-sharedresources-prod | service | rg-sharedresources-prod | prod | API gateway instance | snet-apim; no captured APIs |
| kv-sharedre-prod-ezuzal | vaults | rg-sharedresources-prod | prod | Secret/key management | Pipeline/runtime identity; firewall + RBAC |
| mlw-sharedresources-prod | workspaces | rg-sharedresources-prod | prod | ML/AI Hub/Project workspace | Vault, storage, ACR, telemetry |
| office365 | connections | rg-order_automation-dev | dev | Managed connector binding | Logic App / manual consent |
| conversionservice | connections | rg-order_automation-dev | dev | Managed connector binding | Logic App / manual consent |
| logicapp-order_automation-prod | workflows | rg-order_automation-prod | prod | Mail/notification/incident workflow | Office 365/API connections/backend/job |
| law-orderautomation-prod | workspaces | rg-order_automation-prod | prod | Central log workspace | App Insights/resource diagnostics |
| pgsql-orderautomation-prod | flexibleServers | rg-order_automation-prod | prod | Relational state | OA workflow or Assistant DB |
| stfuncorderautoprodok2ed | storageAccounts | rg-order_automation-prod | prod | Function/deployment storage | Function host/deployment assets |
| functionappservice-order_automation-prod | serverFarms | rg-order_automation-prod | prod | Managed app/Function compute | Corresponding environment sites |
| order-automation-fd-prod | profiles | rg-order_automation-prod | prod | Global edge routing | Static website / App Service origins |
| kv-orderautomation-prod | vaults | rg-order_automation-prod | prod | Secret/key management | Pipeline/runtime identity; firewall + RBAC |
| sqlserver-orderautomation-prod | servers | rg-order_automation-prod | prod | SQL logical server | Extraction SQL DB/auth/firewall |
| orderautomationwafprod | frontdoorWebApplicationFirewallPolicies | rg-order_automation-prod | prod | Front Door WAF policy | AFD security association; actual rule coverage verify |
| ststaticorderautoprod | storageAccounts | rg-order_automation-prod | prod | React static website | Front Door StaticOrigin |
| uami-order_automation-prod | userAssignedIdentities | rg-order_automation-prod | prod | Independent managed identity | ACR pull or Recovery Services permissions |
| egt-orderautomation-prod | topics | rg-order_automation-prod | prod | Extraction-completion topic | Extraction publisher → orchestrator webhook/dead letter |
| stbloborderautoprod | storageAccounts | rg-order_automation-prod | prod | Document/dead-letter Blob | Orchestrator/Event Grid |
| sqlserver-orderautomation-prod/master | databases | rg-order_automation-prod | prod | SQL database | Extraction state or system master DB |
| appserviceplan-order_automation-prod | serverFarms | rg-order_automation-prod | prod | Managed app/Function compute | Corresponding environment sites |
| stdlsorderautoprod | storageAccounts | rg-order_automation-prod | prod | ADLS hierarchy/data zones | Lake/ACL scripts; runtime jobs verify |
| functionapp-orderautomation-prod | sites | rg-order_automation-prod | prod | Test app / Function shell | Plan/runtime storage; functional purpose verify |
| rules-engine-appservice-prod | sites | rg-order_automation-prod | prod | Business decision API | Orchestrator/extraction, rule/reference DB |
| orchestrator-api-appservice-prod | sites | rg-order_automation-prod | prod | Workflow API | PostgreSQL, Blob, extraction/rules, Databricks SQL, SAP CPI |
| extract-orderautomation-prod | sites | rg-order_automation-prod | prod | Async extraction API | SQL, Document Intelligence, LLM router, Event Grid |
| sqlserver-orderautomation-prod/sqldb-order_automation-prod | databases | rg-order_automation-prod | prod | SQL database | Extraction state or system master DB |
| documentintel-orderautomation-prod | accounts | rg-order_automation-prod | prod | AI model/project/DI/Speech | Extraction/Assistant/AI project; actual selected deployment verify |
| aihubsharedres7930556471 | storageAccounts | rg-sharedresources-prod | prod | ML/AI workspace storage | ML/Hub/Project dependencies |
| aihubsharedres3426890388 | accounts | rg-sharedresources-prod | prod | AI model/project/DI/Speech | Extraction/Assistant/AI project; actual selected deployment verify |
| aihub-sharedresources-prod | workspaces | rg-sharedresources-prod | prod | ML/AI Hub/Project workspace | Vault, storage, ACR, telemetry |
| aiproject-sharedresources-prod | workspaces | rg-sharedresources-prod | prod | ML/AI Hub/Project workspace | Vault, storage, ACR, telemetry |
| aihubsharedres3426890388/aifoundry-project-shared-prod | projects | rg-sharedresources-prod | prod | AI model/project/DI/Speech | Extraction/Assistant/AI project; actual selected deployment verify |
| rgorderautomationprbb6d | storageAccounts | rg-order_automation-prod | prod | Function/deployment storage | Function host/deployment assets |
| ASP-rgorderautomationprod-b631 | serverFarms | rg-order_automation-prod | prod | Managed app/Function compute | Corresponding environment sites |
| sapordercallback-prod | sites | rg-order_automation-prod | prod | SAP callback Function | SAP result → orchestrator (active deployment verify) |
| rgorderautomationpr9c9e | storageAccounts | rg-order_automation-prod | prod | Function/deployment storage | Function host/deployment assets |
| office365-1 | connections | rg-order_automation-dev | dev | Managed connector binding | Logic App / manual consent |
| conversionservice | connections | rg-order_automation-qa | qa | Managed connector binding | Logic App / manual consent |
| office365 | connections | rg-order_automation-qa | qa | Managed connector binding | Logic App / manual consent |
| office365 | connections | rg-order_automation-prod | prod | Managed connector binding | Logic App / manual consent |
| conversionservice | connections | rg-order_automation-prod | prod | Managed connector binding | Logic App / manual consent |
| kv-orderautomation | vaults | rg-order_automation-dev | dev | Secret/key management | Pipeline/runtime identity; firewall + RBAC |
| logicapp-test | workflows | rg-order_automation-dev | dev | Mail/notification/incident workflow | Office 365/API connections/backend/job |
| pqsql-assistantfactory-test | flexibleServers | rg-assistant_factory-test | test | Relational state | OA workflow or Assistant DB |
| BASF-ID-RSV-CAIRO-PI-PROD-we | userAssignedIdentities | BASF_RG_RecoveryServicesVault | prod | Independent managed identity | ACR pull or Recovery Services permissions |
| basfkvcairo0947583261 | vaults | BASF_RG_RecoveryServicesVault | Corporate/shared | Secret/key management | Pipeline/runtime identity; firewall + RBAC |
| BASF-RecoveryServicesVault | vaults | BASF_RG_RecoveryServicesVault | Corporate/shared | Backup vault | Corporate protection; protected items verify |
| assistantfactory-swecen-resource | accounts | rg-sharedresources-dev | dev | AI model/project/DI/Speech | Extraction/Assistant/AI project; actual selected deployment verify |
| assistantfactory-swecen-resource/assistantfactory-swecen | projects | rg-sharedresources-dev | dev | AI model/project/DI/Speech | Extraction/Assistant/AI project; actual selected deployment verify |
| assistantfactory-appservice-plan-dev | serverFarms | rg-assistant_factory-dev | dev | Managed app/Function compute | Corresponding environment sites |
| assistantapiwaf | frontdoorWebApplicationFirewallPolicies | rg-assistant_factory-dev | dev | Front Door WAF policy | AFD security association; actual rule coverage verify |
| assistantfactory-dev | profiles | rg-assistant_factory-dev | dev | Global edge routing | Static website / App Service origins |
| assistantfactory-mcp-appservice-plan-dev | serverFarms | rg-assistant_factory-dev | dev | Managed app/Function compute | Corresponding environment sites |
| managed-assistantfactory-appinsights-dev-ws | workspaces | ai_assistantfactory-appinsights-dev_fe9a288d-8039-4aaf-b5b9-fcc86ede7479_managed | dev | Central log workspace | App Insights/resource diagnostics |
| assistantadlsdev | storageAccounts | rg-assistant_factory-dev | dev | ADLS hierarchy/data zones | Lake/ACL scripts; runtime jobs verify |
| assistantfactory-kv-dev | vaults | rg-assistant_factory-dev | dev | Secret/key management | Pipeline/runtime identity; firewall + RBAC |
| assistantfactory-tts-dev | accounts | rg-assistant_factory-dev | dev | AI model/project/DI/Speech | Extraction/Assistant/AI project; actual selected deployment verify |
| assistantfactory-appservice-dev | sites | rg-assistant_factory-dev | dev | Assistant app / MCP app | Shared DEV VNet, ACR/AI/DB; external MCP image |
| assistantfactory-mcp-appservice-dev | sites | rg-assistant_factory-dev | dev | Assistant app / MCP app | Shared DEV VNet, ACR/AI/DB; external MCP image |
| assistantfactory-cosmosdb-account-dev | databaseAccounts | rg-assistant_factory-dev | dev | Assistant Cosmos DB | Assistant data/PE; actual active usage verify |
| assistantfactory-postgresql-dev | flexibleServers | rg-assistant_factory-dev | dev | Relational state | OA workflow or Assistant DB |
| apic-mcp-dev | services | rg-mcp-registry-dev | dev | API/MCP catalog | Registry metadata; not runtime gateway |
| logicapp-email-to-bug-dev | workflows | rg-order_automation-dev | dev | Mail/notification/incident workflow | Office 365/API connections/backend/job |
| office365-support-dev | connections | rg-order_automation-dev | dev | Managed connector binding | Logic App / manual consent |
| tfsharedresourcesdev | storageAccounts | tf_state_shared_dev | dev | Terraform state backend | IaC pipeline/state locking |
| tforderautodev1aafad | storageAccounts | tf_state_shared_dev | dev | Terraform state backend | IaC pipeline/state locking |
| tforderautoqac1b485 | storageAccounts | tf_state_orderautomation_qa | qa | Terraform state backend | IaC pipeline/state locking |
| tfsharedresourcesprod | storageAccounts | tf_state_shared_prod | prod | Terraform state backend | IaC pipeline/state locking |
| tforderautoproddb0729 | storageAccounts | tf_state_orderautomation_prod | prod | Terraform state backend | IaC pipeline/state locking |
| tfassistantapidev1aafad | storageAccounts | tf_state_shared_dev | dev | Terraform state backend | IaC pipeline/state locking |
| DefaultWorkspace-87a57a8d-4e3f-4cf5-b508-ce306917ec5f-DEWC | workspaces | DefaultResourceGroup-DEWC | Corporate/shared | Central log workspace | App Insights/resource diagnostics |



### Known external/source-only components

| Component | Evidence | Why not represented as a live CAIRO resource |
|---|---|---|
| dbks-ec DEV Databricks | Historical adb-5764847957066702.2.azuredatabricks.net | Owning subscription unverified |
| dbks-ec-qual | Historical adb-8081330705788048.8.azuredatabricks.net | CAIRO workspace list empty |
| playgroundacrprod.azurecr.io | Current MCP image reference | External registry not among two captured registries |
| SAP S4HANA / CPI | Business/source integrations | SAP landscape outside captured Azure inventory |
| Content Safety / Translator | Doc/shared library capability | No dedicated captured CAIRO account established |
| Entra registrations/enterprise apps | Code/history auth contracts | ARM resource register is not full directory inventory |
| ADO org basf4EMEA / CAIRO | Repository/YAML/history | ADO live list calls denied |
| Explore frontend endpoint | .env.explore source | No distinct live CAIRO profile captured |

## Glossary


### AIOA — AI Order Automation

Customer document → extracted/enriched/reviewed SAP order workflow.

### CAIRO — BASF project/platform name

Azure DevOps project and primary platform context; not an Azure service.

### ADO — Azure DevOps

Repos, Pipelines, Library, Boards and Artifacts in basf4EMEA/CAIRO.

### BE — Backend

FastAPI services and workflow templates, not browser assets.

### FE — Frontend

React/Vite SPA compiled into Storage static website assets.

### SPA — Single-page application

Client router needs index.html fallback for deep links.

### SaaS — Software as a Service

Provider operates application platform; customer still owns identity/data/config.

### PaaS — Platform as a Service

Provider manages host/engine; customer owns app/schema/security configuration.

### IaaS — Infrastructure as a Service

Customer controls compute/network primitives; Azure operates physical fabric.

### RG — Resource group

ARM management container, not itself a network or identity isolation boundary.

### ARM — Azure Resource Manager

Control-plane API/deployment layer; separate from data-plane authorization.

### AzAPI — Terraform Azure API provider

Used to submit atomic subnet bodies containing the NSG ID.

### AzureRM — Terraform Azure resource provider

Typed Azure resources/data sources; separate from ARM template deployment.

### IaC — Infrastructure as Code

Desired infrastructure and state, not all runtime/manual prerequisites.

### Terraform state — Resource ownership/value record

Can contain sensitive data; privileged remote backend.

### Backend — Terraform remote-state store, or app server

Context matters; Terraform backend is not orchestrator API.

### Drift — Desired/actual state divergence

Can come from manual edits, another pipeline or ignored properties.

### Import — Adopt existing resource into state

Does not by itself prove every property is correctly managed.

### Moved block — Terraform state-address migration

Changes logical address while retaining resource identity.

### Removed block — Terraform ownership removal

With destroy=false forgets state without deleting resource.

### ignore_changes — Terraform lifecycle setting

Avoids overwriting selected drift but does not appoint a new owner.

### TFVAR — Terraform input variable file

Selected file/branch must exist for intended environment.

### CI — Continuous integration

Build/check artifacts; not equivalent to production deployment.

### CD — Continuous delivery/deployment

Publishes image/assets/workflow or infrastructure; functional test separate.

### YAML — Pipeline/workflow configuration format

Source presence does not prove registered/enabled pipeline.

### Variable group — ADO shared configuration collection

Secret/key-linked groups and pipeline permissions need independent control.

### Service connection — ADO external service identity binding

Pipeline caller authorization, not runtime MI authority.

### WIF — Workload identity federation

Short-lived federated trust, not embedded client secret.

### PAT — Personal access token

Secret scoped to service operations; names only in handbook.

### Private feed — Azure Artifacts package source

Packaging permissions/build credential propagation matter.

### Trivy — Container scanner

Pipeline task definition is not proof latest build passed scanning.

### Entra ID — Microsoft identity platform

Tenant, registrations, enterprise apps and workload/user sign-in.

### Tenant — Directory/security authority

Distinct from Azure subscription and application business tenant scope.

### App registration — Application identity definition

Client ID, audiences, scopes, redirect URIs and roles.

### Enterprise application — Service-principal instance

Tenant-local permissions/assignments for the registered application.

### SP — Service principal

Workload directory identity; separate from human and MI selection.

### Client ID — Application identifier

Used to select/request identity; not the principal/object ID used for grant target.

### Object ID — Directory object identifier

Identifies exact user/group/SP object in permission assignments.

### MI — Managed identity

Azure-managed workload credential lifecycle.

### UAMI — User-assigned managed identity

Independent identity attached to OA apps for image pulls/selected auth.

### SAMI — System-assigned managed identity

Identity lifetime tied to owning resource.

### AcrPull — Azure Container Registry pull role

Grants image pull, not network access or push.

### RBAC — Role-based access control

Azure scoped permissions; separate from app DB roles and DB grants.

### IAM — Identity and access management

Azure portal Access control blade plus wider identity process.

### PIM — Privileged Identity Management

Time-bound eligible privileged-role activation, not automatic secret access.

### MSAL — Microsoft Authentication Library

FE login/token handling; inspected API bearer is ID token.

### OAuth 2.0 — Delegated/workload authorization protocol

Token grants/audience/consent contracts need explicit design.

### OIDC — OpenID Connect

Identity/authentication layer; ID token is not automatically an API access token.

### JWT — JSON Web Token

Validate issuer/audience/signature/expiry/caller/claims as required.

### JWKS — JSON Web Key Set

Public signing-key metadata used for token validation.

### Audience — Intended token recipient

Mismatch causes API auth failure even with otherwise valid token.

### Scope — Requested/authorized operation or token resource

Different from Azure role-assignment scope.

### OBO — On-Behalf-Of

Downstream token flow carrying delegated user context; SAP acceptance unverified.

### App role — Application permission/role

system_admin/system_user/power_user are business roles, not Azure RBAC.

### Consent — Approval of delegated/application permissions

Separate from mailbox rights and Azure role grants.

### Least privilege — Minimum required authority

Scope identity/action/resource and review effective/inherited grants.

### Key Vault reference — Runtime setting indirection

Syntax/selected identity/network must resolve; values never embedded here.

### Secrets User — Key Vault secret-read role

Data-plane reading, not role-assignment write.

### Secrets Officer — Key Vault secret-management role

Secret lifecycle authority, not blanket IAM administration.

### Soft delete — Recoverable deletion retention

Protects against immediate loss; inspect purge protection separately.

### Purge protection — Prevents early permanent deletion

Does not protect against all reads/credential misuse.

### VNet — Virtual network

Address/routing/DNS boundary; shared DEV also hosts QA subnets.

### Subnet — Address-space segment

NSG must be in initial request under BASF policy.

### CIDR — Classless Inter-Domain Routing

Prefix represents IP range; subnet must fit VNet and not overlap.

### NSG — Network security group

Stateful prioritized layer-3/4 filtering, not HTTP auth/WAF.

### Service tag — Managed address group

AzureFrontDoor.Backend/ApiManagement/etc.; match purpose and direction.

### Service endpoint — VNet identity to PaaS public endpoint

Not a PE/private NIC.

### PE — Private endpoint

Private NIC to one PaaS subresource; public flag independent.

### Private Link — Private connectivity service

Requires target approval and caller routing/DNS.

### PDNS — Private DNS

Zones/records/VNet links for private name answers; existence alone insufficient.

### Zone group — PE DNS registration association

Connects target endpoint names to appropriate private zones.

### VNet link — Private DNS network association

Makes zone resolvable in the linked VNet context.

### VNet integration — App Service outbound networking

Not automatically an inbound private endpoint/access restriction.

### Delegation — Subnet authorization for managed service

Microsoft.Web/serverFarms for Assistant app integration.

### VNet injection — Managed service network placement

Databricks target pattern, not proved live in captured CAIRO.

### Peering — VNet connectivity relationship

Connected corporate peerings captured; gateway flags independent.

### UDR — User-defined route

Route table control; no route-table resources returned here.

### DNS — Domain Name System

Verify normal target hostname from actual caller network.

### AFD — Azure Front Door

Global edge routing/probing/WAF association.

### FDID — Front Door profile identifier

Header restriction must select intended profile, not merely any Front Door traffic.

### WAF — Web application firewall

Prevention mode needs actual rules/associations to protect intended paths.

### Origin — Edge backend endpoint

Host header/TLS/path/probe must agree with actual app.

### Origin group — Front Door backend group

Health/load-balancing/affinity settings organize origin selection.

### Route — Front Door path-to-origin mapping

Matching does not automatically strip a prefix.

### APIM — Azure API Management

Gateway/API policy engine; no captured APIs on CAIRO instances.

### API Center — API metadata catalog

Different from an executing gateway.

### CORS — Cross-Origin Resource Sharing

Browser response permission; not auth/network control.

### Preflight — Browser OPTIONS permission request

Origin/method/header/credentials contract may fail before actual request.

### TLS — Transport Layer Security

Encrypted channel/certificate identity; HTTPS-only flag separate.

### FTPS — FTP over TLS

Deployment setting, not application HTTP security.

### SCM/Kudu — App Service deployment/diagnostic site

Can have separate access restrictions and build behavior.

### Oryx — App Service build system

Build/dependency restore differs from container deployment.

### WEBSITES_PORT — App Service container port setting

Must match actual bound process port.

### Always On — App Service keep-warm setting

Not a full autoscale/HA/SLA design.

### EP1 — Elastic Premium Function plan tier

Different cost/scaling/runtime model from FC1.

### FC1 — Flex Consumption Function SKU

Callback plans in Germany West Central in snapshot.

### B3 — Basic App Service plan size

Shared OA app capacity; not evidence of production autoscale.

### ACR — Azure Container Registry

Image artifact store with separate pull/push/network grants.

### Image tag — Container version reference

Snapshot build tag not necessarily immutable digest or tested release.

### FastAPI — Python ASGI web framework

OA services' API implementation.

### Uvicorn — ASGI server

Actual bind port must match App Service routing.

### uv — Python environment/package tool

Used in backend image/runtime and private-feed dependency install.

### OpenTelemetry — Tracing/metrics/log instrumentation standard

App instrumentation separate from Azure diagnostic settings.

### App Insights — Application Insights

Application traces/exceptions/dependencies, typically backed by LAW.

### LAW — Log Analytics workspace

Log ingestion/query/retention destination.

### KQL — Kusto Query Language

Azure Monitor/LAW investigation queries, not ADO work-item WIQL.

### Defender for Cloud — Azure security posture/threat platform

Plans/alerts/score require verification, not tags alone.

### Azure Policy — Resource governance engine

Assignments/effects/exemptions/remediation at scopes.

### Audit — Policy visibility effect

Finds noncompliance without denying create/update.

### Deny — Policy enforcement effect

Blocks matching operation; atomic subnet policy illustrates it.

### Modify — Policy property remediation effect

Needs appropriate assignment identity/permissions.

### DeployIfNotExists — Policy deploy remediation effect

Adds required supporting resources; permissions/remediation task matter.

### Initiative — Policy definition set

Bundle mapped to baseline; assignment is separate.

### Exemption — Scoped compliance exception

Approved documented exception, not permanent silent bypass.

### CIS — Center for Internet Security

Technical benchmark controls; not identical to ISO certification.

### ISO 27001 — Information Security Management System standard

Broader process/control certification context.

### RPO — Recovery point objective

Maximum acceptable data loss; backup existence does not verify it.

### RTO — Recovery time objective

Maximum recovery duration; needs restore rehearsal.

### HA — High availability

OA Postgres HA Disabled in snapshot; app plan presence is not resilience proof.

### ADLS Gen2 — Azure Data Lake Storage generation 2

HNS storage hierarchy and filesystem ACLs.

### HNS — Hierarchical namespace

Enables directory/ACL semantics; actual flag must be true.

### ACL — Access control list

Path permissions/inheritance, distinct from Azure control-plane role.

### Default ACL — Inherited path permission template

Affects new descendants; current access ACL controls existing object.

### LRS — Locally redundant storage

Replication in one region; not geo failover.

### GRS — Geo-redundant storage

Different replication durability model; application failover still designed.

### Blob — Object storage data

Original documents/dead letters distinct from DB records.

### $web — Static website container

Public client assets, not credential/config secret store.

### Azure SQL logical server — Managed SQL administration endpoint

Hosts DB resources; not an IaaS SQL VM.

### Flexible Server — Managed PostgreSQL deployment model

Schema/auth/extensions/backup/network still customer tasks.

### Alembic — Database schema migration tool

Version history does not allow-list Azure server extensions.

### pgcrypto — PostgreSQL crypto extension

Required allow-list/install prerequisite in OA migration history.

### pg_trgm — PostgreSQL trigram extension

Supports similarity/fuzzy matching operations.

### fuzzystrmatch — PostgreSQL fuzzy string extension

Additional matching functions; server allowance required.

### ODBC — Open Database Connectivity

Extraction SQL driver/token connection path.

### asyncpg — Async PostgreSQL Python driver

Orchestrator DB connection/SSL/runtime errors.

### Event Grid — Managed event delivery

Publisher/subscriber auth/retries/idempotency separate.

### System topic — Azure resource event source

Storage antimalware source topics are not OA custom topics.

### Custom topic — Application-defined event ingress

egt-orderautomation-* carries extraction completion.

### Dead letter — Undelivered event storage

Existence of destination not proof writes/replay succeed.

### TTL — Time to live

1440-minute event delivery TTL captured; retry semantics depend on errors.

### Idempotency — Safe repeated processing

Required for retries/duplicate uploads/bug creation races.

### Correlation ID — Cross-service investigation identifier

Track document/case/event/order/run without logging secret payloads.

### Databricks SQL — Workspace query service

Retrieves SAP-derived master data; owning workspace external here.

### Git folder — Databricks linked source checkout

Update branch sync does not execute notebook.

### Notebook job — Scheduled/manual compute execution

Requires job/cluster/permissions/parameters/result verification.

### DBC — Databricks archive format

Notebook export/import artifact, not a live job definition by itself.

### IPYNB — Notebook interchange file

Source artifact must be versioned/tested separately from deployment.

### LLM — Large language model

Semantic field extraction; not authoritative master-data identity.

### OCR — Optical character recognition

Text recognition, distinct from business semantic extraction.

### DI — Document Intelligence

Azure layout/document analysis used explicitly by extraction.

### AI Foundry — Azure AI development/deployment platform

Hub/project/account resource models differ.

### AI Hub — Shared AI workspace resource

Dependencies and managed network separate from model account.

### Model deployment — Named inference endpoint/configuration

Model version/SKU/capacity/quota distinct from account.

### Embedding — Numeric text representation

Search dimension/model contracts need compatible index configuration.

### AI Search — Managed search/index engine

Exists live; active OA retrieval use not proved.

### Content Safety — AI content risk classification

Supporting design/library capability; dedicated OA usage unverified.

### Translator — Managed translation API

Capability does not prove every order invokes it.

### GT — Ground truth

Reviewed feedback/matching provider data; exclusion flags affect reuse.

### KNMT — SAP customer-material info record/table

Maps customer material identity to internal material context.

### T005 — SAP country reference/customizing table

Country normalization context; actual external schema lineage verify.

### Material — SAP product/master identifier

Different from customer-written material code/description.

### Partner — SAP business-party identity

Sold-to/ship-to/bill-to and address roles must not be mixed.

### Sales area — SAP sales organization/distribution/division context

Affects partner/material/order eligibility.

### UOM — Unit of measure

Normalize extracted unit against business/SAP reference.

### PO — Purchase order

Customer request input, not the final SAP sales order.

### CPI — SAP Cloud Platform Integration

Integration flows for order/attachment submission and responses.

### S4HANA — SAP enterprise transaction/master system

Target accepted sales orders and source master data.

### mTLS — Mutual TLS

Client certificate trust pattern; not automatic Entra OBO equivalence.

### Webhook — HTTP callback receiver

Validation/auth/idempotency/delivery behavior need independent checks.

### WIQL — Work Item Query Language

ADO bug search/dedup, not Azure log KQL.

### Work item — ADO Boards record

Bug/User Story fields, area/iteration/current sprint permissions.

### Iteration path — ADO sprint/time planning scope

Current-sprint creation differs from across-sprint update query.

### Area path — ADO team/work categorization

Different from iteration/sprint path.

### JSON Patch — Work-item field-operation document

Used with appropriate create/update method and media type.

### Deduplication — Avoid duplicate logical records

Stable title helps, but concurrent check/create still needs durable strategy.


### BASF ? organization/platform owner

Enterprise owner of the CAIRO application landscape; mailbox, IAM, SAP and cloud governance span different responsible teams.

### MSI ? Managed Service Identity

Older naming for managed identity; Logic App action authentication uses ManagedServiceIdentity in the inspected templates.

### SCM_DO_BUILD_DURING_DEPLOYMENT ? App Service build switch

Enables supported deployment-time build/dependency processing for source/ZIP paths; it is not a substitute for a working container build.

### AzureServices bypass ? trusted-service firewall exception

Resource-specific trusted service bypass; it does not mean every Microsoft-hosted pipeline or Azure VM is automatically permitted.

## Troubleshooting vocabulary: common wrong equivalences

- Resource Running is not application health.
- Workflow Enabled is not successful mail processing.
- Git-folder update is not notebook execution.
- Private endpoint is not public access Disabled.
- VNet integration is outbound, not automatically inbound protection.
- Authentication is not authorization.
- Azure Contributor is not vault secret read permission.
- Backend app role is not Azure RBAC role.
- UAMI attached is not UAMI selected for pull/reference.
- WAF Prevention is not proof protective rules are configured.
- Sensitive Terraform display is not secret-free state.
- Frontend 200 is not a verified SAP transaction.
- RG location is not every resource's region.
- Empty returned list is not denied-access evidence.
- A recommended policy is not an assigned/enforced initiative.

## How it connects to other components

The register binds chapter 02's environment map to chapters 03–07's domain detail.
The glossary supports chapter 13 interview preparation and chapters 11–12 operational language.

## Where to look in the Azure Portal / ADO

- Portal global search → exact resource name → verify **Type**, **Resource group**, **Subscription**.
- RG → **Resources** → filter type; resource → **Properties/JSON View**, **Activity log**.
- ADO CAIRO → **Repos/Files**, **Pipelines**, **Library**, **Project settings/Service connections**, **Boards**.
- Databricks workspace and SAP CPI must be opened in their actual owning platform/tenant.

## Interview talking points

- Navigate from a failure to exact resource/environment instead of guessing names.
- Explain PE/PDNS/UAMI/AFD/APIM in one sentence each.
- Explain KNMT/T005/CPI as business integration context.
- Explain OBO/WIQL/idempotency as distinct concerns.
- State what evidence you have and what is unverified.

## Evidence & confidence

Live register: raw_azure.md full resource register/group/type summaries.
Connections: raw_apps.md/raw_iac.md, with external gaps retained.
Terms: standard platform concepts used to explain source/doc/history evidence.
No secret values or signed URLs are included.
