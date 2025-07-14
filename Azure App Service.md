# Azure App Service

## Introduction to Azure App Service

> Azure App Service is a fully managed Platform-as-a-Service (PaaS) offering from Microsoft Azure that enables developers to build, host, and scale web applications, RESTful APIs, and mobile backends without managing underlying infrastructure.

### What is Azure App Service?
- **Fully Managed**: Microsoft handles infrastructure, OS patching, security updates, and maintenance
- **Multi-language Support**: .NET, Java, Node.js, Python, PHP, Ruby, and more
- **Enterprise-grade**: Built-in security, compliance, and scalability features
- **DevOps Integration**: Seamless CI/CD integration with popular development tools

### Key Benefits
- **Focus on Code**: Developers can concentrate on application logic rather than infrastructure
- **Built-in DevOps**: Integrated deployment pipelines and monitoring
- **Global Scale**: Deploy applications worldwide with automatic load balancing
- **Cost-effective**: Pay only for what you use with flexible pricing tiers

---

## Azure App Service Plans

An **App Service Plan** is the foundation that defines the compute resources and features available to your applications.

### What is an App Service Plan?
An App Service Plan defines:
- **Region**: Geographic location where your app runs
- **Instance Size**: VM size (Small, Medium, Large, Extra Large)
- **Number of Instances**: Scale count for your applications
- **Pricing Tier**: Feature set and performance level
- **Operating System**: Windows or Linux

### Multiple Apps per Plan
- One App Service Plan can host multiple applications
- All apps in the same plan share the same compute resources
- Cost optimization by consolidating compatible apps

### Pricing Tiers and Features

#### Free (F1)
- **Purpose**: Development and testing only
- **Features**:
  - 1 GB storage
  - No custom domains
  - No SSL support
  - No scaling options
  - 60 minutes CPU time per day

#### Shared (D1)
- **Purpose**: Low-traffic apps
- **Features**:
  - 1 GB storage
  - Custom domains supported
  - Shared compute resources
  - No scaling options
  - 240 minutes CPU time per day

#### Basic (B1–B3)
- **Purpose**: Small production apps
- **Features**:
  - Dedicated compute resources
  - Custom domains and SSL
  - Up to 3 instances (manual scaling only)
  - 10 GB storage (B1), 10 GB (B2), 10 GB (B3)
  - No deployment slots

#### Standard (S1–S3)
- **Purpose**: Production applications
- **Features**:
  - Auto-scaling (up to 10 instances)
  - 5 deployment slots
  - Daily backups
  - Traffic Manager integration
  - 50 GB storage

#### Premium (P1–P4)
- **Purpose**: High-performance production apps
- **Features**:
  - Enhanced performance and scaling (up to 30 instances)
  - 20 deployment slots
  - VNet integration
  - 250 GB storage
  - Advanced monitoring

#### Isolated (I1–I3)
- **Purpose**: Enterprise-grade applications
- **Features**:
  - Dedicated Azure environment
  - VNet integration with private endpoints
  - Up to 100 instances
  - Enhanced security and compliance
  - 1 TB storage

---

## Deployment Slots

Deployment slots are a powerful feature that allows you to deploy different versions of your app to separate environments within the same App Service.

### What are Deployment Slots?
- **Separate Environments**: Each slot is a live app with its own hostname
- **Production + Staging**: Production slot + additional staging slots
- **Configuration Isolation**: Each slot can have different app settings and connection strings
- **Available in**: Standard, Premium, and Isolated pricing tiers

### Key Benefits
- **Zero-downtime Deployments**: Swap between slots without downtime
- **Testing in Production Environment**: Test with production-like settings
- **Easy Rollback**: Quickly revert to previous version if issues arise
- **Gradual Rollouts**: Use traffic routing for A/B testing

### Slot Configuration Types

#### Slot-specific Settings
- Connection strings
- App settings marked as "slot setting"
- Custom domain bindings
- SSL certificates and bindings
- Authentication settings

#### Swappable Settings
- App settings (unless marked as slot setting)
- General configuration
- Handler mappings
- Monitoring and diagnostic settings

### Common Deployment Patterns

#### Blue-Green Deployment
1. Deploy new version to staging slot
2. Test thoroughly in staging
3. Swap staging with production
4. Monitor production traffic
5. Swap back if issues occur

#### Canary Deployment
1. Deploy new version to staging slot
2. Route small percentage of traffic to staging
3. Monitor metrics and user feedback
4. Gradually increase traffic percentage
5. Complete swap when confident

---

## Scaling in Azure App Service

