# Azure SQL Server

## 1. Introduction to Azure SQL

>Azure SQL is a family of fully managed SQL database services built on the Microsoft SQL Server engine. It provides Platform-as-a-Service (PaaS) capabilities with enterprise-grade performance, security, and availability in the cloud.

### Azure SQL Family Overview:
- **Azure SQL Database**: Fully managed single databases and elastic pools
- **Azure SQL Managed Instance**: Fully managed instance providing near 100% compatibility with SQL Server
- **SQL Server on Azure VMs**: Infrastructure-as-a-Service (IaaS) offering full control over SQL Server

### Key Benefits:
- **Fully Managed**: No infrastructure management required
- **High Availability**: Built-in redundancy and failover capabilities
- **Scalability**: Dynamic scaling based on workload demands
- **Security**: Enterprise-grade security features and compliance
- **Cost-Effective**: Pay-as-you-go pricing with multiple optimization options
- **Integration**: Native integration with Azure services and tools

---

## 2. Creation of SQL Server (Logical Server)

### 2.1 Azure SQL Server Overview

An Azure SQL Server is a logical container that acts as a central administrative point for managing multiple Azure SQL databases. It provides a unified management interface for security, networking, and server-level configurations.

### 2.2 Key Components of SQL Server Creation

#### Server-Level Settings:
- **Server Name**: Globally unique name for the logical server
- **Server Admin Login**: Administrator username for the server
- **Server Admin Password**: Strong password for administrator account
- **Location**: Azure region where the server will be created
- **Resource Group**: Container for organizing related Azure resources

#### Authentication Methods:
- **SQL Authentication**: Username and password authentication
- **Azure AD Authentication**: Integration with Azure Active Directory
- **Mixed Mode**: Both SQL and Azure AD authentication enabled

### 2.3 Server-Level Features

#### Firewall and Networking:
- **Server-level firewall rules**: Control access to all databases on the server
- **Virtual network rules**: Secure access from specific VNet subnets
- **Private endpoints**: Private connectivity within Azure backbone
- **Public endpoint**: Internet-accessible endpoint with firewall protection

#### Server-Level Security:
- **Azure AD Admin**: Designated Azure AD administrator for the server
- **Transparent Data Encryption**: Server-level encryption settings
- **Advanced Data Security**: Threat detection and vulnerability assessment
- **Auditing**: Server-level audit configuration

---

## 3. SQL Architecture

### 3.1 Azure SQL Database Architecture

#### Logical Architecture:
- **SQL Server (Logical)**: Container for databases and server-level settings
- **Databases**: Individual database instances within the server
- **Resource Groups**: Azure management container for organizing resources
- **Subscriptions**: Billing and access management boundary

#### Physical Architecture:
- **Compute Layer**: Processing resources (vCores, memory)
- **Storage Layer**: Persistent storage for data and logs
- **Network Layer**: Connectivity and security infrastructure
- **Control Plane**: Management and monitoring services

### 3.2 Service Architecture Models

#### Single Database Architecture:
- **Dedicated Resources**: Each database has its own compute and storage
- **Isolation**: Complete resource isolation between databases
- **Independent Scaling**: Scale each database independently
- **Backup and Recovery**: Individual backup and restore capabilities

#### Elastic Pool Architecture:
- **Shared Resources**: Multiple databases share a pool of resources
- **Dynamic Allocation**: Resources automatically distributed based on demand
- **Cost Optimization**: Efficient resource utilization across databases
- **Centralized Management**: Unified management of multiple databases

#### Hyperscale Architecture:
- **Distributed Storage**: Storage tier separated from compute
- **Multiple Replicas**: Read-scale replicas for query distribution
- **Fast Backup/Restore**: Snapshot-based backup and restore
- **Massive Scale**: Support for databases up to 100TB

---

## 4. Service Tiers

### 4.1 DTU-Based Service Tiers

#### Basic Tier:
- **Performance Level**: 5 DTUs
- **Max Database Size**: 2 GB
- **Max Storage**: 2 GB
- **Backup Retention**: 7 days
- **Use Case**: Development and testing environments

#### Standard Tier:
- **Performance Levels**: S0 (10 DTUs) to S12 (3000 DTUs)
- **Max Database Size**: 1 TB
- **Max Storage**: 1 TB
- **Backup Retention**: 35 days
- **Use Case**: General-purpose production workloads

