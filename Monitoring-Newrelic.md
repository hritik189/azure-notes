# New Relic

*A SaaS observability platform that ingests telemetry (metrics, events, logs, traces) from your applications, infrastructure, and front-end, stores it in one database, and lets you query, visualize, and alert on all of it together.*

## 1. The Problem It Solves

Traditional monitoring tells you *that* something is wrong (CPU is high, the site is down) but not *why*. The data lives in separate tools — one for server metrics, another for application errors, a third for logs — so when a request is slow, you cannot follow it across the stack. You are left correlating timestamps by hand across dashboards that do not talk to each other.

**Observability** is the shift from "is it up?" to "why is it behaving this way?" It requires all telemetry in one place, linked, so you can start from a symptom (a slow checkout) and trace it down to the exact database query or downstream API call that caused it. New Relic is one such platform.

## 2. What New Relic Is

New Relic is a hosted platform that collects four kinds of telemetry — often abbreviated **MELT** — from everything you run:

- **Metrics** — numeric measurements over time (CPU, request rate, latency).
- **Events** — discrete things that happened (a transaction, a deployment).
- **Logs** — text records emitted by applications and systems.
- **Traces** — the end-to-end path of a single request across services.

Agents or integrations send this data to New Relic, where it lands in one database (NRDB). You then query it with one language (NRQL), build dashboards, and define alerts — all over the same correlated dataset. The key idea: **one store, one query language, every signal linked.**

## 3. How It Works

```
[ your apps / hosts / browser / mobile ]
        │  agents & integrations send MELT
        ▼
   New Relic ingest  ──►  NRDB (one telemetry store)
        │                      │
        │                      ├─► NRQL queries
        │                      ├─► Dashboards
        │                      └─► Alerts → notification channels
```

1. **Instrument** — install an agent (APM language agent, infrastructure agent, browser JS snippet) or an agentless cloud integration.
2. **Ingest** — telemetry streams into New Relic and is stored in NRDB, tagged with metadata so signals from the same entity link together.
3. **Analyze** — query with NRQL, view in dashboards, or let alerts watch the data continuously.

**Billing model — this is the thing to understand before adopting it.** New Relic charges on two axes: **data ingested (per GB)** and **users (by seat type)**. It does *not* charge per host or per agent. The practical consequence: cost is driven by how much telemetry you send (especially logs and high-frequency custom metrics) and how many full-platform users you have — not by infrastructure size. Controlling ingest volume is the main cost lever.

## 4. The Capabilities (what you actually monitor)

All of these feed the same NRDB; they differ only in what they instrument.

- **APM (Application Performance Monitoring)** — a language agent inside your app captures transactions, errors, throughput, and **code-level / method-level** timing, plus **distributed tracing** (following one request across services), **database query** performance, and **external service** calls. Includes **deployment tracking** to correlate a performance change with a release.
- **Infrastructure** — a lightweight host agent reports CPU, memory, disk, and network; integrates with **Kubernetes** and **containers**, and pulls cloud-service metrics (Azure, AWS, GCP) agentlessly.
- **Browser (Real User Monitoring)** — a JS snippet measures real users' experience: **Core Web Vitals** (LCP, INP, CLS), page load, time to first byte, JavaScript errors, AJAX timing, and SPA navigation; supports session replay.
- **Mobile** — native iOS/Android monitoring: crashes, app launch time, network requests, and per-user journeys.
- **Synthetics** — proactive checks that run *before* users hit a problem: ping monitors (uptime), API monitors (endpoint health), and scripted browsers (automated user flows), run from multiple global locations.
- **Logs** — centralized log aggregation with parsing/enrichment, and — because it is the same store — logs are **correlated with traces and metrics**, so you can jump from a slow trace straight to its log lines.

## 5. Alerts

Alerts watch NRDB continuously and notify when conditions are met.

**Condition types:**
- **Static** — fixed threshold (error rate > 5%).
- **Baseline** — compares against a learned historical norm (alert when traffic deviates from the usual pattern).
- **Anomaly** — uses ML to flag unusual behavior without a fixed threshold.
- **NRQL** — an arbitrary NRQL query as the condition; the most flexible form.

**Structure:** a **policy** groups one or more **conditions**; when a condition is breached it opens an **incident**; incidents notify through **channels** (email, Slack, PagerDuty, Teams).

