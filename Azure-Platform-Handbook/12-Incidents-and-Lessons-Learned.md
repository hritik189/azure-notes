# 12 — Incidents and lessons learned

> **Evidence and caution:** `[Chat]` reports historical Copilot-session investigations from
> 2026-07-03 through 2026-10-12; session snippets may be truncated and prior diagnoses are not fresh
> live checks. `[Live]` is 2026-10-13 inventory; `[Code]` is inspected repository config; `[Doc]` is
> architecture/transcript. “Verified” means historical notes report a concrete check/fix, not that
> it remains fixed. Credentials and sensitive identifiers are excluded.

## TL;DR

Recurring AIOA failures were boundary failures: RBAC vs data-plane roles, identity vs network
reachability, Terraform vs portal/CD ownership, frontend symptoms vs backend health, and successful
deployment vs healthy runtime. CORS masked missing DB schema and ACR image-pull authorization; a
vault 403 persisted after human RBAC because pipeline SP/runtime network were separate; Databricks
Git sync did not imply notebook jobs ran; ARM deployment did not imply a working Office 365
connection. `[Chat]`

## 1. Networking, policy and DNS

### 1.1 APIM VNet activation stalled (July 2026)

- **Issue:** `apim-sharedresources-dev` remained `Creating`/failed Internal VNet activation;
internal callers could not reach gateway. `[Chat]`
- **Checked:** SKU/VNet mode/subnet, NSG/effective route, APIM dependencies, subnet policy and
client DNS/network. Portal test also lacked permission for subscription/key operations. `[Chat]`
- **Root cause:** APIM activation depends on subnet NSG/service dependencies, including TCP 3443
management and Azure Load Balancer probes; internal gateway requires reachable VNet/DNS. Public
laptop failure alone did not show service outage. `[Chat]`
- **Fix/verification:** Terraform NSG/service rule correction and subsequent activation check were
the work path; long provisioning remained caveat. Current checked HCL/live inventory show APIM
External, so historical Internal intent is not current proof. `[Chat][Code][Live]`
- **Portal:** APIM → Overview / Network / APIs; VNet → Subnets / NSG; Private DNS zones; test from
approved VNet/VPN. `[Chat]`
- **Prevention:** Keep APIM NSG matrix/runbook; test from in-network client; review exposure and
subnet capacity before switching External/Internal.

### 1.2 Policy blocked subnet creation; Assistant partial apply (September 2026)

- **Issue:** Deny policy required NSG on subnet creation; Assistant partial apply created resources
but failed on eight subnets, duplicate DNS links, KV authorization, global names and address
space. `[Chat]`
- **Checked:** Policy evaluation, atomic subnet payload, CIDR vs VNet, DNS links, vault auth model,
storage/Speech names, deployment operations. `[Chat]`
- **Root cause:** NSG attached after subnet creation rather than in atomic request; some ranges
outside/colliding with VNet; duplicate links and module access-mode mismatch caused separate
failures. `[Chat]`
- **Fix/verification:** Local Assistant AzAPI subnet module attached NSG initially; DEV ranges
changed; duplicate shared DNS links reused; KV RBAC/names adjusted. `terraform validate` passed;
online plan blocked by Conditional Access. Cleanup, PE approvals, FDID and QA/PROD parity remained
open. `[Chat]`
- **Portal:** Policy → Compliance/Assignments; RG → Deployments; VNet → Address space/Subnets;
Private DNS zones → VNet links; KV → Access configuration/IAM; PE target → Connections. `[Chat]`
- **Prevention:** Preflight policy/CIDR, atomically create subnet+NSG, reuse shared DNS, distinguish
validate from plan/apply/health evidence.

### 1.3 VNet overlap, race and private DNS

- **Issue:** Shared and app roots referenced `10.0.2.0/24`; concurrent APIM/VNet apply hit 409;
stale DNS A record remained after KV recreation. `[Chat][Code]`
- **Root cause:** Duplicate state ownership and concurrent mutation collided; record retained old PE
address. `[Chat]`
- **Fix:** Reconcile one state owner per subnet; serialize VNet mutation; compare DNS record to
current PE IP and delete only after zone-owner/consumer review. `[Chat]`
- **Status:** App root still hardcodes `snet-order_automation-dev` / `10.0.2.0/24`; inventory has
many private zones/links. Recheck before reuse. `[Code][Live]`
- **Prevention:** CIDR ledger, single Terraform owner, DNS test from target workload subnet—not
workstation.

