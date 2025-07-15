# Azure Virtual Machines

## Introduction to Azure Virtual Machines

>Azure Virtual Machines (VMs) are Infrastructure as a Service (IaaS) offerings that provide on-demand, scalable computing resources in the cloud. VMs give you the flexibility of virtualization without having to buy and maintain physical hardware.

### Key Benefits of Azure VMs:
- **Full Control**: Complete administrative access to the operating system
- **Flexibility**: Run Windows, Linux, or custom operating systems
- **Scalability**: Scale up or down based on demand
- **Cost-Effective**: Pay only for what you use
- **Integration**: Seamlessly integrate with other Azure services
- **Global Reach**: Deploy across multiple Azure regions worldwide

### When to Use Azure VMs:
- Migrating existing applications to the cloud (lift-and-shift)
- Running legacy applications that require specific OS configurations
- Development and testing environments
- High-performance computing workloads
- Disaster recovery solutions
- Hosting web applications with custom configurations

---

## Azure VM Creation

Creating VMs in Azure involves several configuration steps to ensure optimal performance and security.

### Planning Phase:
1. **Resource Group**: Logical container for related resources
2. **Region Selection**: Choose based on latency, compliance, and cost
3. **Naming Convention**: Establish consistent naming standards
4. **Subscription**: Select appropriate Azure subscription

### VM Creation Steps:

#### 1. Choose an Image
- **Marketplace Images**: Pre-configured OS images (Windows Server, Ubuntu, CentOS, etc.)
- **Custom Images**: Your own prepared VM images
- **Shared Image Gallery**: Centralized repository for custom images
- **Azure Compute Gallery**: Enterprise-grade image management

#### 2. Select VM Size
- Based on CPU, memory, storage, and network requirements
- Can be changed after deployment (with downtime)
- Consider future scaling needs

#### 3. Configure Networking
- **Virtual Network (VNet)**: Network isolation boundary
- **Subnet**: Logical division within VNet
- **Public IP**: For internet connectivity (optional)
- **Network Security Group (NSG)**: Firewall rules
- **Network Interface Card (NIC)**: VM's network connection

#### 4. Set Up Storage
- **OS Disk**: System disk (typically 127GB default)
- **Data Disks**: Additional storage volumes
- **Disk Types**: Standard HDD, Standard SSD, Premium SSD, Ultra SSD
- **Managed vs Unmanaged**: Azure-managed or self-managed storage

#### 5. Configure Security & Management
- **Authentication**: SSH keys (Linux) or username/password
- **Boot Diagnostics**: Troubleshooting capabilities
- **Monitoring**: Azure Monitor integration
- **Backup**: Azure Backup configuration
- **Extensions**: Additional software installation

### Tools for VM Creation:
- **Azure Portal**: Web-based graphical interface
- **Azure CLI**: Command-line interface for automation
- **PowerShell**: Windows-based scripting
- **ARM Templates**: Infrastructure as Code (JSON)
- **Terraform**: Third-party Infrastructure as Code
- **Bicep**: Domain-specific language for ARM templates

#### Real-World Example:
A financial services company creates a Windows Server VM to host their legacy accounting application. They use Premium SSD storage for performance, configure NSG rules for security, and enable Azure Backup for data protection.

---

## Azure VM Sizes

Azure provides numerous VM sizes organized into series, each optimized for specific workloads and performance requirements.

### VM Size Categories:

#### General Purpose (Balanced CPU-to-Memory)
| Series | Description | vCPUs | Memory (GB) | Use Cases |
|--------|-------------|--------|-------------|-----------|
| **B** | Burstable performance | 1-20 | 0.5-80 | Dev/test, light web servers |
| **D** | Standard general purpose | 2-64 | 8-256 | Web apps, small databases |
| **DC** | Confidential computing | 2-8 | 8-32 | Sensitive data processing |

#### Compute Optimized (High CPU-to-Memory)
| Series | Description | vCPUs | Memory (GB) | Use Cases |
|--------|-------------|--------|-------------|-----------|
| **F** | High-performance CPU | 2-72 | 4-144 | Batch processing, gaming servers |
| **FX** | Latest generation compute | 2-72 | 4-144 | High-performance computing |

#### Memory Optimized (High Memory-to-CPU)
| Series | Description | vCPUs | Memory (GB) | Use Cases |
|--------|-------------|--------|-------------|-----------|
| **E** | Memory-optimized | 2-104 | 16-2048 | In-memory databases, analytics |
| **M** | Largest memory offerings | 32-448 | 875-12000 | SAP HANA, large databases |