#### Premium Tier:
- **Performance Levels**: P1 (125 DTUs) to P15 (4000 DTUs)
- **Max Database Size**: 4 TB
- **Max Storage**: 4 TB
- **Backup Retention**: 35 days
- **Features**: Higher performance, advanced features
- **Use Case**: Mission-critical applications requiring high performance

### 4.2 vCore-Based Service Tiers

#### General Purpose Tier:
- **Compute**: 1-80 vCores
- **Memory**: 5.1 GB per vCore
- **Storage**: 5 GB to 4 TB
- **Storage Type**: Remote premium storage
- **Backup Retention**: 1-35 days
- **Use Case**: Balanced price-performance for most workloads

#### Business Critical Tier:
- **Compute**: 1-80 vCores
- **Memory**: 5.1 GB per vCore
- **Storage**: 5 GB to 4 TB
- **Storage Type**: Local SSD storage
- **Backup Retention**: 1-35 days
- **Features**: High availability, read replicas, faster I/O
- **Use Case**: Mission-critical applications with high availability requirements

#### Hyperscale Tier:
- **Compute**: 1-80 vCores
- **Memory**: 5.1 GB per vCore
- **Storage**: Up to 100 TB
- **Storage Type**: Distributed storage architecture
- **Backup Retention**: 1-35 days
- **Features**: Rapid scaling, fast backups, read replicas
- **Use Case**: Large databases requiring massive scale

### 4.3 Compute Tiers

#### Provisioned Compute:
- **Always-on**: Resources are pre-allocated and always available
- **Predictable Performance**: Consistent performance levels
- **Billing**: Pay for provisioned resources regardless of usage
- **Use Case**: Production workloads with consistent usage patterns

#### Serverless Compute:
- **Auto-scaling**: Automatically scales based on workload demand
- **Auto-pause**: Automatically pauses during inactive periods
- **Pay-per-use**: Pay only for compute resources consumed
- **Billing**: Per-second billing based on actual usage
- **Use Case**: Development, testing, and intermittent workloads

---

## 5. Database Creation Process

### 5.1 Database Creation Prerequisites

#### Required Information:
- **Database Name**: Unique name within the server
- **Server Selection**: Choose existing server or create new
- **Service Tier**: Select appropriate performance tier
- **Compute Size**: Choose vCore or DTU configuration
- **Storage Options**: Configure storage size and type
- **Backup Configuration**: Set backup retention and redundancy

#### Configuration Options:
- **Collation**: Database collation settings
- **Elastic Pool**: Option to add database to elastic pool
- **Advanced Security**: Enable threat detection and auditing
- **Networking**: Configure firewall rules and VNet access

### 5.2 Database Configuration Settings

#### Performance Configuration:
- **Service Tier**: Basic, Standard, Premium, or vCore-based
- **Compute Size**: Number of DTUs or vCores
- **Storage Size**: Database storage capacity
- **Storage Type**: Standard or premium storage options

#### Backup and Recovery:
- **Backup Retention**: Configure backup retention period
- **Geo-Redundancy**: Enable geo-redundant backups
- **Point-in-Time Restore**: Configure restore capabilities
- **Long-Term Retention**: Set up long-term backup policies

#### Security Configuration:
- **Transparent Data Encryption**: Enable data encryption at rest
- **Dynamic Data Masking**: Configure sensitive data masking
- **Always Encrypted**: Set up client-side encryption
- **Advanced Threat Protection**: Enable threat detection

---

## 6. Pricing Plans and Models

### 6.1 DTU-Based Pricing

#### Database Transaction Unit (DTU):
- **Definition**: Blended measure of CPU, memory, and I/O resources
- **Simplicity**: Single metric for performance comparison
- **Predictability**: Fixed pricing based on DTU allocation
- **Tiers**: Basic (5 DTUs), Standard (10-3000 DTUs), Premium (125-4000 DTUs)

#### DTU Pricing Structure:
- **Basic**: $4.90/month for 5 DTUs
- **Standard**: $14.70/month for S0 (10 DTUs) to $4,508/month for S12 (3000 DTUs)
- **Premium**: $463/month for P1 (125 DTUs) to $14,024/month for P15 (4000 DTUs)

### 6.2 vCore-Based Pricing