## 2. Identity and RBAC

### 2.1 Key Vault role assignment 403 during Terraform/app access

- **Issue:** Human could access vault, but IaC pipeline secret read/role assignment or App Service
secret access got 403. `[Chat]`
- **Checked:** Vault auth mode, pipeline principal, human PIM, app UAMI, role scope, firewall/public
access, PE DNS and VNet integration. `[Chat]`
- **Root cause:** Human Contributor/Key Vault role does not grant pipeline SP access; Key Vault
Administrator does not grant role-assignment write; runtime UAMI is a third identity. Private
vault requires route/DNS too. `[Chat]`
- **Fix/verification:** Group-based KV Administrator enabled human secret writes; pipeline principal
separately needed Secrets User; app UAMI separately needed runtime role. Two DEV admin passwords
were created in KV and removed from plaintext tfvars. Later pipeline still lacked its own data
role, so not fully fixed until pipeline succeeded. `[Chat]`
- **Portal:** KV → Access configuration / IAM / Networking / Secrets; Entra PIM; ADO Project
settings → Service connections; App Service → Identity / KV reference status / Networking.
`[Chat]`
- **Prevention:** Maintain human/CI/workload identity matrix with exact scope and network route;
validate actual caller. Never widen role for wrong principal.

### 2.2 App Service selected wrong identity / malformed KV reference (September)

- **Issue:** Runtime reference failed even after a UAMI grant. `[Chat]`
- **Root cause:** Site selected SystemAssigned or wrong `keyVaultReferenceIdentity`, and malformed
reference strings were missing final closing parenthesis. `[Chat]`
- **Fix/verification:** Corrected syntax/identity, refreshed/restarted app; historical notes say new
uploads succeeded. Affected cases with null storage metadata needed re-upload. `[Chat]`
- **Portal:** App Service → Identity; Configuration → KV references; Log stream; App Insights
exceptions; Blob containers/case records. `[Chat]`
- **Prevention:** Verify reference resolved status and selected identity per setting; perform
end-to-end storage upload after restart.

### 2.3 Production app role mismatch (August)

- **Issue:** User authenticated but FE showed Pending/forbidden. `[Chat]`
- **Root cause:** Auto-created user lacked required AIOA application role
(`system_admin`/`system_user`) or country/region scope; Azure subscription role was unrelated.
`[Chat]`
- **Fix/verification:** Assign approved app role via app admin flow; validate protected API and
audit. `[Chat]`
- **Portal:** Entra sign-in logs prove login; AIOA user/role admin path is where application
authorization lives. `[Code][Chat]`
- **Prevention:** Document separate Entra login, Azure RBAC and application-role onboarding.

## 3. Key Vault and secrets

### 3.1 Secret migration and vault recreation/drift

- **Issue:** SQL/PostgreSQL admin passwords were in IaC inputs/old vault; recreating vault hit
soft-delete/name reservation and stale DNS. `[Chat]`
- **Root cause:** Secret lifecycle mixed with infra config; purge protection reserves deleted name;
Terraform data-source values remain in state even if sensitive. `[Chat][Code]`
- **Fix:** Historical change created DEV secrets `sql-admin-password` and `postgresqlpassword`;
SQL/PG resources read KV; plaintext tfvars removed and development branch later updated. `[Chat]`
- **Verification/limits:** Secret creation and merge reported; a later pipeline 403 was a separate
blocker. Recheck runtime access. `[Chat]`
- **Portal:** KV → Secrets / Deleted secrets / Networking / IAM; state backend; Activity Log.
`[Chat]`
- **Prevention:** Prefer runtime references where possible; restrict state/artifacts; version and
rotate secrets; plan recover/recreate before replacing vault.

### 3.2 Hosted agent could not reach ACR or vault

- **Issue:** Microsoft-hosted agent image push/lookup returned 403 after public endpoint restricted.
`[Chat]`
- **Root cause:** PE routes only connected VNets; hosted agent lacked private DNS/routing.
Credentials cannot fix network denial. `[Chat]`
- **Fix:** History includes hosted-agent service address allowances and registry/KV public setting
adjustments; historical workaround is not necessarily current safe posture. `[Chat]`
- **Verification:** `acrsharedresourced2slfk` was recorded publicNetworkAccess Disabled at 08:15 UTC
July 23; Docker login/push failed. Later posture must be rechecked. `[Chat]`
- **Portal:** ACR/KV → Networking and PE; ADO → Agent pools; verify caller and egress. `[Chat]`
- **Prevention:** Prefer VNet-connected agents for private data-plane resources or approved narrow
egress; avoid broad allowances.

