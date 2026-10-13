# Azure App Service

*Azure's fully managed platform for hosting web apps, REST APIs, and mobile backends. You provide the code (or a container); Azure runs, scales, patches, and load-balances it.*

## 1. The Problem It Solves

Before managed platforms, hosting a web application required handling all of the following yourself:

1. Provision a server (physical, then later a VM).
2. Install and continually patch the operating system, including security updates.
3. Install the web server (IIS, Nginx) and the language runtime (.NET, Node, Python).
4. Obtain and configure TLS certificates for HTTPS.
5. Set up a load balancer and a second server so a single failure does not cause downtime.
6. Manage scaling — add capacity before traffic spikes and remove it afterward.

None of these tasks are the application itself; they are infrastructure plumbing common to every web app. App Service removes this work. You supply the application; Azure handles items 1–6.

This positions App Service on the spectrum of how much the platform manages on your behalf:

```
You manage everything ◄──────────────────────────────► Azure manages everything
   IaaS                         PaaS                          FaaS
   (Virtual Machine)       (App Service)                  (Functions)
   you patch the OS,       you provide code,              you provide one
   install the runtime,    Azure runs it on               function, Azure
   manage scaling          managed servers                runs it on demand
```

App Service is the **PaaS** middle ground: more control than serverless, far less operational overhead than a virtual machine.

## 2. What App Service Is

App Service runs your application on a pool of virtual machines that Azure owns and operates. You do not have direct access to those machines — there is no SSH login, no OS patching, and no load balancer for you to configure. Azure manages the infrastructure; you manage only the application and its configuration: app settings, scaling rules, and deployments.

Because you do not administer the machines, everything you control is expressed as configuration. That configuration is built on a single relationship, described next.

## 3. How It Works — Plans and Apps

App Service is built on two objects. Everything else follows from them.

- **App Service Plan** — the compute resources you rent: a region, an operating system (Windows or Linux), an instance size, an instance count, and a pricing tier. The plan is what you pay for.
- **App (Web App)** — your application code running on a plan.

A single plan can host multiple apps, and those apps share the plan's CPU and memory:

```
App Service Plan   ← the rented VMs (region · OS · size · count · tier)
   ├── App: storefront      ┐
   ├── App: admin-portal    ├─ all three share the plan's CPU/RAM
   └── App: internal-api    ┘
```

This shared compute enables cost savings (consolidating low-traffic apps onto one plan) but also introduces resource contention (a busy app can consume capacity needed by others — see Gotchas).

## 4. Pricing Tiers

You do not choose a specific VM model; you choose a **tier** on the plan. The tier determines both the power of the instances and which features are available:

| Tier | Compute | Scale out | Slots | Storage | Key features |
|---|---|---|---|---|---|
| **Free (F1)** | Shared, 60 CPU-min/day | none | 0 | 1 GB | testing only; no custom domain or TLS |
| **Shared (D1)** | Shared, 240 CPU-min/day | none | 0 | 1 GB | custom domains |
| **Basic (B1–B3)** | Dedicated | manual, ≤3 | 0 | 10 GB | custom domains + TLS, **VNet integration** |
| **Standard (S1–S3)** | Dedicated | **auto**, ≤10 | 5 | 50 GB | autoscale, deployment slots, daily backups |
| **Premium (Pv3)** | Faster CPU, more RAM | auto, ≤30 | 20 | 250 GB | **private endpoints**, faster storage |
| **Isolated (I1–I3)** | Dedicated environment (ASE) | ≤100 | 20 | 1 TB | full network isolation, single-tenant |

Two points to remember: autoscale and deployment slots both begin at the **Standard** tier. **VNet integration (outbound) is available on any dedicated tier — Basic and above**, whereas inbound **private endpoints** require **Premium v2/v3**. Free and Shared run on shared compute and have none of these features, so dev/test on them does not reproduce production behavior.

**Cost model:** you pay for the **plan** — billed per instance-hour for the chosen tier × instance count — *whether or not the app receives any traffic*. Stopping the app does not stop billing; only scaling the plan down or deleting it does. Multiple apps on one plan share that single bill (the most common source of "why is my idle app costing money").

## 5. Scaling

A plan has both an instance size and an instance count, and each can be changed independently.

**Scale up (vertical):** move to a higher tier for more CPU/RAM per instance and additional features. Causes a brief restart. Use when individual requests are resource-intensive or you need a tier-gated feature.

**Scale out (horizontal):** add more instances behind the load balancer. No downtime. Use when concurrent request volume increases.