#### Virtual Core (vCore) Components:
- **Compute**: Based on number of vCores and generation
- **Storage**: Separate pricing for data storage
- **Backup Storage**: Additional cost for backup storage beyond included amount
- **I/O**: Separate pricing for I/O operations (in some configurations)

#### vCore Pricing Benefits:
- **Flexibility**: Independent scaling of compute and storage
- **Transparency**: Clear understanding of resource allocation
- **Azure Hybrid Benefit**: Use existing SQL Server licenses for cost savings
- **Reserved Capacity**: Significant savings with 1-year or 3-year commitments

### 6.3 Serverless Pricing

#### Serverless Pricing Model:
- **Per-second Billing**: Pay only for compute resources used
- **Auto-pause**: No compute charges during paused periods
- **Storage**: Pay for storage consumed
- **Minimum/Maximum**: Configure minimum and maximum vCore limits

#### Cost Optimization Features:
- **Auto-scaling**: Automatically adjust resources based on demand
- **Pause/Resume**: Automatic pause during inactive periods
- **Development Workloads**: Significant cost savings for intermittent usage

### 6.4 Cost Optimization Strategies

#### Azure Hybrid Benefit:
- **License Mobility**: Use existing SQL Server licenses in Azure
- **Cost Savings**: Up to 55% savings on compute costs
- **Flexibility**: Switch between license-included and hybrid benefit pricing

#### Reserved Capacity:
- **Commitment**: 1-year or 3-year reservation commitments
- **Savings**: Up to 80% savings compared to pay-as-you-go
- **Flexibility**: Exchange or cancel reservations with restrictions

---

## 7. Elastic Pool

### 7.1 Elastic Pool Overview

An Elastic Pool is a cost-effective solution for managing multiple databases with varying and unpredictable usage patterns. It allows databases to share allocated resources within a single pool.

### 7.2 Elastic Pool Architecture

#### Resource Sharing:
- **Shared DTUs/vCores**: Multiple databases share pool resources
- **Dynamic Allocation**: Resources automatically distributed based on demand
- **Performance Isolation**: Individual database performance settings
- **Cost Efficiency**: Optimize costs across multiple databases

#### Pool Configuration:
- **Pool Size**: Total DTUs or vCores allocated to the pool
- **Database Limits**: Minimum and maximum resources per database
- **Storage**: Shared storage pool for all databases
- **Monitoring**: Pool-level and database-level monitoring

### 7.3 Elastic Pool Benefits

#### Cost Optimization:
- **Resource Sharing**: Efficient utilization of allocated resources
- **Peak Smoothing**: Different databases peak at different times
- **Predictable Costs**: Fixed cost for the entire pool
- **Scaling**: Scale the entire pool up or down as needed

#### Management Simplification:
- **Centralized Management**: Manage multiple databases from single interface
- **Consistent Configuration**: Apply settings across all databases in pool
- **Automated Monitoring**: Pool-level performance monitoring and alerting
- **Backup Management**: Centralized backup and restore operations

### 7.4 Elastic Pool Use Cases

#### Multi-Tenant Applications:
- **SaaS Applications**: Separate database per tenant
- **Variable Usage**: Different tenants have different usage patterns
- **Cost Efficiency**: Share resources across tenants
- **Isolation**: Maintain data isolation between tenants

#### Development and Testing:
- **Multiple Environments**: Dev, test, staging databases
- **Intermittent Usage**: Databases used at different times
- **Cost Control**: Avoid over-provisioning individual databases
- **Resource Flexibility**: Easily add or remove databases

### 7.5 Elastic Pool Configuration

#### Pool Sizing:
- **DTU Pools**: 50-4000 DTUs per pool
- **vCore Pools**: 2-80 vCores per pool
- **Database Limits**: Configure min/max resources per database
- **Storage**: Shared storage pool with individual database limits

#### Performance Tuning:
- **Monitor Pool Utilization**: Track overall pool resource usage
- **Database Performance**: Monitor individual database performance
- **Rebalancing**: Move databases between pools as needed
- **Scaling**: Scale pool resources based on aggregate demand

---

## 8. Auditing

### 8.1 Auditing Overview

Azure SQL Database auditing tracks database events and writes them to audit logs for compliance, security analysis, and forensic investigations.

### 8.2 Audit Event Categories