## 4. Pipelines and container deployment

### 4.1 CORS/500 masked DB schema and image pull (July)

- **Issue:** FE `/api/regions` showed CORS/500/503. `[Chat]`
- **Checked:** Browser headers, backend logs, PostgreSQL schema/seed, container startup/image pull
and CORS. `[Chat]`
- **Root cause:** Alembic migrations had not run, tables/regions were empty, and image pull failed
ACR auth; errors lacking CORS headers masked roots. `[Chat]`
- **Fix/verification:** Apply migration or intended startup migration; grant runtime UAMI AcrPull;
configure exact FE origin; verify `/api/regions` 200 and authenticated FE flow. Historic fix path
described, not current guarantee. `[Chat]`
- **Portal:** App Service → Deployment Center/Identity/Configuration/Log stream; ACR →
IAM/Networking; PG → Databases; App Insights → Failures; browser network panel. `[Chat]`
- **Prevention:** Health gates include DB migration/schema and API, not only static home page or
deployment status.

### 4.2 ACR pull authorization / pipeline resets MI (July–August)

- **Issue:** Image present but App Service reported `ImagePullUnauthorized`; hosted agent also
failed push. `[Chat]`
- **Root cause:** PROD UAMI lacked AcrPull; deployment CLI credentials reset
`acrUseManagedIdentityCreds=false`. Build push and app pull are separate identities/network paths.
`[Chat]`
- **Fix/verification:** Grant AcrPull to correct UAMI on correct registry, select UAMI, preserve MI
config in CD and restart; verify tag, container logs, health. `[Chat]`
- **Portal:** ACR → IAM/Networking/Repositories; App Service → Identity/Container
settings/Deployment Center/Log stream; Activity Log. `[Chat]`
- **Prevention:** One MI auth model, one owner for container config, test per-environment registry.

### 4.3 ZIP/Oryx and port mismatch (July–August)

- **Issue:** Linux Kudu ZIP deployment failed or service unhealthy. `[Chat]`
- **Root cause:** ZIP used Windows path separators; Oryx build not enabled; configured
`WEBSITES_PORT` differed from listener (8080 vs 8000); Rules service listened on 8001. `[Chat]`
- **Fix/verification:** POSIX-compatible archive, enable build if needed, match
startup/listener/probe, inspect deployment logs; portal rule-port change could be reset by
Terraform. `[Chat]`
- **Portal:** App Service → Kudu/Deployment Center/General settings/Configuration/Log stream; AFD →
origin health. `[Chat]`
- **Prevention:** Per-service port contract, image smoke test, explicit app-setting owner.

### 4.4 Private feed failure in Docker build (October)

- **Issue:** `uv sync --frozen --no-dev` failed at Dockerfile line 37 accessing
`CAIRO_GenAI_Artifacts`. `[Chat]`
- **Status/root cause:** Feed reader permission or secure credential propagation was missing;
successful restore was not evidenced. **Unresolved.** `[Chat]`
- **Prevention:** Grant pipeline feed read, use short-lived build secret mount rather than persisted
Docker ARG/layer, validate package download in same agent context.
- **UI:** ADO Artifacts → feed permissions; pipeline build logs; never echo token.

## 5. Terraform drift and resource adoption

### 5.1 APIM adoption/recreation and subnet reconciliation

- **Issue:** APIM adoption/recreation and subnet address migrations repeated in shared history.
`[Code][Chat]`
- **Root cause:** Existing Azure resources/state ownership, azurerm/AzAPI address changes and
ForceNew APIM inputs made apply risky; policy and concurrent updates added failure modes.
`[Code][Chat]`
- **Fix:** Import subnet to AzAPI state, use `removed { destroy=false }` for legacy addresses, adopt
APIM deliberately and reconcile mode/network before plan. Preserve transient import until plan
confirms no destructive action. `[Code]`
- **Verification:** Current `imports.tf` imports APIM/MLOps subnets only; no APIM import block
despite historical APIM adoption commits. `[Code]`
- **Portal:** IaC plan artifact; APIM Overview/Network; VNet subnet/NSG; Activity Log.

### 5.2 Key Vault replacement and ignored firewall drift

- **Issue:** KV name could not be recreated immediately; app HCL ignores network ACL changes.
`[Code][Chat]`
- **Root cause:** Soft-delete/purge protection reserves name; PE/DNS may stale; `ignore_changes`
suppresses ACL reconciliation. `[Code][Chat]`
- **Prevention:** Check deleted vaults; recover if possible; plan replacement name with consumer
migration; assign explicit owner to firewall and audit portal changes.