#### Storage Optimized (High Disk Throughput)
| Series | Description | vCPUs | Memory (GB) | Use Cases |
|--------|-------------|--------|-------------|-----------|
| **L** | Storage optimized | 4-80 | 32-640 | Big data, NoSQL databases |

#### GPU Enabled
| Series | Description | vCPUs | Memory (GB) | Use Cases |
|--------|-------------|--------|-------------|-----------|
| **N** | GPU-enabled | 6-448 | 56-3584 | AI/ML, rendering, simulation |

#### High Performance Computing
| Series | Description | vCPUs | Memory (GB) | Use Cases |
|--------|-------------|--------|-------------|-----------|
| **H** | High-performance computing | 44-176 | 350-704 | Scientific modeling, simulations |

### Size Selection Considerations:
- **Current Workload**: Analyze CPU, memory, and storage requirements
- **Future Growth**: Plan for scaling needs
- **Cost Optimization**: Balance performance with budget
- **Regional Availability**: Not all sizes available in all regions

---

## Azure Availability and Scale

### Availability Sets

Availability Sets ensure high availability by distributing VMs across multiple fault domains and update domains within a single data center.

#### Key Concepts:
- **Fault Domain**: Group of VMs sharing common power source and network switch (up to 3 per availability set)
- **Update Domain**: Group of VMs that can be rebooted together during maintenance (up to 20 per availability set)

#### Benefits:
- **99.95% SLA**: When using 2 or more VMs in an availability set
- **Protection**: Against hardware failures and planned maintenance
- **No Additional Cost**: Only pay for individual VMs

#### Limitations:
- **Single Data Center**: Only within one Azure data center
- **Manual Management**: VMs must be manually managed
- **Limited Flexibility**: Cannot easily add/remove VMs

### Availability Zones

Availability Zones are physically separate data centers within an Azure region, providing higher availability than availability sets.

#### Features:
- **99.99% SLA**: When using 2 or more VMs across zones
- **Zone Redundancy**: Protection against entire data center failures
- **Low Latency**: High-speed networking between zones

### Virtual Machine Scale Sets (VMSS)

Scale Sets allow you to deploy and manage a set of identical, auto-scaling VMs.

#### Key Features:
- **Auto-scaling**: Automatically add/remove VMs based on demand
- **Load Balancing**: Built-in load distribution
- **Rolling Updates**: Deploy updates without downtime
- **Health Monitoring**: Automatic replacement of unhealthy instances
- **Large Scale**: Support for up to 1,000 VM instances

#### Scaling Triggers:
- **Metric-based**: CPU, memory, network utilization
- **Schedule-based**: Predictable traffic patterns
- **Custom Metrics**: Application-specific metrics

#### Real-World Example:
An e-commerce platform uses Scale Sets to handle Black Friday traffic. The system automatically scales from 3 to 50 VMs during peak hours and scales back down afterward.

---

## Azure Containers and Functions

### Azure Container Instances (ACI)

ACI provides the fastest and simplest way to run containers in Azure without managing virtual machines.

#### Features:
- **Serverless Containers**: No VM management required
- **Per-Second Billing**: Pay only for execution time
- **Fast Startup**: Containers start in seconds
- **Custom Resource Requirements**: Specify exact CPU and memory
- **Integration**: Works with Azure Kubernetes Service (AKS)

#### Use Cases:
- **CI/CD Pipelines**: Build and test automation
- **Batch Processing**: Short-lived computational tasks
- **Event-Driven Applications**: Triggered by external events
- **Microservices**: Lightweight service deployment

#### Container Groups:
- **Multi-container Pods**: Deploy related containers together
- **Shared Resources**: Share storage and networking
- **Scheduling**: Co-locate containers on same host

### Azure Functions

Azure Functions is a serverless compute service that runs event-triggered code without managing infrastructure.

#### Key Features:
- **Event-Driven**: Triggered by various Azure services
- **Automatic Scaling**: Scale based on demand
- **Consumption-Based Pricing**: Pay only for execution time
- **Language Support**: C#, Java, JavaScript, Python, PowerShell

#### Hosting Plans:
- **Consumption Plan**: Pay-per-use, automatic scaling
- **Premium Plan**: Pre-warmed instances, VNet connectivity
- **Dedicated Plan**: Run on App Service plan

