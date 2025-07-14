# Azure Application Gateway

## 1. Introduction

Azure Application Gateway is a **Layer 7 (HTTP/HTTPS) load balancer** that enables you to manage traffic to your web applications. It provides advanced routing capabilities, SSL termination, Web Application Firewall (WAF), and integration with other Azure services for high availability and security.

### Key Benefits:
- Layer 7 load balancing with advanced routing
- Path-based and multi-site routing capabilities
- SSL offloading and termination
- Integration with Web Application Firewall (WAF)
- Auto-scaling and zone redundancy (v2 SKU)
- Built-in health monitoring and probes
- Supports both private and public endpoints
- Cookie-based session affinity

### Architecture Overview:
- Operates within an Azure Virtual Network
- Sits between clients and backend resources
- Can be deployed across multiple availability zones
- Integrates with Azure Monitor for logging and metrics
- Supports both public and private IP configurations

---

## 2. Core Components

Application Gateway consists of several interconnected components:

| Component | Description | Purpose |
|----------|-------------|---------|
| **Frontend IP Configuration** | Public or private IP address used by clients | Entry point for client requests |
| **Backend Pool** | Set of backend servers (VMs, App Services, IPs) | Destination for routed traffic |
| **HTTP Listener** | Listens for incoming traffic on specified port/protocol | Receives and processes incoming requests |
| **Routing Rule** | Defines how requests are routed from listener to backend | Controls traffic flow logic |
| **Health Probe** | Monitors backend endpoint health | Ensures traffic only goes to healthy backends |
| **SSL Certificate** | Used for terminating HTTPS connections | Enables secure communication |
| **Backend HTTP Settings** | Configuration for backend communication | Defines how gateway talks to backends |
| **Web Application Firewall (WAF)** | Optional security component | Protects against common web vulnerabilities |

### Component Relationships:
- **Listener** → **Routing Rule** → **Backend Pool** → **Backend HTTP Settings**
- **Health Probes** monitor **Backend Pool** members
- **SSL Certificates** attach to **Listeners** for HTTPS termination

---

## 3. Features

### A. **Layer 7 Load Balancing**
- Distributes HTTP(S) traffic based on application-layer information
- Supports multiple algorithms: round-robin, least connections, IP hash
- Content-based routing using URL paths, headers, and query strings

### B. **Path-Based Routing**
- Routes traffic based on URL paths (e.g., `/api/*`, `/images/*`)
- Ideal for microservices architectures
- Supports wildcard patterns and regex matching

### C. **Multi-Site Hosting**
- Host multiple domains using a single Application Gateway
- Route traffic to different backend pools based on hostname
- Supports wildcard certificates for multiple subdomains

### D. **SSL Offloading and Termination**
- Terminate SSL/TLS at the gateway to reduce backend server overhead
- Support for SNI (Server Name Indication) for multiple certificates
- End-to-end SSL encryption support

### E. **HTTP/2 Support**
- Improves performance by reducing connection overhead
- Multiplexes multiple requests over a single connection
- Backward compatible with HTTP/1.1

### F. **Compression**
- GZIP compression for reducing response sizes
- Configurable compression settings
- Improves bandwidth utilization

### G. **Connection Draining**
- Gracefully removes backend instances from service
- Completes existing connections before removal
- Prevents connection drops during maintenance

---

## 4. Backend Pool Configuration

### Supported Backend Types:
- **Virtual Machines** (by IP address or NIC)
- **VM Scale Sets** (automatically managed members)
- **App Services** (via FQDN only)
- **Internal Load Balancers**
- **External IP addresses or FQDNs**
- **Azure Container Instances**

### Backend Pool Best Practices:
- Use health probes to ensure only healthy backends receive traffic
- Configure appropriate timeouts for backend communication
- Implement connection draining for graceful updates
- Use multiple backend pools for different application tiers

### Backend HTTP Settings:
- **Port Configuration**: Define backend communication port (80, 443, custom)
- **Protocol**: HTTP or HTTPS for backend communication
- **Cookie-based Affinity**: Enable/disable session stickiness
- **Request Timeout**: Maximum time to wait for backend response
- **Connection Draining**: Graceful connection termination settings
- **Host Name Override**: Custom host header for backend requests
- **Path Override**: Modify request path before forwarding

---

## 5. Security Features

### A. **Web Application Firewall (WAF)**
- **OWASP Top 10 Protection**: SQL injection, XSS, CSRF, etc.
- **Custom Rules**: IP filtering, rate limiting, request filtering
- **Managed Rule Sets**: Microsoft-managed and community rule sets
- **Modes**: Prevention (blocks threats) or Detection (logs only)
- **Exclusions**: Whitelist legitimate requests that trigger rules

### B. **DDoS Protection**
- Integrated with Azure DDoS Protection Standard
- Network-level protection against volumetric attacks
- Application-level protection through WAF rules

