# Azure Application Gateway

*A managed Layer 7 (HTTP/HTTPS) load balancer and reverse proxy. Because it understands HTTP, it can route by URL path and hostname, terminate TLS, rewrite headers, and — with the WAF add-on — block malicious requests.*

## 1. The Problem It Solves

A Layer 4 load balancer (Azure Load Balancer) only sees IP addresses and ports. It can spread connections across servers, but it cannot read the request, so it cannot:

- send `/api/*` to one set of servers and `/images/*` to another,
- host several domains (`shop.contoso.com`, `api.contoso.com`) behind one public IP,
- terminate HTTPS so backends don't each need certificates, or
- inspect a request and reject SQL injection or cross-site scripting.

All of those require *reading the HTTP request* — Layer 7. Application Gateway is the HTTP-aware entry point that does this, and its optional **Web Application Firewall** adds request inspection for security.

**Where it sits among Azure's traffic services:** Application Gateway is **regional** (one region, inside a VNet). For **global** HTTP routing and CDN across regions you use **Front Door**; for **Layer 4** (non-HTTP, or raw TCP/UDP) you use Azure Load Balancer. A common pattern is Front Door (global) → Application Gateway (regional, in-VNet) → backends.

## 2. What It Is

Application Gateway is a managed reverse proxy deployed **inside your virtual network**, in its **own dedicated subnet**. Clients connect to its frontend IP; it terminates the connection, decides where the request should go based on HTTP information, and forwards it to a healthy backend. It is fully managed (Azure runs and patches it) and, in the v2 SKU, autoscaling and zone-redundant.

## 3. How It Works — Request Flow and Components

A request passes through a chain of components. Understanding the chain is understanding the gateway:

```
Client
  │
  ▼
Frontend IP  →  Listener  →  Routing Rule  →  Backend HTTP Settings  →  Backend Pool
(public/      (port +       (which backend   (how to talk to the       (the servers:
 private)      protocol +    for this host/    backend: port, proto,     VMs, VMSS,
               certificate)  path)             timeout, affinity)        App Service, IPs)
                                                      ▲
                                          Health Probes mark pool
                                          members healthy/unhealthy
```

| Component | Role |
|---|---|
| **Frontend IP** | Entry point — public and/or private IP |
| **Listener** | Listens on a port/protocol; holds the TLS certificate for HTTPS |
| **Routing rule** | Maps a listener to a backend, choosing by hostname and/or URL path |
| **Backend HTTP settings** | How the gateway talks to the backend: port, HTTP/HTTPS, timeout, session affinity, host/path override |
| **Backend pool** | The destination servers |
| **Health probe** | Continuously checks backend health so traffic only goes to healthy members |
| **WAF** (optional) | Inspects requests for attacks before they reach the backend |

Among healthy backend members, the gateway distributes requests **round-robin** (optionally pinned by session affinity). It does not expose selectable algorithms like least-connections.

**Backend pool types:** VMs (by IP/NIC), VM Scale Sets, **App Service (by FQDN only — requires host-name override)**, internal load balancers, and external IPs/FQDNs.

## 4. Routing

- **Path-based routing** — route by URL path; ideal for microservices.
  ```
  /api/*    → API backend pool
  /images/* → static-content backend pool
  /*        → default backend pool
  ```
- **Multi-site hosting** — route by hostname, hosting many domains on one gateway.
  ```
  www.contoso.com   → web pool
  api.contoso.com   → API pool
  ```
- **Rule anatomy** — a rule has a **priority** (lower = evaluated first), a **listener**, a **backend pool**, **backend HTTP settings**, and **conditions** (path, hostname, headers).
- **Rewrites** — modify request/response **HTTP headers**, **rewrite URLs**, and **override the host header** or path before forwarding.

## 5. TLS / SSL

The gateway always **terminates** TLS — there is no passthrough mode (that is a Layer 4 feature). What you choose is what happens *after* termination:

- **SSL termination (offload)** — decrypt at the gateway, send **plain HTTP** to the backend. Removes TLS cost from backends; traffic to the backend is unencrypted (fine inside a trusted VNet).
- **End-to-end SSL (SSL bridging)** — decrypt at the gateway (so it can route/inspect), then **re-encrypt** to the backend. Use when the backend must receive HTTPS for compliance.

Supporting features: **SNI** to serve multiple certificates on one listener, **SSL policies** to enforce a minimum TLS version and cipher suites (with Perfect Forward Secrecy via ECDHE), **mutual TLS** (client certificates), and **Key Vault integration** for certificate storage and automated renewal.

## 6. Web Application Firewall (WAF) and Security

The **WAF SKU** adds request inspection in front of routing:

- **Managed rule sets** — the OWASP Core Rule Set protects against the OWASP Top 10 (SQL injection, XSS, CSRF, etc.); Microsoft maintains and updates them.
- **Modes** — **Detection** logs matches only; **Prevention** actively blocks them. Start in Detection, tune, then switch to Prevention.
- **Custom rules** — IP allow/deny, **geo-filtering** by country, **rate limiting** against brute-force/DoS, and header-based rules.
- **Exclusions** — whitelist legitimate requests that trip a managed rule (false positives), which are common and must be tuned.
- **DDoS / Key Vault / Private Link** — integrates with Azure DDoS Protection, stores certs in Key Vault, and supports private backend connectivity.