#### Common Triggers:
- **HTTP Requests**: RESTful APIs
- **Timer**: Scheduled execution
- **Blob Storage**: File uploads/changes
- **Queue Messages**: Asynchronous processing
- **Event Grid**: Event-driven architectures

#### Real-World Example:
A photo-sharing app uses Azure Functions to automatically resize and optimize images when users upload them to Blob Storage. The function triggers on blob creation and processes the image within seconds.

---

## Virtual Network Peering

VNet Peering connects two or more virtual networks, enabling resources to communicate as if they were on the same network.

### Types of Peering:

#### Regional VNet Peering
- **Same Region**: Connect VNets within the same Azure region
- **High Bandwidth**: Maximum network performance
- **Low Latency**: Minimal network delay

#### Global VNet Peering
- **Cross-Region**: Connect VNets across different Azure regions
- **Global Connectivity**: Worldwide network reach
- **Compliance**: Keep data within specific geographic boundaries

### Key Features:
- **Private Connectivity**: Traffic stays on Microsoft backbone
- **No Gateways Required**: Direct connection between VNets
- **Bi-directional**: Two-way communication by default
- **Cross-Subscription**: Peer VNets in different subscriptions

### Peering Properties:
- **Allow Virtual Network Access**: Enable communication between VNets
- **Allow Forwarded Traffic**: Allow traffic from other networks
- **Allow Gateway Transit**: Share VPN/ExpressRoute gateways
- **Use Remote Gateways**: Use the remote VNet's gateway

### Limitations:
- **No Transitive Routing**: VNet A cannot reach VNet C through VNet B
- **Address Space Overlap**: Cannot peer VNets with overlapping IP ranges
- **Resource Limits**: Maximum of 500 peerings per VNet

#### Real-World Example:
A multinational corporation uses global VNet peering to connect their Azure infrastructure in North America, Europe, and Asia, enabling secure communication between regional offices while maintaining compliance with local data regulations.

---

## VM Settings and Options

### Operating System Configuration

#### Windows VMs:
- **Windows Server Versions**: 2016, 2019, 2022
- **Windows 10/11**: For development scenarios
- **License Options**: Pay-as-you-go or bring your own license
- **Windows Updates**: Automatic or manual configuration

#### Linux VMs:
- **Distributions**: Ubuntu, CentOS, Red Hat, SUSE, Debian
- **SSH Access**: Key-based authentication recommended
- **Package Management**: Distribution-specific package managers
- **Custom Kernels**: Support for specialized kernels

### Storage Configuration

#### Disk Types:
- **Standard HDD**: Cost-effective for infrequent access
- **Standard SSD**: Better performance than HDD
- **Premium SSD**: High-performance for production workloads
- **Ultra SSD**: Highest performance for mission-critical applications

#### Disk Management:
- **Managed Disks**: Azure-managed storage (recommended)
- **Unmanaged Disks**: Self-managed storage accounts
- **Disk Encryption**: Azure Disk Encryption for data protection
- **Backup**: Azure Backup integration

### Networking Configuration

#### Network Interfaces:
- **Primary NIC**: Required for VM operation
- **Secondary NICs**: Additional network connections
- **IP Addressing**: Static or dynamic IP assignment
- **Accelerated Networking**: SR-IOV for high performance

#### Security Groups:
- **Network Security Groups (NSG)**: Subnet or NIC-level firewall
- **Application Security Groups (ASG)**: Group VMs by application role
- **Service Tags**: Predefined IP ranges for Azure services

### Monitoring and Diagnostics

#### Azure Monitor:
- **Metrics**: CPU, memory, disk, network performance
- **Logs**: System and application logs
- **Alerts**: Automated notifications for issues
- **Dashboards**: Visual monitoring displays

#### Boot Diagnostics:
- **Serial Console**: Access to VM console
- **Screenshot**: Visual VM state
- **Logs**: Boot process information

### Extensions and Software

#### VM Extensions:
- **Custom Script Extension**: Run scripts on VM
- **PowerShell DSC**: Desired State Configuration
- **Antimalware**: Microsoft Antimalware protection
- **Backup Agent**: Azure Backup integration
- **Monitoring Agent**: Azure Monitor integration

#### Application Deployment:
- **Azure DevOps**: CI/CD pipeline integration
- **GitHub Actions**: Automated deployments
- **Configuration Management**: Ansible, Chef, Puppet
- **Container Runtime**: Docker, containerd

