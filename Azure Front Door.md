# Azure Front Door

## 1. Introduction

>Azure Front Door is a **Layer 7 (HTTP/HTTPS)** global load balancer and Application Delivery Controller (ADC) that uses Microsoft's global edge network to optimize web traffic routing. It provides high-performance, low-latency, secure, and scalable delivery of websites and web applications across multiple regions.

### Key Benefits:
- **Global HTTP(S) Load Balancing**: Distribute traffic across multiple regions
- **Intelligent Routing**: Latency-based, priority-based, and weighted routing
- **High Availability**: Automatic failover and disaster recovery
- **Performance Optimization**: Edge caching, compression, and HTTP/2 support
- **Security**: Integrated WAF, DDoS protection, and SSL termination
- **Multi-Cloud Support**: Connect to Azure, AWS, GCP, and on-premises origins
- **Real-time Monitoring**: Integration with Azure Monitor and analytics

### Architecture Overview:
- **Global Edge Network**: 100+ Points of Presence (PoPs) worldwide
- **Anycast IP Routing**: Clients connect to nearest PoP automatically
- **Intelligent Backend Selection**: Dynamic routing based on health and performance
- **Edge Processing**: WAF, caching, and SSL termination at the edge

---

## 2. Core Components

Azure Front Door consists of several interconnected components:

| Component | Description | Purpose |
|----------|-------------|---------|
| **Frontend Host/Domain** | Public DNS name for client access | Entry point for user requests |
| **Backend Pool** | Set of backend origins (web apps, VMs, CDNs) | Destination for routed traffic |
| **Routing Rule** | Defines traffic routing logic | Controls how requests are processed |
| **Health Probe** | Monitors backend endpoint health | Ensures traffic goes to healthy origins |
| **Web Application Firewall (WAF)** | Security layer for threat protection | Protects against common web attacks |
| **Rules Engine** | Advanced request/response processing | Custom logic for headers, redirects, etc. |
| **Origin Group** | Logical grouping of origins | Enables failover and load balancing |

### Component Relationships:
- **Frontend Host** → **Routing Rule** → **Backend Pool** → **Origins**
- **Health Probes** monitor **Backend Pool** members
- **WAF** filters traffic before **Routing Rules**
- **Rules Engine** processes requests/responses

---

## 3. Configurations

### A. **Profile Configuration**
- **Profile Name**: Unique identifier for Front Door instance
- **Resource Group**: Logical container for Front Door resources
- **Location**: Not applicable (global service)
- **Tier Selection**: Standard or Premium tier

### B. **Network Configuration**
- **Anycast IP**: Automatically assigned global IP addresses
- **IPv6 Support**: Dual-stack IPv4/IPv6 configuration
- **Private Link**: Secure connection to private origins
- **Custom Domains**: CNAME mapping to Front Door endpoints

### C. **SSL/TLS Configuration**
- **Certificate Management**:
  - Azure-managed certificates (automatic)
  - Custom certificates (Key Vault integration)
  - Third-party certificates (manual upload)
- **TLS Versions**: Support for TLS 1.0, 1.1, 1.2, 1.3
- **Cipher Suites**: Configurable encryption algorithms
- **HTTPS Redirect**: Automatic HTTP to HTTPS redirection

### D. **Caching Configuration**
- **Cache Behavior**: Enable/disable caching per route
- **Cache Duration**: TTL settings for different content types
- **Cache Keys**: Custom cache key generation
- **Bypass Cache**: Query string parameters to bypass cache
- **Cache Compression**: Enable compression for cached content

### E. **Security Configuration**
- **DDoS Protection**: Automatic L3/L4 protection
- **IP Filtering**: Allow/deny lists by IP address or range
- **Rate Limiting**: Request throttling per client
- **Geographic Filtering**: Block traffic from specific countries
- **Security Headers**: Custom security headers injection

---

## 4. Web Application Firewall (WAF)

### A. **WAF Overview**
Azure Front Door integrates with Azure Web Application Firewall to provide comprehensive protection against web-based threats and vulnerabilities.

### B. **WAF Features**
- **OWASP Top 10 Protection**: SQL injection, XSS, CSRF, etc.
- **Microsoft Threat Intelligence**: Real-time threat data integration
- **Bot Protection**: Automated bot detection and mitigation
- **API Protection**: Rate limiting and parameter validation
- **Custom Rules**: User-defined security policies

### C. **Rule Types**
- **Managed Rule Sets**:
  - Microsoft Default Rule Set (DRS)
  - OWASP Core Rule Set (CRS)
  - Bot Manager Rule Set
- **Custom Rules**:
  - IP-based rules (allow/block)
  - Geographic rules
  - Rate limiting rules
  - Request header/body rules
  - Session-based rules