### C. **Access Control**
- **IP Whitelisting/Blacklisting**: Restrict access by source IP
- **Geographic Filtering**: Block traffic from specific countries
- **Rate Limiting**: Prevent brute-force and DoS attacks
- **Custom Security Headers**: Add security headers to responses

### D. **Secure Communication**
- **TLS Policy Configuration**: Minimum TLS version enforcement
- **Cipher Suite Selection**: Control supported encryption algorithms
- **Certificate Management**: Automated certificate renewal with Key Vault
- **Private Link Integration**: Secure backend connectivity

---

## 6. Configuration and Scaling

### A. **SKU Types and Scaling**

| SKU | Scaling Type | Features |
|-----|-------------|----------|
| **Standard_v1** | Manual scaling (2-32 instances) | Basic load balancing, SSL termination |
| **Standard_v2** | Auto-scaling (0-125 instances) | Zone redundancy, improved performance |
| **WAF_v1** | Manual scaling (2-32 instances) | Includes WAF functionality |
| **WAF_v2** | Auto-scaling (0-125 instances) | WAF + zone redundancy |

### B. **Auto-Scaling Configuration (v2 SKU)**
- **Minimum Instance Count**: Always-on instances for consistent performance
- **Maximum Instance Count**: Upper limit to control costs
- **Scaling Metrics**: CPU, memory, connection count, request rate
- **Scaling Policies**: Scale-out and scale-in rules
- **Warm-up Time**: Time for new instances to become fully operational

### C. **Zone Redundancy**
- Deploy across multiple availability zones
- Automatic failover between zones
- Improved availability and disaster recovery

### D. **Performance Optimization**
- **Connection Multiplexing**: Reuse connections to backends
- **Keep-Alive**: Maintain persistent connections
- **Compression**: Reduce response sizes
- **Caching**: Cache static content (preview feature)

---

## 7. Routing Rules

### A. **Rule Types**
- **Basic Rules**: Simple listener-to-backend pool mapping
- **Path-based Rules**: Route based on URL path patterns
- **Multi-site Rules**: Route based on hostname

### B. **Rule Components**
- **Priority**: Order of rule evaluation (lower number = higher priority)
- **Listener**: Source of incoming requests
- **Backend Pool**: Destination for routed traffic
- **Backend HTTP Settings**: Configuration for backend communication
- **Conditions**: URL path, hostname, headers, query strings

### C. **Path-based Routing Examples**
```
/api/* → API Backend Pool
/images/* → Static Content Backend Pool
/admin/* → Admin Backend Pool
/* → Default Backend Pool
```

### D. **Multi-site Routing Examples**
```
www.contoso.com → Contoso Backend Pool
api.contoso.com → API Backend Pool
admin.contoso.com → Admin Backend Pool
```

### E. **Advanced Routing Features**
- **URL Rewrite**: Modify request URLs before forwarding
- **Header Modification**: Add, remove, or modify HTTP headers
- **Query String Routing**: Route based on query parameters
- **Host Header Override**: Change host header for backend requests

---

## 8. Session Affinity

### A. **Cookie-based Affinity**
- **Mechanism**: Uses ApplicationGatewayAffinity cookie
- **Configuration**: Enabled/disabled per backend HTTP settings
- **Cookie Properties**: HttpOnly, Secure, SameSite attributes
- **Timeout**: Configurable session timeout

### B. **Use Cases**
- **Stateful Applications**: Applications that store session state locally
- **Shopping Carts**: E-commerce applications with cart state
- **User Preferences**: Applications with user-specific configurations
- **Legacy Applications**: Applications not designed for stateless operation

### C. **Limitations and Considerations**
- **Scalability Impact**: Can cause uneven load distribution
- **Failover Behavior**: Sessions lost if backend instance fails
- **Alternative Solutions**: External session stores (Redis, database)
- **Best Practices**: Use for legacy apps, prefer stateless design

---

## 9. SSL Profile

### A. **SSL Profile Components**
- **SSL Certificate**: TLS certificate for HTTPS termination
- **SSL Policy**: TLS version and cipher suite configuration
- **Client Certificate**: Mutual TLS authentication settings
- **SSL Protocols**: Supported TLS versions (1.2, 1.3)

### B. **SSL Certificate Management**
- **Certificate Sources**:
  - Azure Key Vault integration
  - Direct certificate upload
  - Managed certificates (App Service domains)
- **Certificate Formats**: PFX, PEM formats supported
- **Multiple Certificates**: SNI support for multiple domains
- **Certificate Renewal**: Automated renewal with Key Vault

### C. **SSL Policies**
- **Predefined Policies**: Microsoft-managed cipher suites
- **Custom Policies**: User-defined cipher suite selection
- **TLS Versions**: Minimum TLS version enforcement
- **Perfect Forward Secrecy**: ECDHE cipher suite support

### D. **End-to-End SSL**
- **SSL Termination**: Decrypt at gateway, HTTP to backend
- **SSL Bridging**: Decrypt and re-encrypt to backend
- **SSL Passthrough**: Direct SSL tunnel to backend
- **Mixed Mode**: Different SSL modes for different listeners