Azure App Service provides two types of scaling to handle varying loads and performance requirements.

### Scale Up (Vertical Scaling)
**Definition**: Increasing the power of existing instances by upgrading to a higher pricing tier.

#### When to Scale Up
- Need more CPU, memory, or storage
- Require additional features (deployment slots, custom domains)
- Single-instance applications that can't be distributed

#### How to Scale Up
1. Navigate to App Service in Azure Portal
2. Go to "Scale up (App Service plan)"
3. Select desired pricing tier
4. Click "Apply"

#### Considerations
- **Downtime**: Brief interruption during tier change
- **Cost Impact**: Immediate cost change
- **Feature Availability**: Higher tiers unlock additional features

### Scale Out (Horizontal Scaling)
**Definition**: Increasing the number of instances running your application.

#### Manual Scaling
- Manually set instance count
- Suitable for predictable workloads
- Available in Basic tier and above

#### Auto-scaling
- Automatically adjusts instance count based on rules
- Available in Standard tier and above
- More cost-effective for variable workloads

### Auto-scaling Configuration

#### Metric-based Scaling
**CPU Percentage**
- Scale out when CPU > 70%
- Scale in when CPU < 30%

**Memory Usage**
- Scale out when memory > 80%
- Scale in when memory < 40%

**HTTP Queue Length**
- Scale out when queue length > 10
- Scale in when queue length < 5

**Custom Metrics**
- Application-specific metrics
- External metrics via webhooks

#### Schedule-based Scaling
```
Business Hours Pattern:
- Monday-Friday 9 AM - 6 PM: 5 instances
- Evenings and weekends: 2 instances

Peak Season Pattern:
- Black Friday period: 10 instances
- Regular periods: 3 instances
```

#### Auto-scaling Rules Configuration
1. **Metric Source**: Choose metric (CPU, Memory, etc.)
2. **Threshold**: Define trigger value
3. **Action**: Scale out/in and by how many instances
4. **Cool-down Period**: Wait time before next scaling action
5. **Min/Max Instances**: Set boundaries for scaling

---

## Deploying Web Apps to Azure App Service

Azure App Service supports multiple deployment methods to accommodate different development workflows.

### Deployment Methods

#### 1. GitHub Actions (Recommended)
**Setup Process**:
1. Connect GitHub repository to App Service
2. Azure generates workflow file automatically
3. Customize build and deployment steps
4. Commit triggers automatic deployment

**Benefits**:
- Full CI/CD pipeline
- Built-in testing integration
- Deployment history and rollback
- Multi-environment support

**Example Workflow**:
```yaml
name: Deploy to Azure App Service
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'
    - name: Install dependencies
      run: npm install
    - name: Build application
      run: npm run build
    - name: Deploy to Azure
      uses: azure/webapps-deploy@v2
      with:
        app-name: 'your-app-name'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```

#### 2. Azure DevOps
**Features**:
- Advanced pipeline configurations
- Multi-stage deployments
- Environment approvals
- Enterprise integration

#### 3. Local Git
**Setup**:
1. Enable local Git in deployment center
2. Get Git clone URL
3. Add as remote repository
4. Push to deploy

```bash
git remote add azure https://your-app@your-app.scm.azurewebsites.net/your-app.git
git push azure main
```

#### 4. ZIP Deploy
**Quick Deployment**:
```bash
az webapp deployment source config-zip \
  --resource-group myResourceGroup \
  --name myApp \
  --src myapp.zip
```

#### 5. FTP/FTPS
**Manual Deployment**:
- Get FTP credentials from Azure Portal
- Upload files to `/site/wwwroot`
- Suitable for static files or legacy workflows

### Deployment Best Practices

#### Use Deployment Slots
- Deploy to staging slot first
- Test thoroughly before production swap
- Keep production slot stable

#### Implement Health Checks
- Configure application health endpoints
- Enable automatic rollback on failure
- Monitor deployment success rates

#### Database Migrations
- Run migrations during maintenance windows
- Use deployment slots for database changes
- Implement backward-compatible changes

---

## Authentication and Authorization

Azure App Service provides built-in authentication and authorization capabilities without requiring custom code.

### Authentication Overview
- **Identity Providers**: Integration with popular identity services
- **Token Management**: Automatic token handling and refresh
- **User Claims**: Access to user information from tokens
- **No Code Required**: Configure through Azure Portal or ARM templates

### Supported Identity Providers

#### Azure Active Directory (AAD)
- **Enterprise Identity**: Integration with organizational directories
- **Multi-tenant Support**: Support for multiple Azure AD tenants
- **Advanced Features**: Conditional access, MFA, group claims