## 6. APIM, AFD, CORS and direct-origin issues

- APIM access from outside VNet failed while Internal mode was expected; current checked HCL/live
inventory instead show External. Gateway reachability follows mode/network/DNS, not resource
`Succeeded`. `[Chat][Code][Live]`
- FE CORS symptom hid backend 500; later one repair added FE AFD origin. Check actual
preflight/response, AFD route, App Service logs, DB and auth before CORS changes. `[Chat]`
- Direct App Service origin restrictions were conditional on Front Door ID inputs; current DEV
extraction config showed allow-all. Avoid assuming the origin is protected because AFD is
configured. `[Code][Live]`
- **Portal:** AFD → Routes/Origins/Health/WAF; App Service → CORS/Logs/Networking; APIM →
APIs/Policies/Network; App Insights → Exceptions.
- **Prevention:** Use exact allowed origins; never pair wildcard CORS with credentials. Verify
direct-origin access and WAF logs.

## 7. Logic Apps, Office 365 and incident automation

### 7.1 Workflow shell existed but PROD flow was empty

- **Issue:** PROD workflow reported Enabled but no triggers/actions/runs; IaC created shell only.
`[Chat]`
- **Root cause:** Terraform shell, ARM definition, O365 connection authorization and mailbox
delegation are separate. `[Code][Chat]`
- **Fix/status:** DEV/QA availability checked; PROD workflow definition/mailbox connection needed
inspection. Email test `$connections` binding fix deployed 2026-10-05; post-deploy send test not
evidenced. `[Chat]`
- **Portal:** Logic App → Designer/Trigger history/Runs; API Connections → Authorization; ADO ARM
pipeline.
- **Prevention:** Acceptance is successful trigger + connector action + received test mail, not ARM
deployment success.

### 7.2 Email incident capture vs ADO bug creation

- **Issue:** Expected email failure → ADO bug with dedup, but components were at different
completion levels. `[Chat][Doc]`
- **Root cause:** DEV `email-to-bug` captures mail only; Databricks notebook separately performs
WIQL/current iteration/title dedupe and work-item create/comment. Durable state/job execution not
established. `[Code][Chat]`
- **Fix/status:** Sept 29 transcript describes current sprint title check, cross-sprint duplicate
updates/comments and creation only when none exists. IMAP/EWS code path was blocked by Office 365,
so Logic App connector used. End-to-end production handoff remained unproven. `[Doc][Chat]`
- **Prevention:** Define trigger→durable capture→idempotent issue update→failure handling; provision
ADO identity/area/iteration rights; run safe test project.

## 8. Database and environment promotion incidents

### 8.1 PostgreSQL extension allowlist / production connectivity

- **Issue:** Migration errors for missing extensions and App Service DB connection timeout.
`[Doc][Chat]`
- **Root cause:** Flexible Server extension allowlist is server-level and per environment; PROD
client path/firewall did not permit expected access. Broad firewall was emergency mitigation.
`[Chat]`
- **Fix/verification:** Allow `pgcrypto`, `pg_trgm`, `fuzzystrmatch` on each server, apply
migrations and test from service. Emergency `AllowAll` was reported to fix connectivity but is not
a safe permanent posture. `[Doc][Chat]`
- **Portal:** PostgreSQL → Server parameters/Networking/Databases/Logs; App Service → VNet
integration/Log stream.
- **Prevention:** Include extension + DNS + route checks in env readiness; server allowlist is not
created by schema migration.

### 8.2 QA selected wrong branch/group or stale config

- **Issue:** QA pipeline could use DEV values; trigger names differed; QA YAMLs were renamed PROD in
a feature branch. `[Chat]`
- **Root cause:** Pipeline registration/default branch, selected YAML, trigger, variable group and
tfvars are independent. Explicit endpoints in config survive environment label changes.
`[Code][Chat]`
- **Fix/status:** Inspect exact run branch/group; restore/verify QA definitions; do not rely on a
stopped session. ADO live registration could not be listed. `[Chat][Live]`
- **Prevention:** Safe preflight prints intended environment/resource group; use protected
Environment checks; immutable image promotion.
- **Portal:** ADO Pipelines → Run branch/YAML; Library → groups; Repos → Branches/PR; Environments
and Service connections.

## 9. Databricks and AI service incidents