## 7. SKUs and Scaling

| SKU | Scaling | Notes |
|---|---|---|
| **Standard_v2** | Autoscale (0–125 instances), zone-redundant | Current standard SKU |
| **WAF_v2** | Autoscale + zone-redundant | v2 plus the Web Application Firewall |
| ~~Standard_v1 / WAF_v1~~ | Manual (2–32), no zones | Legacy; v1 is being retired — use v2 |

**Autoscale (v2):** set a **minimum** instance count (warm capacity for sudden spikes) and a **maximum** (cost ceiling); the gateway scales on load. **Zone redundancy** spreads instances across availability zones for automatic failover. Performance features include connection reuse/keep-alive to backends, HTTP/2, and gzip compression.

## 8. Session Affinity

When enabled in the backend HTTP settings, the gateway issues an `ApplicationGatewayAffinity` cookie so a client's requests keep returning to the **same backend** — needed by stateful apps that store session state locally (shopping carts, legacy apps). Trade-offs: it can unbalance load, and if that backend fails the session is lost. Prefer stateless apps with an external session store (Redis); use affinity for apps that cannot be made stateless.

## 9. Health Probes and Monitoring

**Health probes** decide which backends receive traffic. Configurable: protocol (HTTP/HTTPS), path, interval (15–300s), timeout, unhealthy threshold, and the HTTP status codes considered healthy. A backend failing the threshold is pulled from rotation until it recovers.

**Monitoring** flows to **Azure Monitor / Log Analytics**: diagnostic logs (access logs, **WAF logs**, performance logs) and metrics. Key metrics: throughput, latency (total and backend), 4xx/5xx rates, connection counts, and healthy/unhealthy backend count.

## 10. Going Deeper

- **Integration** — lives in a VNet subnet with NSGs; integrates with Azure DNS, Key Vault (certs), Azure AD, DDoS Protection, and Azure Monitor/App Insights. Backends can be reached over ExpressRoute/VPN for hybrid setups.
- **Pricing** — billed on **fixed instance/capacity hours** plus **capacity units** (data processed, connections, compute); WAF SKUs cost more. Optimize with right-sized autoscale min/max and by scaling down off-peak.
- **Deployment outline** — set up VNet/dedicated subnet/NSG → deploy the gateway (choose v2/WAF_v2) → configure backend pools + health probes → listeners (ports, certs) → routing rules → WAF/TLS policy → test → cut over DNS → wire up monitoring/alerts.

## ⚠️ Gotchas

- **It needs its own dedicated subnet.** Application Gateway must be alone in its subnet — you cannot share it with other resources. Size the subnet for autoscale headroom (it consumes IPs as it scales).
- **No SSL passthrough.** The gateway always terminates TLS. If you need an encrypted tunnel straight to the backend untouched, that is Layer 4 (Azure Load Balancer), not Application Gateway.
- **App Service backends need host-header override.** App Service routes by host header, so set "pick host name from backend address" / override the host in the backend HTTP settings, or you get 404s.
- **WAF Prevention causes false positives.** Managed rules will block legitimate traffic (file uploads, rich payloads). Run in **Detection** first, add **exclusions**, then enable Prevention.
- **v1 is legacy.** It lacks autoscale and zone redundancy and is being retired — deploy **v2**.
- **Round-robin only.** There is no least-connections option; uneven backend load usually means session affinity is on or backends differ in capacity.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| **502 Bad Gateway** | All backends unhealthy, probe misconfigured, or backend timeout | Check health-probe path/status codes; verify backend is up; raise request timeout |
| 404 from an App Service backend | Host header not overridden | Enable host-name override in backend HTTP settings |
| TLS handshake fails | Cert/chain issue or TLS policy too strict | Validate the certificate chain; check the SSL policy's min TLS version |
| Legitimate requests blocked | WAF Prevention false positive | Review WAF logs; add an exclusion or tune the rule |
| Traffic hits the wrong backend | Rule priority or path-pattern mismatch | Check rule priorities and path patterns (order matters) |
| Uneven backend load | Session affinity pinning clients | Disable affinity or move to an external session store |

## Quick Reference

Application Gateway is a **regional, Layer 7** load balancer and reverse proxy that lives in a **dedicated VNet subnet**. A request flows **frontend IP → listener (holds the TLS cert) → routing rule (picks the backend by host/path) → backend HTTP settings → backend pool**, with **health probes** gating which members receive traffic, distributed **round-robin**. It does **path-based** and **multi-site** routing, header/URL **rewrites**, and always **terminates TLS** (offload to HTTP, or re-encrypt for end-to-end — never passthrough). The **WAF SKU** adds OWASP managed rules with **Detection/Prevention** modes and custom geo/rate/IP rules. Use the **v2 SKU** for autoscale and zone redundancy. It is regional — pair with **Front Door** for global routing and **Azure Load Balancer** for Layer 4.