### D. **WAF Modes**
- **Prevention Mode**: Blocks malicious requests
- **Detection Mode**: Logs threats without blocking
- **Disabled Mode**: WAF rules not applied

### E. **WAF Configuration**
- **Policy Assignment**: Associate WAF policies with domains
- **Rule Exclusions**: Whitelist legitimate traffic
- **Custom Error Pages**: Branded error responses
- **Logging**: Detailed WAF logs for analysis
- **Tuning**: Adjust rules to reduce false positives

---

## 5. Frontend Hosts/Domains

### A. **Domain Types**
- **Default Domain**: `*.azurefd.net` (automatically provided)
- **Custom Domains**: Your own domain names (e.g., `www.contoso.com`)
- **Apex Domains**: Root domains without subdomain (e.g., `contoso.com`)
- **Wildcard Domains**: Supports wildcard certificates (e.g., `*.contoso.com`)

### B. **Domain Configuration**
- **CNAME Mapping**: Point custom domain to Front Door endpoint
- **Domain Validation**: Verify domain ownership
- **Certificate Provisioning**: Automatic or manual certificate management
- **HTTPS Enforcement**: Redirect HTTP to HTTPS
- **HSTS Headers**: HTTP Strict Transport Security

### C. **Multi-Domain Support**
- **Single Front Door**: Host multiple domains
- **Shared Backend Pools**: Use same origins for different domains
- **Domain-Specific Routing**: Route different domains to different backends
- **SSL/SNI Support**: Multiple SSL certificates per Front Door

### D. **Domain Security**
- **Certificate Validation**: DV, OV, EV certificates supported
- **Key Vault Integration**: Secure certificate storage
- **Auto-Renewal**: Automatic certificate renewal
- **Security Policies**: Per-domain security configurations

---

## 6. Backend Pools

### A. **Supported Backend Types**
- **Azure App Service**: Web apps, API apps, function apps
- **Azure Virtual Machines**: Single VMs or VM Scale Sets
- **Azure Container Instances**: Containerized applications
- **Azure Kubernetes Service**: K8s services and ingress
- **Azure Storage**: Static website hosting
- **Azure API Management**: API gateways
- **External Origins**: Any publicly accessible HTTP/HTTPS endpoint
- **Multi-Cloud**: AWS, GCP, and other cloud providers

### B. **Backend Configuration**
- **Host Header**: Override default host header
- **Backend Host Name**: FQDN or IP address of origin
- **HTTP/HTTPS Port**: Custom port configuration
- **Priority**: Failover priority (1 = highest)
- **Weight**: Load balancing weight (1-1000)
- **Enabled Status**: Enable/disable individual backends

### C. **Backend Pool Settings**
- **Load Balancing**: Weighted round-robin distribution
- **Session Affinity**: Cookie-based sticky sessions
- **Health Probe**: Monitor backend availability
- **Failover Logic**: Automatic failover to healthy backends
- **Drain Mode**: Graceful backend removal

### D. **Advanced Backend Features**
- **Private Link**: Secure connection to private endpoints
- **Origin Override**: Modify requests before forwarding
- **Timeout Settings**: Backend connection and response timeouts
- **Retry Logic**: Automatic retry on failures
- **Circuit Breaker**: Stop sending traffic to failing backends

---

## 7. Routing Rules

### A. **Rule Components**
- **Rule Name**: Unique identifier for the rule
- **Frontend Hosts**: Domains that trigger the rule
- **Route Type**: Forward or Redirect
- **Accepted Protocol**: HTTP, HTTPS, or both
- **Patterns to Match**: URL path patterns
- **Route Details**: Backend pool and forwarding protocol

### B. **Routing Methods**
- **Latency-Based**: Route to lowest latency backend
- **Priority-Based**: Active/passive failover routing
- **Weighted**: Distribute traffic based on weights
- **Session Affinity**: Maintain user session stickiness
- **Path-Based**: Route based on URL path patterns

### C. **URL Path Matching**
- **Exact Match**: Specific path matching
- **Wildcard Match**: Pattern-based matching (e.g., `/api/*`)
- **Regex Support**: Regular expression matching
- **Case Sensitivity**: Configure case-sensitive matching
- **Query String**: Include query parameters in matching

### D. **Advanced Routing Features**
- **URL Rewrite**: Modify request URLs before forwarding
- **Header Manipulation**: Add, remove, or modify headers
- **Redirect Rules**: HTTP to HTTPS, domain redirects
- **Conditional Logic**: Complex routing based on multiple criteria
- **A/B Testing**: Traffic splitting for testing

### E. **Rules Engine**
- **Match Conditions**: Complex conditions for rule execution
- **Actions**: URL rewrite, header modification, redirects
- **Rule Priority**: Order of rule evaluation
- **Variables**: Dynamic values in rules
- **Nested Conditions**: Complex logical expressions