- **Manual scaling** — you set a fixed instance count. Available at Basic and above. Suitable for predictable load.
- **Autoscale** — Azure adjusts the instance count automatically. Available at Standard and above. Two trigger styles:

**Metric-based** — scale on observed load:

| Metric | Scale out when | Scale in when |
|---|---|---|
| CPU percentage | > 70% | < 30% |
| Memory usage | > 80% | < 40% |
| HTTP queue length | > 10 | < 5 |
| Custom (App Insights / external) | app-defined | app-defined |

**Schedule-based** — scale on time, for known patterns:

```
Business hours (Mon–Fri 9–18):  5 instances
Evenings / weekends:            2 instances
Peak event (e.g. sale day):     10 instances
```

**Autoscale rule anatomy:** metric source → threshold → action (scale out/in by N) → **cooldown** (wait before the next action) → **min/max** instance bounds. Always set a maximum (a cost ceiling so a spike or retry storm cannot scale without limit) and a cooldown (prevents flapping — repeated scale-out/scale-in on one spike). Keep the scale-out threshold more sensitive than scale-in so capacity is not removed while load is still high.

## 6. Deployment Slots

A **slot** is a separate, live copy of the app on the same plan, each with its own hostname (e.g. `myapp-staging.azurewebsites.net`). Available at Standard and above. Slots let you deploy and validate a new version in a production-identical environment, then move it into production without downtime.

**Which settings move during a swap** is the detail that causes the most production incidents:

| Setting type | Behavior on swap |
|---|---|
| Standard app settings, general config, handler mappings, monitoring settings | **Move with the app** (follow the code) |
| Settings marked **"slot setting" (sticky)** — slot-specific connection strings, custom domains, TLS bindings, auth settings | **Stay pinned** to their slot |

Mark every environment-specific value (database connection string, keys, feature flags) as a slot setting; otherwise the staging database connection string moves into production during the swap.

## 7. How a Slot Swap Works

A swap is not a redeploy — it is a routing change, which is why it causes no downtime. Azure runs three phases:

1. **Pre-swap (warmup + validation)** — the target (staging) instances are restarted with the destination (production) configuration applied, then warmed by sending startup requests. Custom warmup paths and health checks run here. If the app fails to start with production config, the swap is aborted before any traffic moves.
2. **Swap execution** — once the target is warm and healthy, Azure updates internal routing so production traffic is directed to the now-warm staging instances. The two slots exchange their running instances. Because the instances are already warm, users experience no cold start.
3. **Post-swap monitoring** — the swapped app is monitored for health and error rates; if checks fail, you swap back (the previous version is still running in the other slot).

**Swap with preview** splits this into two manual steps: phase 1 applies production config to staging and warms it so you can validate against real production settings; you then complete (phase 2) or cancel. Use it whenever startup depends on configuration.

```bash
# Standard swap
az webapp deployment slot swap -g myRG -n myApp --slot staging --target-slot production

# Swap with preview (phase 1)
az webapp deployment slot swap -g myRG -n myApp --slot staging --target-slot production --action preview
```

## 8. Deployment Strategies

Slots and traffic routing make several release strategies possible.

**Blue-Green** is exactly the slot swap in §7: staging (green) holds the new version, the swap promotes it to production, the old version stays in the other slot, and rollback is swapping back. It is the default slot workflow — nothing beyond §7 to add.

**Canary** — release to a small fraction of users first, then increase gradually. Deploy to a staging slot, then use **slot traffic routing** to send a small percentage (e.g. 10%) of production traffic to it. App Service pins each user to their slot with the `x-ms-routing-name` cookie, so a session stays consistent. Monitor error rate and latency, raise the percentage in steps (10 → 25 → 50 → 100), and complete the swap when confident. To abort, route 0% back. Unlike blue-green's all-at-once cutover, canary shifts traffic incrementally to limit blast radius.

```
1. Deploy new version → staging
2. Route 10% of production traffic → staging
3. Monitor metrics; increase % gradually
4. Complete swap at confidence, or route 0% to roll back
```

**A/B testing** — mechanically the same as canary (split traffic across slots), but the intent is comparing feature variants by user engagement / business metrics rather than validating release safety.

## 9. Deploying Your Code

| Method | When to use |
|---|---|
| **CI/CD (Azure DevOps / GitHub Actions)** | Default for production — build, test, deploy to a slot, swap, with deployment history and rollback |
| **Container** (pull from **ACR**) | App is already containerized; App Service runs the single container without AKS overhead |
| **ZIP deploy** (`az webapp deploy`) | Quick manual deployment or scripts |
| **Local Git** | Push to an Azure-hosted Git remote to deploy |
| **FTP/FTPS** | Legacy workflows or static files into `/site/wwwroot` |