### 9.1 Databricks Git-folder sync blocked

- **Issue:** Workspace import/push was changed to Git-folder pull; repo ID, workspace, Entra auth
and Git credential required. `[Chat]`
- **Root cause:** Four layers differ: Azure service connection auth, Databricks SP entitlement,
workspace folder ACL, Azure Repo Git credential. Entra token used by pipeline is not automatically
Git clone credential. `[Chat]`
- **Fix/status:** CLI syntax changed to positional `repos update <repo-id> --branch`; repo/workspace
access addressed; last evidence still reported AAD Git credential error. Notebook/job execution
unverified. `[Chat]`
- **Portal:** Databricks Admin → SP/workspace; Repos → Share; Settings → Linked accounts; ADO repo
permissions; pipeline logs.
- **Prevention:** Test `repos get`, identity's Git credential, branch/repo root and ACL, then
execute a job and validate output.

### 9.2 AI 401/403 and key vs role behavior

- **Issue:** AI service-principal call returned PermissionDenied while API key worked; Foundry
deployment/region/quota were separate. `[Chat]`
- **Root cause:** Cognitive Services/OpenAI data-plane role differs from control-plane Contributor;
audience, role, endpoint, model deployment and quota are separate. `[Doc][Chat]`
- **Fix:** Grant approved Cognitive Services OpenAI User/Cognitive Services User at required scope
or use approved key auth; check model endpoint/deployment. `[Doc][Chat]`
- **Portal:** Cognitive Services/Foundry → IAM/model deployments/network/diagnostics; Entra sign-in
logs.
- **Prevention:** Document auth per caller and map model/region/quota; rotate keys if key path
retained.

## 10. Functions, observability and frontend

### 10.1 Production Function returned 503 while resource was Running

- **Issue:** Sept 23 Function App was Running/deployment succeeded but root/health returned 503.
`[Chat]`
- **Checked:** Worker runtime config, health path, deployment package and App Insights. `[Chat]`
- **Root cause/status:** Missing runtime settings/health diagnostics suspected; exact host startup
cause **not confirmed**. `[Chat]`
- **Portal:** Function App → Diagnose and solve problems/Configuration/Deployment Center/Log
stream/Application Insights.
- **Prevention:** Add worker/health diagnostics and verify function route invocation, not resource
provisioning state.

### 10.2 Frontend auth/environment/malformed references

- Vite environment is embedded at build; PROD FE 200 did not prove correct Entra/API config. Rebuild
with production env and verify `/api` host. `[Chat]`
- `ENVIRONMENT=dev` was rejected by runtime accepted values; explicit QA URLs/storage strings caused
cross-env behavior. `[Chat]`
- SPA deep links require `index.html` error fallback. `[Chat]`
- **Prevention:** Verify environment, API endpoint, tenant/client, SPA fallback and authenticated FE
flow per artifact.

### 10.3 Observability coverage gaps

- `[Code]` IaC diagnostics route only extraction App Service, Function, SQL Server/database; live
LAW/App Insights are not proof all services send telemetry.
- `[Chat]` App Insights query was blocked by certificate chain; no result is not proof no telemetry
exists.
- **Prevention:** Add diagnostic settings for Logic Apps, KV, PostgreSQL, Event Grid, Storage,
AFD/WAF, Rules and Orchestrator; test alert receiver and correlate by case/request ID.

## 11. Cost, policy and governance lessons

- `[Doc]` Policy recommendations propose Audit-first CIS/ISO initiatives and phased Deny/Modify.
They are recommendations, not evidence of active assignment.
- `[Code]` Shared Azure Policy assignments are commented; inspect subscription directly.
- `[Chat]` Defender/SOC work was constrained by subscription permissions; state removal to avoid 403
does not enable Defender/alert routing. Secure Score story target is not persistent coverage
proof.
- `[Chat]` Emergency public/allow-all firewall mitigated outage but creates security debt. Record
owner/expiry and replace with private path.
- **Prevention:** Budget/tag by environment, review SKUs/idle services, verify backups/restore and
RPO/RTO. Inventory does not prove DR test coverage.

## 12. Selected historical session catalog

Selected summaries from a 516-session corpus; not every conversation. Outcome is historical
evidence, not present status. `[Chat]`