#### Database Events:
- **Data Access**: SELECT, INSERT, UPDATE, DELETE operations
- **Schema Changes**: CREATE, ALTER, DROP operations
- **Security Changes**: GRANT, REVOKE, DENY operations
- **Login Events**: Successful and failed login attempts
- **Connection Events**: Connection establishment and termination

#### Administrative Events:
- **Database Operations**: Database creation, deletion, modification
- **Server Operations**: Server configuration changes
- **Backup Operations**: Backup and restore activities
- **Maintenance Operations**: Index maintenance, statistics updates

### 8.3 Audit Configuration Levels

#### Server-Level Auditing:
- **Scope**: All databases on the server
- **Inheritance**: New databases inherit server audit settings
- **Centralized Management**: Single audit policy for entire server
- **Administrative Events**: Captures server-level operations

#### Database-Level Auditing:
- **Scope**: Specific database only
- **Granular Control**: Fine-grained audit configuration
- **Override**: Can override server-level settings
- **Database-Specific**: Tailored to specific database requirements

### 8.4 Audit Log Destinations

#### Azure Storage Account:
- **Long-term Storage**: Cost-effective for long-term retention
- **Access Control**: Manage access using storage account permissions
- **Retention**: Configure retention policies for automatic cleanup
- **Analysis**: Use Azure tools for log analysis

#### Log Analytics Workspace:
- **Real-time Analysis**: Near real-time audit log analysis
- **Advanced Queries**: Use KQL for complex log analysis
- **Dashboards**: Create custom dashboards and visualizations
- **Alerting**: Set up alerts based on audit events

#### Event Hubs:
- **Streaming**: Real-time audit log streaming
- **Integration**: Integrate with external SIEM systems
- **Processing**: Process audit logs with Azure Stream Analytics
- **Third-party Tools**: Send logs to external monitoring tools

### 8.5 Audit Policy Configuration

#### Audit Actions:
- **Predefined Groups**: Common audit action groups
- **Custom Actions**: Specify individual audit actions
- **Exclusions**: Exclude specific actions from auditing
- **Conditional Auditing**: Audit based on conditions

#### Audit Filters:
- **Principal Filter**: Audit specific users or roles
- **Object Filter**: Audit specific database objects
- **Statement Filter**: Audit specific SQL statements
- **Result Filter**: Audit based on operation results

### 8.6 Compliance and Security Features

#### Threat Detection:
- **Anomaly Detection**: Identify unusual database access patterns
- **Brute Force Attacks**: Detect login brute force attempts
- **SQL Injection**: Identify potential SQL injection attacks
- **Data Exfiltration**: Detect unusual data access patterns

#### Vulnerability Assessment:
- **Security Scans**: Regular security configuration assessments
- **Remediation**: Recommendations for security improvements
- **Compliance**: Check against security best practices
- **Reporting**: Generate security assessment reports

---

## 9. Azure SQL Database Deployment Models

### 9.1 Single Database

#### Overview:
A fully isolated database with dedicated resources, providing predictable performance and complete isolation from other databases.

#### Key Features:
- **Resource Isolation**: Dedicated compute and storage resources
- **Independent Scaling**: Scale resources independently
- **Flexible Tiers**: Support for all service tiers and compute models
- **Full Feature Support**: Access to all Azure SQL Database features

#### Use Cases:
- **Dedicated Applications**: Applications requiring guaranteed resources
- **Microservices**: Each service has its own database
- **Mission-Critical**: Applications requiring maximum performance isolation
- **Variable Workloads**: Databases with unique performance requirements

### 9.2 Elastic Pool

#### Overview:
A cost-effective solution that allows multiple databases to share a set of resources, optimizing costs for applications with variable usage patterns.

#### Key Features:
- **Resource Sharing**: Multiple databases share pool resources
- **Cost Optimization**: Efficient resource utilization
- **Performance Management**: Automated resource balancing
- **Simplified Administration**: Centralized management interface

#### Use Cases:
- **Multi-tenant SaaS**: Applications with multiple tenant databases
- **Development/Testing**: Multiple databases with intermittent usage
- **Variable Workloads**: Databases with unpredictable usage patterns
- **Cost Optimization**: Reduce overall database costs

### 9.3 Hyperscale