**Best practices:** set thresholds by business impact (not arbitrary numbers), use escalation for critical incidents, and actively fight **alert fatigue** — too many low-value alerts train people to ignore all of them. Tie alerts to runbooks for consistent response.

## 6. NRQL and Dashboards

**NRQL (New Relic Query Language)** is a SQL-like language over NRDB; it powers ad-hoc analysis, custom dashboard widgets, and NRQL alert conditions:

```sql
SELECT percentile(duration, 95) FROM Transaction
WHERE appName = 'checkout-api' FACET name SINCE 1 hour ago
```

**Dashboards** combine NRQL-driven charts into shared views — both pre-built (per capability) and fully custom — for exploration and team collaboration.

## 7. Going Deeper

- **Applied intelligence (AI/ML)** — anomaly detection, alert noise reduction/correlation (grouping related incidents), and assisted root-cause analysis.
- **Custom instrumentation** — record custom events, metrics, and attributes via the agent APIs or direct ingest, so business KPIs sit alongside system telemetry.
- **Integrations** — DevOps (Jenkins, GitHub Actions, GitLab), cloud platforms (Azure/AWS/GCP), collaboration (Slack, Teams, PagerDuty), and ITSM (ServiceNow, Jira).
- **Security / governance** — role-based access control, audit logging, and data encryption in transit and at rest; data retention varies by plan.

**Azure relevance:** New Relic integrates with Azure to pull resource metrics agentlessly, and its Kubernetes integration monitors AKS clusters — so an App Service or AKS workload can report infrastructure, APM, and logs into one place.

## 8. Setup

1. **Create an account** and obtain the **license (ingest) key**.
2. **Install the agent** for each layer — APM agent in the app, infrastructure agent on hosts/K8s, browser snippet in the front-end.
3. **Configure** the agent with the license key (via environment variable or config file) and any custom attributes/app name.
4. **Validate** that data is arriving in NRDB.
5. **Set up users, dashboards, and alert policies.**

## ⚠️ Gotchas

- **Ingest is the cost driver.** Logs and high-frequency custom metrics can balloon the bill. Drop or sample noisy data at the source; do not ship every debug log.
- **High-cardinality attributes are expensive and slow.** Tagging events with unbounded values (user IDs, request IDs) explodes the data and degrades queries. Keep attribute values bounded.
- **Free tier limits matter.** One full user, 100 GB/month, 8-day retention — fine for evaluation, not for a team.
- **Alert fatigue is self-inflicted.** Too many static alerts with arbitrary thresholds get ignored. Alert on symptoms users feel, not on every metric.
- **Agent overhead is small but real.** Verbose custom instrumentation or aggressive sampling in a hot path can add latency. Measure it.
- **Data retention is plan-dependent.** Querying beyond your tier's retention window returns nothing — know your window before relying on historical analysis.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| No data appearing | Wrong/missing license key, agent not started, network egress blocked | Verify key, check agent logs, confirm outbound to New Relic endpoints |
| Unexpectedly high bill | Log or custom-metric ingest spike | Audit ingest by source in the data management view; sample/drop noisy data |
| Slow dashboards / NRQL timeouts | High-cardinality data or wide time ranges | Reduce cardinality; narrow `SINCE`; pre-aggregate |
| Traces not linking across services | Distributed tracing not enabled on all services | Enable DT in every agent; ensure trace headers propagate |
| Alerts not firing | Condition threshold never met, or policy has no channel | Test the NRQL; attach a notification channel |
| Too many noisy alerts | Thresholds too sensitive | Use baseline/anomaly conditions; raise thresholds; enable correlation |

## Quick Reference

New Relic is an observability platform: agents and integrations send **MELT** (metrics, events, logs, traces) into one store (**NRDB**), which you query with **NRQL** and surface through **dashboards** and **alerts**. Its capabilities — **APM** (with distributed tracing and code-level detail), **Infrastructure** (hosts, containers, Kubernetes, cloud), **Browser** (RUM / Core Web Vitals), **Mobile**, **Synthetics**, and **Logs** — all write to the same correlated dataset, so you can trace a symptom from the front-end down to a database query. Alerts use **static, baseline, anomaly, or NRQL** conditions grouped into **policies** that open **incidents** and notify **channels**. Billing is by **data ingested (per GB)** and **users** — not per host — so controlling ingest volume and cardinality is the main cost and performance lever.