| Date (2026) | Topic | Outcome / caveat |
|---|---|---|
| Jul 8–10 | ACR auth, storage static site checks | Agent/network and storage checks; current posture needs recheck. |
| Jul 10 | CORS 500 investigation | Root DB/migration and image pull; CORS masked backend failures. |
| Jul 14–15 | APIM endpoints/internal activation | NSG/VNet investigated; current mode differs. |
| Jul 20–22 | Defender/SOC coverage | Story scoped; durable IaC/alert routing unproven. |
| Jul 23 | ACR private endpoint/hosted agent | Public network disable caused login/push 403. |
| Jul 27–Aug 5 | FE/API CORS and service ports | Portal fixes; drift risk. |
| Aug 13–20 | QA promotion and Docker release | Branch/group/config mismatch; verify env values. |
| Aug 25 | PROD FE/PostgreSQL/roles | Missing AcrPull, DB network/extensions and app role issues; emergency firewall. |
| Aug 27–31 | Logic App email; extraction 409 | ARM binding needed actual test; extraction 409 may mean already processing. |
| Sep 9 | App settings, storage refs, SPA | Environment/config and fallback fixes reported. |
| Sep 11 | Malformed references/wrong UAMI | Syntax/identity fixed; some failed cases required re-upload. |
| Sep 23 | Function 503 | Root cause unresolved; runtime/health diagnostics checked. |
| Sep 25 | Assistant partial IaC apply | Subnet/DNS/RBAC/name fixes; online plan/PE approvals/env parity open. |
| Sep 2026 | Databricks Git automation | Sync path built; later Git credential failure; job run unproven. |
| Sep 29 | Failure notebook meeting | WIQL/sprint/dedupe/comment/create behavior described; live flow unproven. |
| Oct 5 | Logic App `$connections` fix | ARM fix deployed; post-deploy mail test not recorded. |
| Oct 11 | Private Python feed build | `uv sync` failure unresolved in captured evidence. |
| Oct 12 | Platform inventory/access | Azure inventory gathered; ADO listing denied `TF400813`. |

## 13. Root-cause patterns

1. Separate identity, role scope, network route and DNS; any one can independently block.
2. Separate control plane from data plane; Azure Contributor is not secret read or blob data access.
3. Give one owner to each resource/setting: Terraform, app CD or manual with expiry and audit.
4. Deployment success, Running state, image presence or static homepage 200 are not app health.
5. Environment suffix is not isolation; subscription/branch/tfvars/group/host/identity/DB must
align.
6. Diagnose CORS/API symptoms by checking backend status/auth/schema before changing headers.
7. Async flows need correlation, idempotency, durable status, retry/dead-letter and replay.
8. Preserve uncertainty: some items were blocked, drafted, or not live-verified.

## Where to look in Azure Portal / ADO

- Portal → Resource → Activity Log / Deployments / Diagnose and solve problems.
- VNet → Subnets/NSGs/Private DNS; PE target connection and zone group.
- APIM → Network/APIs/Diagnostics; Front Door → Origin health/routes/WAF; App Service →
Identity/Configuration/Log stream.
- KV → Secrets/Networking/Access configuration/IAM; ACR → Networking/IAM; PostgreSQL → Server
parameters/Networking/Logs.
- Logic Apps → Trigger history/Runs/API connections; Databricks → Repos/Jobs/Identity; Azure Policy
→ Assignments/Compliance; Defender for Cloud → Regulatory compliance.
- ADO → Pipelines/Runs; Library/Variable groups; Project settings/Service connections/Agent pools;
Environments/Approvals; Repos/Branch policies.

## Interview talking points

- Tell CORS story as layered investigation: browser symptom → backend 500 → schema/migration and
image pull, not “CORS was the root cause.”
- Tell Key Vault 403 story separating human, pipeline, runtime identity plus network/DNS.
- Explain why Terraform success does not prove PE approval, O365 consent, app health or DB extension
setup.
- Show how undocumented portal fixes become ownership drift and how to make verification repeatable.
- State clearly that Function 503 and live ADO registration status remain evidence gaps.

## Evidence & confidence

- Chat summary enumerated 516 historical sessions (July 3–October 12); excerpts can be truncated.
`[Chat]`
- Resource inventory is timestamped 2026-10-13; HCL/app snapshot may differ from deployment.
`[Live][Code]`
- Policy file is a recommendation doc; do not call its assignments active. `[Doc]`

## Further reading

- [08 Terraform/IaC](08-Terraform-IaC.md) · [09 CI/CD](09-CICD-Azure-DevOps.md) · [10 End-to-end
flows](10-End-to-End-Flows.md) · [11 Runbooks](11-Manual-Steps-and-Runbooks.md)