#### Overview:
A highly scalable service tier designed for large databases requiring massive scale, fast backup/restore, and high throughput.

#### Key Features:
- **Massive Scale**: Support for databases up to 100TB
- **Fast Backup/Restore**: Snapshot-based backup and restore
- **Read Scale-out**: Multiple read replicas for query distribution
- **Rapid Scaling**: Quick scale-out and scale-up capabilities

#### Use Cases:
- **Large Databases**: Databases requiring massive storage capacity
- **High Throughput**: Applications with high transaction volumes
- **Analytics Workloads**: Large-scale data analysis requirements
- **Rapid Growth**: Applications with unpredictable growth patterns

### 9.4 Serverless

#### Overview:
An on-demand compute tier that automatically scales based on workload demand and pauses during inactive periods.

#### Key Features:
- **Auto-scaling**: Automatic compute scaling based on demand
- **Auto-pause**: Automatic pause during inactive periods
- **Pay-per-use**: Pay only for compute resources consumed
- **Instant Resume**: Quick resume from paused state

#### Use Cases:
- **Development/Testing**: Intermittent database usage
- **Proof of Concepts**: Short-term projects and prototypes
- **Seasonal Applications**: Applications with predictable usage patterns
- **Cost Optimization**: Minimize costs for infrequently used databases

---

## 10. Security Features

### 10.1 Authentication and Authorization

#### Azure Active Directory Integration:
- **Single Sign-On**: Use Azure AD credentials for database access
- **Multi-Factor Authentication**: Enhanced security with MFA
- **Conditional Access**: Apply conditional access policies
- **Managed Identity**: Use managed identities for application authentication

#### SQL Authentication:
- **Username/Password**: Traditional SQL Server authentication
- **Strong Password Policy**: Enforce strong password requirements
- **Account Lockout**: Automatic account lockout after failed attempts
- **Password Expiration**: Configure password expiration policies

### 10.2 Data Protection

#### Transparent Data Encryption (TDE):
- **Data at Rest**: Automatic encryption of database files
- **Real-time Encryption**: Encrypt data before writing to storage
- **Key Management**: Service-managed or customer-managed keys
- **Performance Impact**: Minimal performance overhead

#### Always Encrypted:
- **Client-side Encryption**: Encrypt data on client before sending
- **Key Protection**: Encryption keys never exposed to database
- **Deterministic/Randomized**: Two types of encryption available
- **Application Changes**: Requires application modifications

#### Dynamic Data Masking:
- **Real-time Masking**: Mask sensitive data for non-privileged users
- **Masking Rules**: Define masking patterns for different data types
- **User-based**: Different masking rules for different users
- **No Data Changes**: Original data remains unchanged

### 10.3 Network Security

#### Firewall Rules:
- **Server-level Rules**: Apply to all databases on server
- **Database-level Rules**: Apply to specific database
- **IP Address Ranges**: Allow specific IP addresses or ranges
- **Azure Services**: Control access from Azure services

#### Virtual Network Integration:
- **VNet Rules**: Allow access from specific VNet subnets
- **Service Endpoints**: Secure connectivity from VNet
- **Private Endpoints**: Private IP addresses for database access
- **Network Security Groups**: Additional network-level security

---

## 11. Backup and Recovery

### 11.1 Automated Backups

#### Backup Types:
- **Full Backups**: Complete database backup (weekly)
- **Differential Backups**: Changes since last full backup (12-24 hours)
- **Transaction Log Backups**: Continuous log backups (5-10 minutes)
- **Copy-only Backups**: Manual backups that don't affect backup chain

#### Backup Retention:
- **Short-term**: 1-35 days (configurable)
- **Long-term**: Up to 10 years (configurable)
- **Geo-redundant**: Backup copies in paired region
- **Locally Redundant**: Backup copies in same region

### 11.2 Restore Options

#### Point-in-Time Restore:
- **Granularity**: Restore to any point within retention period
- **Deleted Databases**: Restore deleted databases
- **Cross-region**: Restore to different region
- **New Database**: Restore to new database instance

#### Geo-Restore:
- **Regional Disaster**: Restore from geo-redundant backups
- **Cross-region**: Restore to any Azure region
- **Latest Backup**: Restore from most recent geo-replicated backup
- **Service Tier**: Can change service tier during restore

---

## 12. Monitoring and Performance