---

## 8. Use Cases

### A. **Global Load Balancing**
- **Multi-Region Deployment**: Route users to nearest region
- **Geographic Distribution**: Serve content from multiple locations
- **Traffic Optimization**: Intelligent routing based on performance
- **Disaster Recovery**: Automatic failover between regions

### B. **Content Delivery and Performance**
- **Static Content Acceleration**: Cache images, CSS, JS at edge
- **Dynamic Content Acceleration**: Optimize API and database calls
- **Mobile Optimization**: Responsive content delivery
- **Progressive Web Apps**: Enhanced PWA performance

### C. **Microservices Architecture**
- **Service Routing**: Route different paths to different services
- **API Gateway**: Centralized API management and routing
- **Blue-Green Deployment**: Seamless application updates
- **Canary Releases**: Gradual rollout of new features

### D. **Security and Compliance**
- **DDoS Protection**: Absorb and mitigate attacks
- **WAF Protection**: Block malicious requests
- **SSL/TLS Termination**: Centralized certificate management
- **Compliance**: Meet regulatory requirements

### E. **Hybrid and Multi-Cloud**
- **Cloud Migration**: Gradual migration to Azure
- **Multi-Cloud Strategy**: Distribute across multiple providers
- **On-Premises Integration**: Connect to existing infrastructure
- **Vendor Lock-in Avoidance**: Flexibility in cloud choices

---

## 9. Cloud CDN Integration

### A. **Azure CDN Integration**
- **Combined Architecture**: Front Door + Azure CDN
- **Layered Caching**: Edge caching at multiple levels
- **Origin Shield**: Reduce origin load with CDN
- **Purge Coordination**: Synchronized cache invalidation

### B. **CDN Features in Front Door**
- **Edge Caching**: Cache static content at PoPs
- **Dynamic Caching**: Cache API responses conditionally
- **Cache Control**: TTL management and cache headers
- **Compression**: GZIP and Brotli compression
- **Image Optimization**: Automatic image format conversion

### C. **Caching Strategies**
- **Cache-First**: Serve from cache when available
- **Cache-Aside**: Populate cache on demand
- **Write-Through**: Update cache on content changes
- **Cache Warming**: Pre-populate cache proactively

### D. **CDN Configuration**
- **Cache Behavior**: Per-route caching settings
- **Cache Keys**: Custom cache key generation
- **Bypass Parameters**: Query strings that bypass cache
- **Cache Duration**: TTL configuration by content type
- **Purge Policies**: Automated cache invalidation

---

## 10. Front Door Tiers and Pricing

### A. **Service Tiers**

| Feature | Standard | Premium |
|---------|----------|---------|
| **WAF** | Basic WAF | Advanced WAF with Bot Protection |
| **Private Link** | Not Available | Available |
| **Advanced Analytics** | Basic | Advanced with real-time metrics |
| **Rules Engine** | 25 rules | 100 rules |
| **Custom Domains** | 100 | 500 |
| **Backend Pools** | 50 | 100 |
| **Caching** | Basic | Advanced with edge-side includes |
| **DDoS Protection** | Basic | Advanced |
| **SLA** | 99.95% | 99.99% |

### B. **Pricing Components**

| Cost Factor | Standard | Premium |
|-------------|----------|---------|
| **Base Fee** | $35/month | $330/month |
| **Data Transfer** | $0.087/GB | $0.087/GB |
| **WAF Requests** | $0.60/1M requests | $0.60/1M requests |
| **Rules Engine** | $0.60/1M requests | $0.60/1M requests |
| **Health Probes** | $0.30/1M probes | $0.30/1M probes |
| **Private Link** | N/A | $0.045/hour per endpoint |

### C. **Cost Optimization**
- **Right-Sizing**: Choose appropriate tier based on needs
- **Caching Strategy**: Reduce origin requests with effective caching
- **Health Probe Optimization**: Minimize probe frequency
- **WAF Tuning**: Optimize rules to reduce false positives
- **Geographic Optimization**: Use regional backends efficiently

### D. **Billing Considerations**
- **Data Transfer**: Ingress free, egress charged
- **Request Volume**: High-volume discounts available
- **Reserved Capacity**: Commit to usage for discounts
- **Monitoring**: Track usage patterns for optimization

---

## 11. Health Probes and Monitoring

### A. **Health Probe Configuration**
- **Probe Path**: HTTP endpoint to check (e.g., `/health`)
- **Probe Interval**: Frequency of health checks (30-255 seconds)
- **Probe Timeout**: Maximum wait time for response
- **Probe Protocol**: HTTP or HTTPS
- **Expected Status Codes**: Success response codes (200, 202, etc.)
- **Probe Method**: GET, HEAD, or POST