### Security Configuration

#### Access Control:
- **Just-in-Time Access**: Temporary administrative access
- **Bastion Host**: Secure RDP/SSH access
- **Conditional Access**: Azure AD integration
- **Multi-Factor Authentication**: Enhanced security

#### Compliance:
- **Azure Policy**: Governance and compliance
- **Security Center**: Security posture management
- **Compliance Manager**: Regulatory compliance tracking

---

## Advanced VM Concepts

### VM Generations
- **Generation 1**: Traditional BIOS-based VMs
- **Generation 2**: UEFI-based VMs with enhanced security features

### Specialized VM Types
- **Confidential VMs**: Hardware-based security enclaves
- **Spot VMs**: Unused Azure capacity at reduced cost
- **Reserved Instances**: Prepaid capacity with significant discounts

### Performance Optimization
- **Accelerated Networking**: SR-IOV for network performance
- **Premium Storage**: High-performance storage options
- **Proximity Placement Groups**: Reduce latency between VMs
- **Dedicated Hosts**: Single-tenant physical servers

### Disaster Recovery
- **Azure Site Recovery**: Automated disaster recovery
- **Backup Strategies**: Regular backup scheduling
- **Cross-Region Replication**: Geographic redundancy
- **Recovery Testing**: Validate disaster recovery procedures

---

## Best Practices and Recommendations

### Cost Optimization
- **Right-sizing**: Choose appropriate VM sizes
- **Auto-shutdown**: Schedule development VMs to shut down
- **Reserved Instances**: Commit to long-term usage for discounts
- **Spot Instances**: Use for fault-tolerant workloads

### Security Best Practices
- **Principle of Least Privilege**: Minimal required access
- **Network Segmentation**: Isolate workloads
- **Regular Updates**: Keep OS and applications current
- **Encryption**: Encrypt data at rest and in transit

### Performance Best Practices
- **Monitoring**: Continuously monitor resource utilization
- **Scaling**: Plan for growth and traffic patterns
- **Storage Performance**: Match storage to workload requirements
- **Network Optimization**: Optimize network configuration

### Management Best Practices
- **Automation**: Use Infrastructure as Code
- **Tagging**: Organize resources with consistent tags
- **Documentation**: Maintain comprehensive documentation
- **Change Management**: Implement proper change procedures

---

## Troubleshooting Common Issues

### VM Startup Issues
- **Boot Diagnostics**: Check boot screenshots and logs
- **Serial Console**: Access VM console for troubleshooting
- **Recovery VM**: Attach problem disk to working VM

### Performance Issues
- **Resource Metrics**: Monitor CPU, memory, disk, network
- **Scaling**: Consider vertical or horizontal scaling
- **Storage Performance**: Evaluate disk performance requirements

### Networking Issues
- **NSG Rules**: Verify security group configurations
- **Connectivity**: Test network connectivity between resources
- **DNS Resolution**: Check DNS configuration

### Security Issues
- **Access Reviews**: Regular access permission audits
- **Vulnerability Scanning**: Identify and remediate vulnerabilities
- **Compliance Monitoring**: Ensure ongoing compliance

---

## Glossary of Key Terms

| Term | Definition |
|------|------------|
| **VM** | Virtual Machine - Virtualized computing resource |
| **Image** | Pre-configured OS template for VM creation |
| **SKU** | Stock Keeping Unit - Specific VM size configuration |
| **Availability Set** | Logical grouping for high availability within a data center |
| **Availability Zone** | Physically separate data centers within a region |
| **Scale Set** | Collection of identical, auto-scaling VMs |
| **Container Instance** | Serverless container hosting service |
| **Azure Functions** | Serverless compute for event-driven code |
| **VNet Peering** | Network connection between virtual networks |
| **Managed Disk** | Azure-managed storage for VM disks |
| **NSG** | Network Security Group - Network firewall rules |
| **Extension** | Software component that extends VM functionality |
| **Spot VM** | Low-cost VM using unused Azure capacity |
| **Reserved Instance** | Prepaid VM capacity with cost savings |
| **Bastion** | Secure access service for VMs |

---

*This comprehensive guide covers all essential aspects of Azure Virtual Machines, from basic concepts to advanced configurations. Regular updates and hands-on practice are recommended to stay current with Azure's evolving capabilities.*