#### Microsoft Account
- **Consumer Identity**: Personal Microsoft accounts
- **Integration**: Seamless with Microsoft ecosystem

#### Social Providers
- **Facebook**: OAuth integration
- **Google**: Google account authentication
- **Twitter**: Twitter OAuth
- **GitHub**: Developer-focused authentication

### Authentication Configuration

#### Step-by-step Setup
1. **Navigate to App Service** in Azure Portal
2. **Go to Authentication** in left menu
3. **Toggle "Enable Authentication"**
4. **Choose Action**:
   - Allow anonymous requests
   - Require authentication
5. **Add Identity Provider**:
   - Select provider (AAD, Facebook, etc.)
   - Configure client ID and secret
   - Set redirect URLs
6. **Configure Token Store**: Enable token storage if needed

#### Authentication Settings
```json
{
  "enabled": true,
  "httpApiPrefixPath": "/.auth",
  "cookieExpiration": {
    "convention": "FixedTime",
    "timeToExpiration": "08:00:00"
  },
  "preserveUrlFragmentsForLogins": false,
  "allowedExternalRedirectUrls": [
    "https://myapp.com/callback"
  ]
}
```

### Authorization Features

#### Role-Based Access Control (RBAC)
- **App-level Roles**: Define custom roles for your application
- **AAD Groups**: Use Azure AD groups for authorization
- **Claims-based**: Access user claims for fine-grained control

#### Custom Authorization
- **Easy Auth Headers**: Access user information via HTTP headers
- **Token Validation**: Validate tokens in application code
- **API Integration**: Secure APIs with same authentication

### Working with Authentication in Code

#### Accessing User Information
```csharp
// C# Example
var user = HttpContext.User;
var name = user.Identity.Name;
var email = user.FindFirst("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress")?.Value;
```

```javascript
// Node.js Example
app.get('/profile', (req, res) => {
  const user = req.headers['x-ms-client-principal'];
  const userInfo = JSON.parse(Buffer.from(user, 'base64').toString());
  res.json(userInfo);
});
```

---

## Slot Swapping Operations

Slot swapping is a core feature that enables zero-downtime deployments and easy rollbacks.

### Understanding Slot Swapping
- **Instant Switch**: URLs and traffic immediately redirect to swapped content
- **Configuration Handling**: Manages different types of settings appropriately
- **Rollback Capability**: Easy reversion if issues are detected

### Swap Process Details

#### Pre-swap Validation
1. **Warmup Phase**: Target slot is warmed up with production settings
2. **Health Check**: Verify application starts successfully
3. **Validation**: Run custom validation scripts if configured

#### Swap Execution
1. **DNS Update**: Update internal DNS to point to new slot
2. **Configuration Apply**: Apply production configuration to target slot
3. **Traffic Routing**: Route all traffic to newly swapped slot

#### Post-swap Monitoring
1. **Health Monitoring**: Continuous monitoring of swapped application
2. **Metrics Collection**: Collect performance and error metrics
3. **Rollback Trigger**: Automatic rollback if health checks fail

### Swap Types

#### Standard Swap
- **Complete Traffic Switch**: All traffic moves to target slot
- **Immediate Effect**: Changes take effect immediately
- **Use Case**: Full production deployments

#### Swap with Preview
- **Two-phase Process**: Preview changes before completing swap
- **Validation Opportunity**: Test with production configuration
- **Manual Completion**: Complete swap after validation

#### Traffic Routing
- **Gradual Transition**: Route percentage of traffic to new slot
- **A/B Testing**: Compare performance between slots
- **Canary Deployment**: Minimize risk with gradual rollout

### Swap Configuration

#### Performing Swap via Azure Portal
1. Navigate to **App Service** > **Deployment slots**
2. Click **Swap** button
3. Select **Source** and **Target** slots
4. Review configuration changes
5. Click **Swap** to execute

#### Swap via Azure CLI
```bash
# Standard swap
az webapp deployment slot swap \
  --resource-group myResourceGroup \
  --name myApp \
  --slot staging \
  --target-slot production

# Swap with preview
az webapp deployment slot swap \
  --resource-group myResourceGroup \
  --name myApp \
  --slot staging \
  --target-slot production \
  --action preview
```

#### Swap via PowerShell
```powershell
# Standard swap
Switch-AzWebAppSlot -ResourceGroupName "myResourceGroup" `
  -Name "myApp" `
  -SourceSlotName "staging" `
  -DestinationSlotName "production"
