# Production Networking Handbook for DevOps & Cloud Engineers

A comprehensive, production-oriented reference guide connecting networking fundamentals to real-world cloud architectures, Linux internals, containers, and Kubernetes.

---

## Table of Contents
1. [Network Models & Architecture (OSI vs. TCP/IP)](#1-network-models--architecture-osi-vs-tcpip)
2. [Data Encapsulation, Packets & Frames](#2-data-encapsulation-packets--frames)
3. [IP Addressing, Subnetting & CIDR](#3-ip-addressing-subnetting--cidr)
4. [L2 & L3 Foundations (MAC, ARP, Switches & Routers)](#4-l2--l3-foundations-mac-arp-switches--routers)
5. [Network Automation & Addressing: DHCP & NAT](#5-network-automation--addressing-dhcp--nat)
6. [Domain Name System (DNS) & Service Discovery](#6-domain-name-system-dns--service-discovery)
7. [Ports, Sockets & Kernel Networking](#7-ports-sockets--kernel-networking)
8. [Transport Layer: TCP vs. UDP](#8-transport-layer-tcp-vs-udp)
9. [Application Layer Protocols: HTTP & HTTPS](#9-application-layer-protocols-http--https)
10. [Cryptography & Security: TLS 1.3 & PKI](#10-cryptography--security-tls-13--pki)
11. [Network Defense: Firewalls, Security Groups & NACLs](#11-network-defense-firewalls-security-groups--nacls)
12. [Secure Tunnels: VPNs & Overlay Networks](#12-secure-tunnels-vpns--overlay-networks)
13. [Traffic Control: Forward Proxies, Reverse Proxies & Load Balancers](#13-traffic-control-forward-proxies-reverse-proxies--load-balancers)
14. [DevOps Production Troubleshooting Cheat Sheet](#14-devops-production-troubleshooting-cheat-sheet)

---

## 1. Network Models & Architecture (OSI vs. TCP/IP)

### 1.1 The Two Frameworks
* **OSI (Open Systems Interconnection)**: A 7-layer theoretical reference model standardized by ISO. Useful for conceptual fault isolation and vendor interoperability.
* **TCP/IP (Internet Protocol Suite)**: A pragmatic 4-layer (or 5-layer practical) model that describes how the real Internet and modern Linux network stacks actually operate.

| OSI Layer | Name | TCP/IP Layer | Primary Protocol Data Unit (PDU) | Core Protocols / Tech | DevOps Relevance |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **L7** | Application | Application | Data / Payload | HTTP, DNS, gRPC, SSH | Ingress, API Gateways, App metrics |
| **L6** | Presentation | Application | Data | TLS, JSON, Protobuf | SSL termination, serialization |
| **L5** | Session | Application | Data | RPC, SOCKS | Connection reuse, Keep-Alive |
| **L4** | Transport | Transport | **Segment** (TCP) / **Datagram** (UDP) | TCP, UDP, QUIC | L4 Load Balancers (NLB), Port forwarding |
| **L3** | Network | Network / Internet | **Packet** | IPv4, IPv6, ICMP, BGP | VPC routing, CNI overlay (Calico/Cilium) |
| **L2** | Data Link | Network Access / Link | **Frame** | Ethernet, ARP, VLAN, VXLAN | Linux bridges (`cbr0`), MAC tables |
| **L1** | Physical | Network Access / Link | **Bits** | Cables, Fiber, RF | NIC hardware, Link speed/duplex |

### 1.2 DevOps Mental Model: The Stack in Microservices
When a request leaves a container pod in Kubernetes targeted at another pod:
1. **L7**: The application creates an HTTP/JSON request.
2. **L6/L5**: Service mesh (Envoy) terminates/applies mutual TLS (mTLS).
3. **L4**: The Linux kernel wraps the stream in a TCP segment with a source and destination port.
4. **L3**: The kernel encapsulates the segment into an IP packet with Pod source and destination IPs.
5. **L2**: The Container Network Interface (CNI) routes the packet via a `veth` pair into a virtual bridge or VXLAN tunnel interface.
6. **L1**: The host network interface card (NIC) transmits physical electrical or optical signals onto the network fabric.

---

## 2. Data Encapsulation, Packets & Frames

### 2.1 The Encapsulation / Decapsulation Cycle
* **Encapsulation (Outbound / TX)**: Each layer appends a metadata header (and sometimes a trailer) to the data received from the layer above it.
* **Decapsulation (Inbound / RX)**: As bits arrive at the network card, each layer strips off its corresponding header, validates checksums, and dispatches the payload to the appropriate protocol handler.

```text
[ Application Payload ]
[ TCP Header | Application Payload ]                          --> TCP Segment (L4)
[ IP Header  | TCP Header | Application Payload ]             --> IP Packet   (L3)
[ Frame Hdr  | IP Header  | TCP Header | Payload | Frame FCS ] --> Frame       (L2)
```

### 2.2 Maximum Transmission Unit (MTU) & Segment Sizes
* **MTU (Maximum Transmission Unit)**: The maximum size (in bytes) of an L3 packet that can be transmitted on a network layer without fragmentation.
  * Standard Ethernet MTU: $1500\text{ bytes}$.
  * Cloud Jumbo Frames (e.g., AWS VPC inter-instance): $9001\text{ bytes}$.
* **MSS (Maximum Segment Size)**: The maximum amount of L4 TCP data a machine can receive in a single unfragmented piece:
  $$\text{MSS} = \text{MTU} - (\text{IP Header Size} + \text{TCP Header Size})$$
  * Standard IPv4 ($20\text{B}$ IP header $+ 20\text{B}$ TCP header):
    $$\text{MSS} = 1500 - (20 + 20) = 1460\text{ bytes}$$

> **DevOps Pitfall**: When using overlay networks (like VXLAN or Geneve in Kubernetes or Calico), encapsulation adds $50\text{ bytes}$ of outer headers. If the host MTU is $1500$ and Pod MTU is not adjusted to $1450$, packets will fragment or silently drop when the **Don't Fragment (DF)** bit is set, causing sudden connection hangs during large data transfers (e.g., `git push` or large HTTP uploads).

---

## 3. IP Addressing, Subnetting & CIDR

### 3.1 IPv4 vs. IPv6
* **IPv4**: 32-bit address space represented as four decimal octets separated by dots (e.g., `192.168.1.1`). Total addresses: $2^{32} \approx 4.29 \times 10^9$.
* **IPv6**: 128-bit address space written in 8 groups of 4 hexadecimal digits separated by colons (e.g., `2001:0db8:85a3::8a2e:0370:7334`). Total addresses: $2^{128} \approx 3.4 \times 10^{38}$.

### 3.2 Private IP Allocations (RFC 1918)
These ranges are non-routable on the public internet and are used inside private networks and Cloud VPCs:
* `10.0.0.0/8` $\rightarrow 10.0.0.0 - 10.255.255.255$ ($16,777,216$ addresses)
* `172.16.0.0/12` $\rightarrow 172.16.0.0 - 172.31.255.255$ ($1,048,576$ addresses)
* `192.168.0.0/16` $\rightarrow 192.168.0.0 - 192.168.255.255$ ($65,536$ addresses)

### 3.3 CIDR (Classless Inter-Domain Routing) Math
CIDR replaces legacy Class A/B/C networks by explicitly defining how many bits belong to the **Network ID** versus the **Host ID**:
$$\text{Host Bits} = 32 - \text{Prefix Length } (n)$$
$$\text{Total Addresses} = 2^{32 - n}$$
$$\text{Usable Host Addresses (Standard)} = 2^{32 - n} - 2$$
*(Minus 2 accounts for the Network Address [all host bits 0] and the Broadcast Address [all host bits 1]).*

| CIDR Prefix | Subnet Mask | Total Addresses | Usable (Standard) | Usable in AWS VPC* |
| :--- | :--- | :--- | :--- | :--- |
| `/28` | `255.255.255.240` | $16$ | $14$ | $11$ |
| `/24` | `255.255.255.0` | $256$ | $254$ | $251$ |
| `/22` | `255.255.252.0` | $1,024$ | $1,022$ | $1,019$ |
| `/20` | `255.255.240.0` | $4,096$ | $4,094$ | $4,091$ |
| `/16` | `255.255.0.0` | $65,536$ | $65,534$ | $65,531$ |

> **AWS VPC Rule**: In AWS, **5 IP addresses** are reserved per subnet:
> 1. `.0`: Network address.
> 2. `.1`: VPC Router / Default Gateway.
> 3. `.2`: Amazon Provided DNS (Route 53 Resolver).
> 4. `.3`: Reserved for future use.
> 5. `.255`: Network broadcast address.

---

## 4. L2 & L3 Foundations (MAC, ARP, Switches & Routers)

### 4.1 MAC Addresses vs. IP Addresses
* **MAC Address (L2)**: 48-bit burned-in physical identifier (e.g., `52:54:00:12:34:56`). Used strictly for hop-by-hop local link delivery inside the same broadcast domain.
* **IP Address (L3)**: Logical, hierarchical address. Used for end-to-end packet delivery across disparate networks worldwide.

### 4.2 ARP (Address Resolution Protocol)
Before a device can send an Ethernet frame to an IP address on the local subnet, it must discover the target MAC address.
1. **ARP Request**: Host broadcasts an L2 frame to `ff:ff:ff:ff:ff:ff`: *"Who has IP 192.168.1.50? Tell 192.168.1.10."*
2. **ARP Reply**: The target host responds with an L2 unicast frame: *"192.168.1.50 is at 52:54:00:ab:cd:ef."*
3. **ARP Cache**: Both machines store the mapping in their local ARP cache to avoid repeated broadcasts.

```bash
# View and manipulate local ARP / neighbor tables in Linux
ip neigh show
```

### 4.3 Switches vs. Routers
* **Switch (Layer 2)**:
  * Connects devices on the **same** local network.
  * Inspects destination MAC addresses and consults its **MAC Address Table (CAM Table)**.
  * Forwards traffic out only the specific physical switch port where the destination MAC resides.
* **Router (Layer 3)**:
  * Connects **different** networks (e.g., Private Subnet to Public Internet, or VPC to VPC).
  * Strips incoming L2 frames, inspects the destination IP address, checks its **Routing Table**, decrements TTL (Time to Live), re-encapsulates the packet in a new L2 frame, and forwards it to the next hop.

```bash
# View Linux routing table
ip route show
# Example output:
# default via 172.31.16.1 dev eth0 proto dhcp src 172.31.25.100 metric 100
# 172.31.16.0/20 dev eth0 proto kernel scope link src 172.31.25.100
```

---

## 5. Network Automation & Addressing: DHCP & NAT

### 5.1 DHCP (Dynamic Host Configuration Protocol)
Automates client network setup via the 4-step **DORA** sequence over UDP ports 67 (server) and 68 (client):
1. **D**iscover: Client broadcasts looking for a DHCP server.
2. **O**ffer: Server offers an available IP address, lease time, subnet mask, and options.
3. **R**equest: Client requests the offered IP lease.
4. **A**cknowledge: Server commits the lease and delivers DNS server and gateway IPs.

*In Cloud environments (AWS, GCP, Azure), virtual metadata servers emulate DHCP to dynamically configure cloud VMs during boot.*

### 5.2 NAT (Network Address Translation)
Enables multiple internal private devices to access external networks using fewer public IPs, or redirects incoming traffic.

#### 1. SNAT (Source NAT / IP Masquerading)
* **What it does**: Replaces the private source IP and port of an outgoing packet with the public IP and an ephemeral port of the router/NAT gateway.
* **DevOps Use Case**: Cloud NAT Gateways allowing private EC2 instances or private GKE/EKS worker nodes to download updates from the internet without exposing them to inbound attacks.

#### 2. DNAT (Destination NAT / Port Forwarding)
* **What it does**: Replaces the destination IP and port of an inbound packet with an internal private IP and port.
* **DevOps Use Case**: Kubernetes `NodePort` and `ClusterIP` services implemented via `iptables` or `IPVS` in `kube-proxy`.

```text
[ Pod / VM: 10.0.1.5:43210 ] 
         │ (Outgoing Request)
         ▼
[ NAT Gateway / Router ] ── (Replaces Source IP:Port with 203.0.113.10:61001)
         │
         ▼
[ Public Internet Server: 93.184.216.34:443 ]
```

---

## 6. Domain Name System (DNS) & Service Discovery

### 6.1 The Resolution Hierarchy
When resolving a fully qualified domain name (FQDN) like `api.prod.example.com.`:
1. **Browser / OS Cache**: Checks `/etc/hosts` and local DNS cache (e.g., `systemd-resolved`).
2. **Recursive Resolver**: Queries configured resolver (e.g., `8.8.8.8` or AWS VPC DNS `10.x.x.2`).
3. **Root Nameserver (`.`)**: Returns the authoritative servers for the `.com` TLD.
4. **TLD Nameserver (`.com`)**: Returns the authoritative nameservers for `example.com`.
5. **Authoritative Nameserver**: Returns the concrete record (e.g., `A` record containing `93.184.216.34`).

### 6.2 Common DNS Record Types
* **A**: Hostname $\rightarrow$ IPv4 address.
* **AAAA**: Hostname $\rightarrow$ IPv6 address.
* **CNAME**: Canonical Name; alias from one hostname to another.
* **ALIAS / ANAME**: Virtual CNAME at the apex/root domain (`example.com`), resolved by cloud DNS providers (e.g., AWS Route 53).
* **TXT**: Arbitrary text; used for domain verification, SPF, DKIM, and TLS issuance (ACME).
* **SRV**: Defines hostname and port for specific services (frequently used by Consul and internal service discovery).
* **PTR**: Reverse DNS; IP address $\rightarrow$ Hostname.

### 6.3 DevOps Focus: Kubernetes CoreDNS & The `ndots:5` Issue
In Kubernetes, every pod's `/etc/resolv.conf` defaults to:
```text
nameserver 10.96.0.10
search <namespace>.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```
* **How `ndots:5` works**: If a query has fewer than 5 dots (e.g., `api.stripe.com` has 2 dots), the resolver first iterates through every domain in the `search` path before attempting an external lookup:
  1. `api.stripe.com.<namespace>.svc.cluster.local` (NXDOMAIN)
  2. `api.stripe.com.svc.cluster.local` (NXDOMAIN)
  3. `api.stripe.com.cluster.local` (NXDOMAIN)
  4. `api.stripe.com.` (SUCCESS)
* **Optimization**: For external high-throughput services, use a trailing dot (e.g., `api.stripe.com.`) in app configurations to bypass the search path, or tune `ndots` via the pod `dnsConfig`.

---

## 7. Ports, Sockets & Kernel Networking

### 7.1 Port Classifications
Ports provide application-level multiplexing on top of IP addresses (0 to 65535):
* **Well-Known / System Ports ($0 - 1023$)**: Require root/elevated privileges in Linux (e.g., 22: SSH, 80: HTTP, 443: HTTPS, 53: DNS).
* **Registered Ports ($1024 - 49151$)**: Assigned to specific vendor services (e.g., 3306: MySQL, 5432: PostgreSQL, 6379: Redis, 8080: Web proxy).
* **Dynamic / Ephemeral Ports ($49152 - 65535$)**: Allocated automatically by the OS kernel for outbound client connections. (In Linux, check via `cat /proc/sys/net/ipv4/ip_local_port_range`).

### 7.2 What is a Network Socket?
A socket is an OS kernel abstraction representing a network endpoint. A complete TCP connection is uniquely identified by a **5-Tuple**:
$$\text{Connection ID} = \{\text{Protocol}, \text{Source IP}, \text{Source Port}, \text{Destination IP}, \text{Destination Port}\}$$

> **DevOps Pitfall: Ephemeral Port Exhaustion**: If a reverse proxy or load balancer makes tens of thousands of outbound requests per second to a single backend IP without HTTP Keep-Alive, it can exhaust all available ephemeral ports, leading to:
> `Cannot assign requested address (errno 99)`.
> *Remediation*: Enable connection pooling, activate `tcp_tw_reuse`, or add multiple target backend IP addresses.

---

## 8. Transport Layer: TCP vs. UDP

### 8.1 TCP (Transmission Control Protocol)
* **Characteristics**: Connection-oriented, reliable, ordered, byte-stream protocol with built-in flow control and congestion avoidance.

#### The 3-Way Handshake
Establishes sequence numbers and window scaling parameters:
1. **Client $\rightarrow$ Server: `SYN`** (Synchronize sequence number $X$)
2. **Server $\rightarrow$ Client: `SYN-ACK`** (Acknowledge $X+1$, Synchronize sequence number $Y$)
3. **Client $\rightarrow$ Server: `ACK`** (Acknowledge $Y+1$)

#### Connection Teardown (4-Way Handshake)
* Initiator sends `FIN`, Receiver replies with `ACK`.
* Receiver later sends `FIN`, Initiator replies with `ACK`.
* Initiator enters **`TIME_WAIT`** state (typically $60\text{ seconds}$) to ensure late packets in flight do not corrupt subsequent new connections on the same port.

### 8.2 UDP (User Datagram Protocol)
* **Characteristics**: Connectionless, unreliable, unordered, lightweight message-based protocol. No handshakes, no acknowledgments, minimal header overhead ($8\text{ bytes}$ vs. TCP's $20\text{ bytes}$).
* **Primary Use Cases**:
  * Latency-sensitive real-time communication: Voice/Video (VoIP/WebRTC), live telemetry.
  * Simple request-response queries: DNS queries, DHCP, NTP.
  * Modern internet transports: **QUIC / HTTP/3** (runs over UDP, implementing reliability and encryption at user space).
  * Cloud overlay encapsulation: **VXLAN** (runs over UDP port 4789).

---

## 9. Application Layer Protocols: HTTP & HTTPS

### 9.1 Protocol Evolution
* **HTTP/1.1**: Plain text, persistent connections (`Keep-Alive`), but suffers from **Head-of-Line (HoL) blocking** at the HTTP request level (one request per connection at a time).
* **HTTP/2**: Binary framing, single TCP connection per host with **multiplexed streams**, header compression (HPACK), and server push. Susceptible to TCP-level HoL blocking if packets are dropped.
* **HTTP/3**: Operates over **QUIC (UDP)** instead of TCP. Eliminates TCP HoL blocking; provides independent stream delivery, connection migration across network changes, and integrated 0-RTT/1-RTT encryption.

### 9.2 Key HTTP Methods & Status Codes
* **Methods**:
  * `GET`: Idempotent data retrieval.
  * `POST`: Non-idempotent resource creation or processing.
  * `PUT`: Idempotent full resource replacement.
  * `PATCH`: Partial resource update.
  * `DELETE`: Resource removal.
* **DevOps Critical Status Codes**:
  * `499 (Nginx specific)`: Client closed connection before server replied.
  * `502 Bad Gateway`: Reverse proxy received an invalid response (or connection reset) from upstream backend.
  * `503 Service Unavailable`: Backend server pool is full, down, or failing health checks.
  * `504 Gateway Timeout`: Upstream backend did not respond within configured proxy timeout.

---

## 10. Cryptography & Security: TLS 1.3 & PKI

### 10.1 The TLS 1.3 Handshake (1 Round Trip Time - 1-RTT)
TLS secures HTTP connections through asymmetric cryptography for key exchange/authentication and symmetric cryptography for fast payload encryption.

```text
Client                                                  Server
  │                                                       │
  ├─── ClientHello (Supported Ciphers + Key Share) ──────>│
  │                                                       │
  │<── ServerHello (Selected Cipher + Server Key Share) ──┤
  │<── EncryptedExtensions + Certificate + CertVerify ────┤
  │<── Finished ──────────────────────────────────────────┤
  │                                                       │
  ├─── Finished [Encrypted HTTP Data Begins] ─────────────>│
```

### 10.2 Public Key Infrastructure (PKI) in Operations
* **Certificate Authority (CA)**: Trusted third party (e.g., Let's Encrypt, DigiCert) that signs domain certificates.
* **Certificate Anatomy**: Contains the Subject Alternate Names (SANs), Public Key, Expiration Date, and the CA's digital signature.
* **Kubernetes Automation**: Tools like `cert-manager` automate the ACME challenge lifecycle (HTTP-01 or DNS-01) to auto-renew TLS certificates before the standard 90-day expiration window.

---

## 11. Network Defense: Firewalls, Security Groups & NACLs

### 11.1 Stateful vs. Stateless Filtering
* **Stateful Firewalls (e.g., Cloud Security Groups, Linux `iptables/conntrack`)**:
  * Tracks the active state of network connections.
  * If inbound traffic is allowed on port 443, the response outbound traffic is **automatically allowed**, regardless of outbound rule configurations.
* **Stateless Firewalls (e.g., Cloud Network Access Control Lists - NACLs)**:
  * Evaluates every packet independently without connection tracking.
  * If inbound traffic is allowed on port 80/443, you **must explicitly configure outbound rules** allowing traffic back out to ephemeral ports ($1024 - 65535$).

```text
┌─────────────────────────────────────────────────────────────┐
│ Cloud VPC Architecture                                      │
│                                                             │
│  [ Internet Gateway ]                                       │
│          │                                                  │
│          ▼                                                  │
│  ┌───────────────┐                                          │
│  │ Stateless ACL │  (Evaluates all packets at subnet edge)  │
│  └───────┬───────┘                                          │
│          │                                                  │
│          ▼                                                  │
│    ┌───────────┐                                            │
│    │  Subnet   │                                            │
│    │    ┌──────────────────┐                                │
│    │    │ Stateful SecGrp  │ (Tracks connection context)    │
│    │    │   ┌──────────┐   │                                │
│    │    │   │  EC2/VM  │   │                                │
│    │    │   └──────────┘   │                                │
│    │    └──────────────────┘                                │
│    └───────────────────────┘                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 12. Secure Tunnels: VPNs & Overlay Networks

### 12.1 VPN Protocols in Cloud Infrastructure
* **IPSec (Internet Protocol Security)**:
  * Heavyweight, enterprise standard protocol operating at L3.
  * Typical choice for **Site-to-Site VPNs** connecting on-premises data centers to Cloud VPCs (e.g., AWS Direct Connect failover or VPN Gateway).
* **WireGuard**:
  * Modern, lightweight, high-performance VPN running in Linux kernel space over UDP.
  * Uses modern cryptography (ChaCha20, Curve25519).
  * Foundational protocol powering tools like Tailscale and Cilium's transparent node-to-node encryption.

### 12.2 Overlay Networks (Cisco ACI, VXLAN, CNI)
Encapsulates L2/L3 packets inside outer UDP packets to allow virtual networks (e.g., Kubernetes pods) to communicate seamlessly across a physical, routed underlay network without modifying underlying hardware routers.

---

## 13. Traffic Control: Forward Proxies, Reverse Proxies & Load Balancers

### 13.1 Proxy Types Compared
* **Forward Proxy**:
  * Sits in front of **clients**.
  * Mask client identities, enforce enterprise egress compliance, filter outbound malware, cache external resources.
* **Reverse Proxy**:
  * Sits in front of **backend servers**.
  * Performs TLS termination, rate limiting, request compression, URL routing, and security shielding. Examples: Nginx, Traefik, Envoy.

### 13.2 Load Balancers: Layer 4 vs. Layer 7

```text
Layer 4 Load Balancer (TCP/UDP)                Layer 7 Load Balancer (HTTP/gRPC)
Client ──[ TCP SYN ]──> LB ──> Backend         Client ──[ TLS / HTTP Request ]──> LB (Terminates)
(Routes at IP & Port level; ultra-fast)                                            │ (Inspects URI/Headers)
                                                                                  └──[ New HTTP Req ]──> Backend
```

| Dimension | Layer 4 Load Balancer (NLB) | Layer 7 Load Balancer (ALB) |
| :--- | :--- | :--- |
| **Inspection Depth** | IP, Port, TCP/UDP headers | HTTP Headers, Cookies, URL Paths, JSON |
| **Performance / Latency** | Extreme throughput; sub-millisecond | Higher compute overhead; slightly higher latency |
| **TLS Handling** | Pass-through or high-speed termination | Deep inspection, mTLS, header injection |
| **Routing Decisions** | By IP, Port, or TCP state | By URL path (`/api` vs `/static`), host header |
| **Kubernetes Context** | Cloud Provider Service `type: LoadBalancer` | Ingress Controllers (Ingress NGINX, ALB Controller) |

### 13.3 Scheduling Algorithms
1. **Round Robin**: Sequential distribution across active instances.
2. **Least Connections**: Dispatches traffic to the server currently handling the fewest active connections (optimal for long-lived sessions or compute-heavy requests).
3. **IP Hash / Consistent Hashing**: Hashes client IP to ensure the same client consistently routes to the same backend cache/instance (sticky routing without cookies).

---

## 14. DevOps Production Troubleshooting Cheat Sheet

Keep these commands in your incident response runbook for diagnosing networking failures on Linux and container hosts:

### 1. Interface & Routing Checks
```bash
# Check interface status, IP assignment, and MTU
ip -d address show

# Check host routing table and default gateway
ip route show

# Identify which interface and route the kernel selects for an IP
ip route get 1.1.1.1
```

### 2. Sockets, Ports & Process Binding
```bash
# Display listening TCP/UDP ports with process IDs (modern replacement for netstat)
ss -tulpn

# Check socket state summary (detect TIME_WAIT or CLOSE_WAIT pileup)
ss -s
```

### 3. DNS Diagnostics
```bash
# Perform clean DNS query with timing stats
dig api.example.com +stats

# Trace full iterative DNS resolution from root servers
dig api.example.com +trace

# Reverse lookup an IP address
dig -x 10.0.1.50
```

### 4. L4 Connectivity & Firewalls
```bash
# Fast port check without timeout hang (TCP)
nc -zvw3 10.0.1.10 443

# Fast UDP port check
nc -zvuw3 10.0.1.10 53

# Continuous hop-by-hop latency and packet loss analysis (combines ping & traceroute)
mtr --report -c 10 1.1.1.1
```

### 5. Application Layer (HTTP & TLS)
```bash
# View complete HTTP handshake, response headers, and status code
curl -Iv https://example.com

# Test TLS certificate expiry and inspect certificate chain
openssl s_client -connect example.com:443 -servername example.com
```

### 6. Deep Packet Inspection (The Last Resort)
```bash
# Capture incoming HTTP/TLS traffic on interface eth0
sudo tcpdump -nnvv -i eth0 port 443 -c 50

# Capture DNS requests inside a specific Kubernetes node/pod
sudo tcpdump -i any -nn port 53
```