```bash
# ZIP deploy straight to a staging slot
az webapp deploy -g myRG -n myApp --slot staging --src-path ./app.zip --type zip
```

**Deployment best practices:** deploy to a staging slot first and swap; configure a **health check** endpoint so failed instances are removed and swaps can validate; run database migrations as backward-compatible changes so old and new versions can coexist during a swap.

**App Service vs AKS:** for a single container with no orchestration requirements, App Service (Linux, pulling from ACR via managed identity) requires less operational effort. Use AKS when you need multi-container orchestration, a service mesh, or pod-level control.

## 10. Configuration and Secrets

App settings defined on the app are injected into the application as **environment variables** at runtime, so you change behavior (connection strings, flags, endpoints) without redeploying code. They are encrypted at rest, and any of them can be marked as a slot setting (§6).

**Key Vault references** let an app setting point to a secret in Azure Key Vault instead of holding the value directly:

```
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/db-password/)
```

App Service resolves the reference at startup using the app's **managed identity** (§13), so the secret never lives in configuration or source control, and rotating it in Key Vault updates the app without a redeploy. This is the standard pattern for secrets in App Service.

## 11. Kudu / the SCM Site

Every app has a companion **SCM (Source Control Management) site** at `https://<app>.scm.azurewebsites.net`, powered by the **Kudu** engine. It is the machinery behind deployment and the first place a senior engineer goes to debug App Service. Kudu provides:

- the **deployment engine** — receives ZIP/Git/CI pushes, runs the build (Oryx on Linux), and places files in `/site/wwwroot`;
- **log streaming** plus access to deployment and application logs;
- a **debug console** (Bash/PowerShell) into the live instance's file system;
- a **process explorer**, environment-variable inspection, and WebJobs management;
- REST and Git endpoints that CI/CD pipelines call.

Access uses the same credentials/role as the app. On Linux, the app runs in one container and Kudu runs in a separate sidecar container.

## 12. Authentication and Authorization ("Easy Auth")

App Service provides built-in authentication that runs as a layer **in front of** the application, requiring no authentication code. It intercepts each request, validates the token, optionally stores tokens, and passes the user identity to the app through HTTP headers. Because it sits in front, the app can remain authentication-unaware or simply read the injected identity.

**Identity providers:** Entra ID (AAD) for organizational identity (adds MFA, conditional access, multi-tenant, group claims), Microsoft accounts for consumer identity, and social providers (Google, Facebook, GitHub).

**Authorization:** restrict access using app-level roles, Entra ID group membership, or token claims for fine-grained control.

**Reading the identity in code** — the platform injects a base64-encoded JSON principal header:

```javascript
// Node.js — read the authenticated user
app.get('/profile', (req, res) => {
  const header = req.headers['x-ms-client-principal'];
  const user = JSON.parse(Buffer.from(header, 'base64').toString());
  res.json(user);
});
```

## 13. Networking and Security

- **VNet integration** (**Basic+**, any dedicated tier) — *outbound* access: the app reaches private resources (a SQL Managed Instance, a private endpoint) inside your virtual network.
- **Private endpoint** (**Premium v2/v3**) — *inbound* access: the app receives a private IP and is removed from public exposure.
- **IP restrictions / service tags** — allow-list the sources permitted to reach the app.
- **TLS certificates** — free Azure-managed certificates for custom domains, or upload your own and bind them.
- **Managed identity** — pull images from ACR and read secrets from Key Vault (§10) without storing any credentials.

## 14. App Service Environment (ASE)

The Isolated tier runs on an **App Service Environment** — a fully dedicated, **single-tenant** deployment of App Service injected directly into your own virtual network. Unlike the shared multi-tenant service, no other customer's apps run on the same infrastructure, and both inbound and outbound traffic stay inside your VNet. It is used for high scale, strong network isolation, and compliance requirements, and carries a higher baseline cost because the environment itself is dedicated to you.

## 15. Monitoring and Diagnostics

- **Application Insights** — request rates and response times, dependency tracking (DB, HTTP calls), automatic exception capture, and custom business metrics. The primary tool for performance and failure analysis.
- **Diagnostic logging** — application logs, web server (request) logs, detailed error messages, and failed-request tracing.
- **Log stream** — a live tail of the app's logs (also reachable through Kudu); the first place to look when an app fails to start.