```

### Swap Rollback Strategies

#### Immediate Rollback
- **Quick Revert**: Swap back to previous slot immediately
- **Minimal Downtime**: Usually completes in seconds
- **Emergency Response**: For critical issues

#### Gradual Rollback
- **Traffic Reduction**: Gradually reduce traffic to problematic slot
- **Monitoring**: Monitor metrics during rollback process
- **User Impact**: Minimize user disruption

---

## Additional Important Topics

### Application Settings and Configuration

#### Environment Variables
- **Runtime Configuration**: Configure app behavior without code changes
- **Secure Storage**: Sensitive values encrypted at rest
- **Slot-specific**: Can be configured per deployment slot

#### Connection Strings
- **Database Configuration**: Secure database connection management
- **Type-specific**: SQL Server, MySQL, PostgreSQL, Custom
- **Automatic Injection**: Available as environment variables

### Monitoring and Diagnostics

#### Application Insights
- **Performance Monitoring**: Track response times and dependencies
- **Error Tracking**: Automatic exception capture and analysis
- **User Analytics**: Understand user behavior and usage patterns
- **Custom Metrics**: Track business-specific metrics

#### Diagnostic Logging
- **Application Logs**: Custom application logging
- **Web Server Logs**: IIS request logs
- **Detailed Error Messages**: Comprehensive error information
- **Failed Request Tracing**: Debug request processing issues

### Security Best Practices

#### Network Security
- **VNet Integration**: Connect to private networks
- **Private Endpoints**: Secure inbound connectivity
- **IP Restrictions**: Limit access to specific IP ranges
- **Service Tags**: Use Azure service tags for firewall rules

#### Certificate Management
- **SSL/TLS**: Secure communications with HTTPS
- **Custom Certificates**: Upload and manage certificates
- **Managed Certificates**: Free SSL certificates from Azure
- **Certificate Binding**: Bind certificates to custom domains

### Performance Optimization

#### Caching Strategies
- **Output Caching**: Cache rendered pages
- **Data Caching**: Cache frequently accessed data
- **CDN Integration**: Use Azure CDN for static content
- **Redis Cache**: Implement distributed caching

#### Resource Optimization
- **Connection Pooling**: Optimize database connections
- **Static Content**: Serve static files efficiently
- **Compression**: Enable gzip compression
- **Minification**: Minimize CSS and JavaScript files

---

## Common Scenarios and Solutions

### Scenario 1: Zero-Downtime Deployment
1. **Setup**: Create staging deployment slot
2. **Deploy**: Deploy new version to staging slot
3. **Test**: Thoroughly test in staging environment
4. **Swap**: Perform slot swap to production
5. **Monitor**: Watch for issues and rollback if needed

### Scenario 2: A/B Testing
1. **Deploy**: Deploy variant to staging slot
2. **Configure**: Set up traffic routing (e.g., 10% to staging)
3. **Monitor**: Collect metrics from both versions
4. **Analyze**: Compare performance and user engagement
5. **Decide**: Choose winning version and complete rollout

### Scenario 3: Scaling for Traffic Spikes
1. **Monitor**: Set up CPU and memory monitoring
2. **Configure**: Create auto-scaling rules
3. **Test**: Verify scaling behavior during load testing
4. **Optimize**: Adjust thresholds based on application behavior
5. **Alert**: Set up alerts for scaling events

### Scenario 4: Multi-region Deployment
1. **Plan**: Choose regions based on user geography
2. **Deploy**: Create App Services in multiple regions
3. **Configure**: Set up Traffic Manager for routing
4. **Sync**: Ensure consistent deployments across regions
5. **Monitor**: Track performance and availability globally

---

## Troubleshooting Common Issues

### Deployment Failures
- **Check Build Logs**: Review build output for errors
- **Verify Dependencies**: Ensure all dependencies are included
- **Configuration Issues**: Validate app settings and connection strings
- **Resource Limits**: Check if hitting storage or compute limits

### Performance Issues
- **Enable Diagnostics**: Turn on detailed logging
- **Monitor Metrics**: Use Application Insights for performance tracking
- **Database Performance**: Check database connection and query performance
- **Resource Utilization**: Monitor CPU and memory usage

### Authentication Problems
- **Redirect URLs**: Verify correct redirect URLs are configured
- **Token Expiration**: Check token refresh configuration
- **Provider Configuration**: Validate identity provider settings
- **CORS Issues**: Configure CORS for single-page applications

>This comprehensive guide covers all the essential aspects of Azure App Service, from basic concepts to advanced deployment strategies and troubleshooting techniques.