---

## 10. Health Probes and Monitoring

### A. **Health Probe Configuration**
- **Probe Type**: HTTP or HTTPS
- **Probe Path**: Health check endpoint URL
- **Probe Interval**: Frequency of health checks (15-300 seconds)
- **Timeout**: Maximum wait time for response (1-120 seconds)
- **Unhealthy Threshold**: Failed probes before marking unhealthy
- **Expected Status Codes**: HTTP status codes indicating health

### B. **Monitoring Integration**
- **Azure Monitor**: Metrics and logs integration
- **Log Analytics**: Centralized log analysis
- **Application Insights**: Application performance monitoring
- **Alert Rules**: Proactive notifications for issues

### C. **Key Metrics**
- **Throughput**: Requests per second, bytes per second
- **Latency**: Response time, backend response time
- **Error Rates**: 4xx, 5xx error percentages
- **Connection Metrics**: Current connections, failed connections
- **Health Status**: Healthy/unhealthy backend count

---

## 11. Pricing and Cost Optimization

### A. **Cost Components**
- **Instance Hours**: Based on gateway size and uptime
- **Data Processing**: Measured per GB processed
- **WAF Usage**: Additional cost for WAF-enabled SKUs
- **Outbound Data Transfer**: Standard Azure egress charges

### B. **Cost Optimization Strategies**
- **Right-sizing**: Choose appropriate instance size
- **Auto-scaling**: Scale down during low-traffic periods
- **Reserved Instances**: Commit to long-term usage for discounts
- **Monitoring**: Track usage patterns and optimize accordingly

---

## 12. Troubleshooting Guide

### A. **Common Issues**
- **5xx Errors**: Backend health, timeout, capacity issues
- **SSL Handshake Failures**: Certificate, TLS policy problems
- **Routing Issues**: Rule priority, path matching problems
- **WAF Blocking**: False positive rule triggers

### B. **Diagnostic Tools**
- **Activity Logs**: Configuration changes and errors
- **Diagnostic Logs**: Access logs, WAF logs, health probe logs
- **Network Watcher**: Network connectivity diagnostics
- **Azure CLI/PowerShell**: Automation and troubleshooting scripts

### C. **Best Practices**
- **Regular Health Checks**: Monitor backend and gateway health
- **Log Analysis**: Review logs for patterns and issues
- **Performance Testing**: Validate configuration under load
- **Disaster Recovery**: Plan for failover scenarios

---

## 13. Integration with Other Azure Services

### A. **Networking Services**
- **Virtual Network**: Subnet placement and NSG integration
- **Azure DNS**: Custom domain resolution
- **ExpressRoute**: On-premises connectivity
- **VPN Gateway**: Site-to-site connectivity

### B. **Security Services**
- **Key Vault**: Certificate and secret management
- **Azure AD**: Authentication and authorization
- **DDoS Protection**: Network-level protection
- **Security Center**: Security posture management

### C. **Monitoring Services**
- **Azure Monitor**: Metrics and alerting
- **Log Analytics**: Centralized logging
- **Application Insights**: APM integration
- **Network Watcher**: Network diagnostics

---

## 14. Migration and Deployment

### A. **Planning Phase**
- **Traffic Analysis**: Understand current load patterns
- **Dependency Mapping**: Identify backend dependencies
- **SSL Certificate Planning**: Migrate or obtain certificates
- **DNS Strategy**: Plan for DNS cutover

### B. **Deployment Steps**
1. **Infrastructure Setup**: VNet, subnets, NSGs
2. **Gateway Deployment**: Choose SKU and configure basics
3. **Backend Configuration**: Set up backend pools and health probes
4. **Listener Configuration**: Configure ports, protocols, certificates
5. **Routing Rules**: Define traffic routing logic
6. **Security Configuration**: Enable WAF, configure SSL policies
7. **Testing**: Validate configuration with test traffic
8. **DNS Cutover**: Update DNS records to point to gateway
9. **Monitoring Setup**: Configure alerts and dashboards

### C. **Post-Deployment**
- **Performance Monitoring**: Track key metrics
- **Security Monitoring**: Review WAF logs and alerts
- **Optimization**: Fine-tune configuration based on usage
- **Documentation**: Update runbooks and procedures

---

## Summary

Azure Application Gateway is a comprehensive Layer 7 load balancer that provides advanced routing, security, and performance features for modern web applications. Key capabilities include:

- **Advanced Routing**: Path-based and multi-site routing with URL rewrite
- **Security**: Integrated WAF, DDoS protection, and SSL termination
- **Scalability**: Auto-scaling and zone redundancy in v2 SKU
- **Performance**: HTTP/2, compression, and connection optimization
- **Integration**: Deep integration with Azure services and monitoring

>The service is ideal for microservices architectures, multi-tenant applications, and any scenario requiring sophisticated HTTP(S) traffic management with enterprise-grade security and performance features.