## 16. WebJobs (background tasks)

WebJobs run background scripts or programs in the same plan as the app, for work that is not request-driven (queue processing, scheduled cleanup):

- **Continuous** — start when deployed and run constantly (e.g. a queue listener). Require **Always On**, otherwise the host unloads them when the app goes idle.
- **Triggered** — run on a CRON schedule or on demand, then exit.

WebJobs are managed through Kudu. For heavier or independently scaled background work, Azure Functions is usually the better fit.

## 17. Backup and Restore

Standard and above support scheduled and on-demand backups of the app's content and configuration (and optionally a linked database) to a storage account, with restore to the same or a new app. The tier governs backup frequency and retention.

## 18. Performance

- **Always On** — keeps the app warm so it is not unloaded after idle, eliminating cold starts (Basic+; also required for continuous WebJobs).
- **Caching** — output caching for rendered pages, Azure Cache for Redis for distributed/session data, and Azure CDN for static content.
- **Compression and minification** — enable gzip and minify CSS/JS to reduce payload size.
- **Connection pooling** — reuse database connections to avoid per-request connection overhead.

## ⚠️ Gotchas

- **You pay for the plan even when the app is stopped.** Billing is per instance-hour for the tier, regardless of traffic. Scale down or delete the plan to stop charges.
- **Free/Basic tiers do not match production.** No autoscale or slots (and Free/Shared have no dedicated compute), so testing on them does not reflect production behavior.
- **Apps on the same plan share CPU/RAM.** Consolidating apps to reduce cost can let one busy app starve the others. Isolate high-traffic apps on their own plan.
- **Settings not marked "slot setting" move during a swap.** A non-sticky connection string moves into production, pointing it at the staging database. Mark all environment-specific values as sticky.
- **Autoscale without a maximum can lead to unexpected cost.** A traffic spike or retry storm scales to the limit. Always set a maximum instance count.
- **Swapping without warmup causes a cold-start delay.** Use swap with preview or a warmup path so the target slot is warm before it takes traffic.
- **VNet integration is outbound only.** It does not make the app private; that requires a private endpoint. The two are frequently confused.
- **Canary traffic routing needs session affinity.** Without `x-ms-routing-name` pinning, a user can bounce between old and new versions mid-session.
- **Windows App Service runs in a sandbox.** Restricted Win32 APIs, no GDI/font calls, limited registry and drive access. Libraries needing these (some PDF/imaging or COM components) can fail — use Linux, a container, or a VM instead.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| App won't start / 503 on boot | Wrong startup command, missing app setting, port mismatch | Check **Log stream** / Kudu; Linux apps must listen on `WEBSITES_PORT` |
| Deploy succeeds but old code serves | Browser/CDN cache, or deployed to the wrong slot | Hard refresh; confirm the target slot |
| Production uses the wrong database after a swap | Connection string not marked **slot setting** | Mark it sticky, redeploy |
| Autoscale never triggers | Below Standard tier, or threshold not met | Upgrade to Standard+; lower threshold/duration |
| Cannot reach private SQL / storage | No VNet integration | Use a dedicated tier (Basic+) with VNet integration (outbound) |
| Key Vault reference shows as empty | Managed identity missing or no Key Vault access policy | Enable managed identity; grant it `get` on the secret |
| Random slow first requests | Instance recycled or scaled in | Enable **Always On**; warm slots before swap |
| Auth redirect loop / 401 | Wrong redirect URL or provider config | Verify redirect URLs and client ID/secret |

## Quick Reference

App Service runs your code on Azure-managed virtual machines behind a load balancer. You rent the compute as an **App Service Plan** (the tier determines available features; size × count determines capacity, billed per instance-hour whether or not traffic arrives) and run one or more **apps** on it. **Standard** enables autoscale and **deployment slots**; **VNet integration** (outbound) works on any dedicated tier (**Basic+**), while inbound **private endpoints** need Premium v2/v3, and full single-tenant isolation needs an **App Service Environment** (Isolated). Deployment goes through **Kudu/SCM** — typically CI/CD to a **staging slot**, which is warmed and **swapped** into production (a routing change, not a redeploy, so no downtime; only **sticky settings** stay with their slot). Slots enable **blue-green** (the swap itself, with instant rollback) and **canary / A/B** (gradual traffic routing). Configuration is injected as **environment variables**, with **Key Vault references** resolved via **managed identity** for secrets. Scale **up** for heavy requests and **out** for volume, always with a maximum and cooldown. Monitoring is through **Application Insights**.
