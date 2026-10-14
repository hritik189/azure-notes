# BASF CAIRO Azure Platform Handbook

## TL;DR

- Understand AI Order Automation from customer document to SAP order.
- Follow real resources in `BASF_SC_CAIRO_PI_PROD`, not only the reference architecture.
- Learn Terraform infrastructure and the manual operational work around it.
- Distinguish live state, code implementation, historical fixes and documented intent.
- Use portal paths and interview prompts to explain decisions and troubleshoot.

## Purpose and audience

An evidence-led handbook for the platform engineer on BASF project CAIRO.
The application is called AI Order Automation, AIOA, EC Order Automation or Order Automation.
Azure DevOps organization is `basf4EMEA`; project is `CAIRO`.
The business flow converts unstructured customer orders into reviewed SAP sales orders. [Doc][Code]
Shared AI resources, Assistants Factory and experimental MCP infrastructure are adjacent workloads, not automatically mandatory order-processing hops.

Four views are combined:

1. **Business:** order intake, enrichment, human review and SAP publication.
2. **Runtime:** executing code, persisted data and downstream API calls.
3. **Platform:** network, identity, governance and environment boundaries.
4. **Operations:** deployment ownership, prerequisites, diagnosis and verification.

This is a snapshot, not authorization to mutate production.
No source repository or cloud resource was changed while writing this handbook.
A successful resource enumeration does not prove an application transaction succeeded.

## How to read

### High-level path

Read **01 → 02 → 05 → 06 → 07 → 10**.
Start with business context, locate environments, identify runtime/data services, then trace live routing and end-to-end flows.

### Deep-dive path

Read **02 → 14 → 03 → 04 → 05 → 06 → 08 → 09 → 11 → 12**.
Use chapter 14 as an index while investigating a concrete resource.
Establish caller/network/identity before applying a historical fix.

### Interview path

Read **01 → 13**, then rehearse each chapter's interview bullets.
Prepare three evidence-backed incidents: ACR pull authorization, DB migration/extensions, Key Vault reference/DNS.
Explain what you personally changed, who supplied missing permissions, and how functional verification differed from deployment success.

## Table of contents

| Chapter | Topic |
|---|---|
| [00 README](00-README.md) | Guide, sources, confidence and open questions |
| [01 Executive overview](01-Executive-Overview.md) | Business problem, architecture and service models |
| [02 Subscriptions, environments and naming](02-Subscriptions-Environments-Naming.md) | Locations, counts, tags and boundaries |
| [03 Networking](03-Networking.md) | VNets, subnets, NSGs, endpoints and DNS |
| [04 Identity, security and governance](04-Identity-Security-Governance.md) | Entra, MI, RBAC, Key Vault and Policy |
| [05 Compute and applications](05-Compute-and-Applications.md) | Containers, App Services, Functions, workflows and frontend |
| [06 Data and AI](06-Data-and-AI.md) | Databases, lakes, events, models and SAP data |
| [07 API edge, Front Door and APIM](07-API-Edge-FrontDoor-APIM.md) | Live routes, origins, WAF and gateway history |
| [08 Terraform IaC](08-Terraform-IaC.md) | Roots, modules, state and drift |
| [09 CI/CD Azure DevOps](09-CICD-Azure-DevOps.md) | Pipelines, connections, groups and promotion |
| [10 End-to-end flows](10-End-to-End-Flows.md) | Interactive, email, extraction, SAP and incident flows |
| [11 Manual steps and runbooks](11-Manual-Steps-and-Runbooks.md) | Non-Terraform operational prerequisites |
| [12 Incidents and lessons learned](12-Incidents-and-Lessons-Learned.md) | Troubleshooting and historical fixes |
| [13 Interview preparation](13-Interview-Prep.md) | Questions, answers and experience narratives |
| [14 Resource register and glossary](14-Resource-Register-and-Glossary.md) | Concrete resource index and terminology |

## Source/evidence statement

Snapshot date: **13 October 2026**.
Live collection began at **2026-10-13T06:52:24.582282Z**. [Live]
Historical evidence extends through 12 October. [Chat]
The September 29 call explains failure-notebook work items and mailbox access constraints. [Doc]

| Label | Meaning | Does not prove |
|---|---|---|
| **[Live]** | Safe read-only Azure CLI list/show projections | Functional health, transactions, complete effective access |
| **[Code]** | Inspected source, Terraform, YAML or ARM | That the inspected branch is deployed |
| **[Chat]** | Historical requests, investigations and reported tests | That requests were granted or fixes still persist |
| **[Doc]** | Official architecture, recommendations or meeting transcript | That the intended design is fully deployed |
| **Unverified** | Missing, denied, omitted or conflicting evidence | Absence of the capability/resource |