### B. **Health Probe Features**
- **Custom Headers**: Add headers to probe requests
- **Body Matching**: Validate response content
- **SSL Verification**: Certificate validation for HTTPS probes
- **Retry Logic**: Multiple attempts before marking unhealthy
- **Probe Regions**: Multiple regions for probe reliability

### C. **Monitoring and Analytics**
- **Azure Monitor**: Metrics, logs, and alerts
- **Real-time Metrics**: Traffic, latency, and error rates
- **Custom Dashboards**: Business-specific monitoring views
- **Log Analytics**: Advanced log analysis and queries
- **Application Insights**: End-to-end application monitoring

### D. **Key Metrics**
- **Request Volume**: Total requests per second
- **Response Time**: Average and percentile latencies
- **Error Rates**: HTTP 4xx and 5xx error percentages
- **Cache Hit Ratio**: Edge cache effectiveness
- **Origin Health**: Backend availability status
- **WAF Metrics**: Blocked requests and attack patterns

---

## 12. Security Features

### A. **DDoS Protection**
- **Automatic Mitigation**: L3/L4 attack absorption
- **Volumetric Protection**: Bandwidth-based attacks
- **Protocol Protection**: TCP SYN, UDP flood protection
- **Resource Protection**: CPU and memory exhaustion prevention
- **Real-time Monitoring**: Attack detection and alerting

### B. **Access Control**
- **IP Filtering**: Allow/deny lists by IP or CIDR
- **Geographic Blocking**: Block traffic by country/region
- **Rate Limiting**: Request throttling per client IP
- **Custom Rules**: Complex access control logic
- **Session Management**: User session tracking and limits

### C. **SSL/TLS Security**
- **Certificate Management**: Automated certificate lifecycle
- **Perfect Forward Secrecy**: ECDHE cipher suites
- **HSTS Support**: HTTP Strict Transport Security
- **TLS 1.3**: Latest TLS protocol support
- **Cipher Suite Control**: Custom encryption algorithms

### D. **Compliance and Governance**
- **Data Residency**: Control where data is processed
- **Audit Logging**: Comprehensive access logs
- **Compliance Reports**: SOC, ISO, PCI DSS compliance
- **Privacy Controls**: GDPR and regional privacy laws
- **Security Assessments**: Regular security evaluations

---

## 13. Troubleshooting Guide

### A. **Common Issues**
- **5xx Errors**: Backend connectivity and health issues
- **DNS Resolution**: Domain configuration problems
- **SSL Certificate**: Certificate validation failures
- **WAF Blocking**: False positive rule triggers
- **Caching Issues**: Stale content or cache misses

### B. **Diagnostic Tools**
- **Azure Monitor**: Metrics and alerting
- **Diagnostic Logs**: Detailed request/response logs
- **Network Watcher**: Connectivity diagnostics
- **Browser Tools**: Client-side debugging
- **Azure CLI/PowerShell**: Automation and scripting

### C. **Performance Troubleshooting**
- **Latency Analysis**: Identify bottlenecks
- **Cache Optimization**: Improve hit rates
- **Origin Optimization**: Backend performance tuning
- **Routing Optimization**: Efficient traffic distribution
- **Compression Settings**: Optimize content delivery

---

## 14. Best Practices

### A. **Architecture Design**
- **Multi-Region Deployment**: Distribute across regions
- **Health Check Strategy**: Comprehensive monitoring
- **Failover Planning**: Disaster recovery procedures
- **Security Layering**: Defense in depth approach
- **Performance Optimization**: Caching and compression

### B. **Operational Excellence**
- **Monitoring Setup**: Comprehensive observability
- **Alerting Strategy**: Proactive issue detection
- **Documentation**: Maintain configuration records
- **Change Management**: Controlled updates and rollbacks
- **Testing**: Regular disaster recovery testing

### C. **Cost Management**
- **Resource Optimization**: Right-size configurations
- **Usage Monitoring**: Track and optimize costs
- **Reserved Capacity**: Commit to predictable usage
- **Automation**: Reduce operational overhead
- **Regular Review**: Periodic cost optimization

---

## Summary

Azure Front Door is a comprehensive global load balancer and application delivery controller that provides:

- **Global Performance**: Intelligent routing via 100+ PoPs worldwide
- **High Availability**: Automatic failover and disaster recovery
- **Advanced Security**: Integrated WAF, DDoS protection, and SSL termination
- **Flexible Routing**: Path-based, latency-based, and weighted routing options
- **Multi-Cloud Support**: Connect to any publicly accessible origin
- **Enterprise Features**: Private Link, advanced analytics, and compliance

The service is ideal for global applications requiring high performance, availability, and security with support for both simple and complex routing scenarios across multiple clouds and regions.