### 12.1 Monitoring Tools

#### Azure Monitor:
- **Metrics Collection**: CPU, memory, storage, and database metrics
- **Custom Dashboards**: Create personalized monitoring dashboards
- **Alerting**: Set up alerts based on metric thresholds
- **Log Analytics**: Analyze logs and metrics with KQL queries

#### Query Performance Insight:
- **Top Queries**: Identify resource-consuming queries
- **Query Statistics**: Execution time, frequency, and resource usage
- **Performance Trends**: Historical performance analysis
- **Optimization Recommendations**: Automated tuning suggestions

### 12.2 Performance Optimization

#### Automatic Tuning:
- **Index Management**: Automatic index creation and removal
- **Query Optimization**: Automatic query plan optimization
- **Performance Monitoring**: Continuous performance monitoring
- **Recommendation Engine**: AI-powered optimization recommendations

#### Manual Tuning:
- **Index Analysis**: Identify missing or unused indexes
- **Query Optimization**: Optimize slow-running queries
- **Resource Monitoring**: Monitor resource utilization
- **Performance Testing**: Test performance under different loads

---

## 13. Migration to Azure SQL

### 13.1 Migration Assessment

#### Assessment Tools:
- **Data Migration Assistant**: Assess compatibility and migration readiness
- **Azure Migrate**: Comprehensive migration assessment and planning
- **SQL Server Migration Assistant**: Migrate from other database platforms
- **Database Experimentation Assistant**: Test performance post-migration

#### Compatibility Analysis:
- **Feature Compatibility**: Identify unsupported features
- **Performance Analysis**: Assess performance requirements
- **Security Configuration**: Review security settings
- **Dependency Analysis**: Identify application dependencies

### 13.2 Migration Methods

#### Azure Database Migration Service:
- **Online Migration**: Minimal downtime migrations
- **Offline Migration**: Full migration with planned downtime
- **Hybrid Migration**: Combination of online and offline methods
- **Monitoring**: Real-time migration progress monitoring

#### Manual Migration:
- **BACPAC Export/Import**: Logical migration for smaller databases
- **Backup/Restore**: Physical migration for larger databases
- **Data Sync**: Continuous data synchronization
- **Transactional Replication**: Real-time data replication

---

## 14. Summary and Best Practices

### 14.1 Deployment Model Selection

#### Decision Matrix:
- **Single Database**: Dedicated resources, predictable performance
- **Elastic Pool**: Multiple databases, variable usage, cost optimization
- **Hyperscale**: Large databases, massive scale, high performance
- **Serverless**: Intermittent usage, automatic scaling, cost optimization

### 14.2 Service Tier Recommendations

#### General Guidelines:
- **Basic**: Development and testing only
- **Standard/General Purpose**: Most production workloads
- **Premium/Business Critical**: Mission-critical applications
- **Hyperscale**: Large-scale applications requiring massive capacity

### 14.3 Security Best Practices

#### Security Checklist:
- **Enable Azure AD Authentication**: Use Azure AD for centralized identity management
- **Configure Firewall Rules**: Implement least-privilege network access
- **Enable TDE**: Encrypt data at rest
- **Use Always Encrypted**: Protect sensitive data end-to-end
- **Enable Auditing**: Track database access and changes
- **Regular Security Assessments**: Perform vulnerability assessments

### 14.4 Cost Optimization

#### Cost Management:
- **Right-sizing**: Choose appropriate service tiers and compute sizes
- **Reserved Capacity**: Use reservations for predictable workloads
- **Azure Hybrid Benefit**: Leverage existing SQL Server licenses
- **Elastic Pools**: Optimize costs for multiple databases
- **Serverless**: Use for intermittent workloads

### 14.5 Performance Optimization

#### Performance Best Practices:
- **Monitor Performance**: Use Azure Monitor and Query Performance Insight
- **Enable Automatic Tuning**: Let Azure optimize performance automatically
- **Optimize Queries**: Review and optimize slow-running queries
- **Index Management**: Maintain appropriate indexes
- **Resource Monitoring**: Monitor resource utilization and scale as needed

Azure SQL Database provides a comprehensive, fully managed database solution that scales with your needs while providing enterprise-grade security, performance, and availability. The key to success is choosing the right deployment model, service tier, and configuration options based on your specific requirements and workload characteristics.