A source note on a generated inventory heading applies to all fields underneath it.
Concept explanations and proposed verification steps are teaching material, not executed changes.

### Evidence files

| File | Role |
|---|---|
| `doc_aioa.txt` | Official Infrastructure & CI/CD Overview: business/high-level intent |
| `doc_policy.txt` | Policy baseline recommendations, not deployed initiative list |
| `doc_call.txt` | Notebook deduplication and Logic App mailbox meeting |
| `raw_azure.md` | Ground truth snapshot metadata, settings names and query gaps |
| `raw_iac.md` | Shared/OA/module/Assistant Terraform and pipelines |
| `raw_apps.md` | BE/FE/Databricks/Assistant/GenAI application evidence |
| `raw_chats.md` | Historical investigations and source session appendix |

Source files reside in the source session's persistent `files` folder.
Resource identifiers and secret names are included; secret values are not.

## Corrections to the simplified architecture

- [Doc] Describes a frontend container App Service; [Live][Code] OA frontend is a Storage static website behind Front Door.
- [Doc] Puts APIM universally in path; [Live] OA origins point directly to apps and both APIM API lists returned empty.
- [Doc] Describes per-environment VNets; [Live] QA shares the DEV VNet.
- [Doc] Calls production locked down; [Live] OA app ingress permits all, principal apps have `httpsOnly=false`, and many dependencies retain public access.
- [Doc] Describes MI-scoped storage; [Code] orchestrator Blob uses a connection string and extraction Event Grid publishing uses a topic key.
- [Live] CAIRO has no captured Databricks workspace; [Code][Chat] external workspace/SQL usage is evidenced.
- [Chat] PROD intake once had an empty shell; [Live] it now has trigger/actions. A successful mail run still needs evidence.
- [Code] Incident capture initially had no downstream handoff; [Live] action `Run_Databricks_test_job` now exists. Its name does not prove ADO bug creation.

## Open questions and unverified items

1. Which subscription owns Databricks DEV/QUAL, and what clusters/jobs/VNet injection are active there?
2. Which Git credential is authorized for each pipeline SP within each workspace?
3. Which delivery branches supply QA/PROD tfvars, and which state owns adopted resources?
4. Is APIM intentionally unused by OA, or awaiting publication/cutover?
5. Why does FE `.env.prod` name an older/different edge than the live PROD endpoint?
6. What are effective runtime AcrPull/Secrets User grants? Captured IAM is partial.
7. Do current runtime vault references resolve under the intended UAMI?
8. Are required PostgreSQL extensions allow-listed on every active OA server?
9. What is the current real function-route health after the PROD 503 incident?
10. Are all PEs approved and DNS records/private answers correct from actual callers?
11. What WAF protection is approved? OA policies show no captured managed/custom rules.
12. How will public data planes close without breaking hosted agents/non-integrated apps?
13. Which Defender plans, SOC routing, exemptions and compliance scores are active?
14. Does email-to-bug complete with durable idempotency and correct sprint placement?
15. Are Content Safety/Translator external/shared? Dedicated CAIRO accounts were not identified.
16. Are ARGUS reads intentionally unavailable? Counts remain unknown.
17. Which callback/test/legacy resources can safely retire? Similar names are insufficient evidence.

## How it connects to other components

Use chapter 14 to find a resource, then its domain chapter to trace dependency and ownership.
Use chapter 12 before interpreting a browser symptom as root cause.

## Where to look in the Azure Portal / ADO

- Azure Portal → **Subscriptions** → CAIRO → **Resource groups** / **Resources**.
- Resource → **Activity log**, **Access control (IAM)** and service networking blade.
- ADO → basf4EMEA → CAIRO → **Repos**, **Pipelines**, **Library**, **Boards**.
- Databricks → **Workspace**, **Jobs & Pipelines**, **Compute**, **SQL Warehouses**, **Settings**.

## Interview talking points

- Separate documented target from deployed state and source implementation.
- Explain production-named subscription hosting DEV/QA/PROD.
- Explain why Terraform success is not business success.
- Describe mailbox, Git credentials, DB parameters, IAM and runtime-reference work.

## Evidence & confidence

High confidence for captured metadata and source behavior; historical persistence needs recheck.
Denied/omitted queries remain unknown.
No secrets, keys, passwords, tokens, SAS, signed callback URLs or connection-string values are reproduced.
