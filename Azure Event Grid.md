# Azure Event Grid

## 1. Introduction
>Azure Event Grid is a fully managed event routing service that enables developers to build event-based architectures using a publish-subscribe model. It provides reliable event delivery at massive scale, supporting millions of events per second with low latency. Event Grid acts as an intelligent event broker that simplifies the development of reactive applications by decoupling event producers from event consumers.

**Key Benefits:**
- Serverless and fully managed
- Built-in resilience and high availability
- Global scale with regional presence
- Pay-per-use pricing model
- Rich ecosystem integration
- Simplified event-driven application development
- Automatic retry and dead-lettering capabilities

**What Makes Event Grid Unique:**
- No infrastructure to manage
- Automatic scaling based on demand
- Built-in integration with Azure services
- Support for both Azure and non-Azure event sources
- CloudEvents standard compliance

## 2. Core Features

### 2.1 Event Routing and Delivery
- **Intelligent Event Routing**: Automatically routes events from sources to appropriate handlers
- **Event Filtering**: Advanced filtering capabilities to deliver only relevant events
- **Reliable Delivery**: Built-in retry mechanisms with exponential backoff
- **Dead Letter Handling**: Automatic handling of undeliverable events
- **At-least-once Delivery**: Guarantees event delivery with possible duplicates

### 2.2 Scalability and Performance
- **Massive Scale**: Handles millions of events per second
- **Low Latency**: Sub-second event delivery
- **Auto-scaling**: Automatic capacity adjustment based on load
- **Global Distribution**: Multi-region deployment capabilities
- **High Availability**: 99.99% SLA with built-in redundancy

### 2.3 Integration Capabilities
- **Azure Services Integration**: Native support for 25+ Azure services
- **Custom Applications**: Support for custom event sources and handlers
- **Third-party Services**: Integration with external SaaS providers
- **Webhook Support**: HTTP endpoint integration
- **Event Hubs Integration**: Stream processing capabilities

### 2.4 Security Features
- **Authentication**: Multiple authentication methods (AAD, SAS, Access Keys)
- **Authorization**: Role-based access control (RBAC)
- **Network Security**: Private endpoints, service endpoints, IP filtering
- **Encryption**: TLS encryption for all communications
- **Managed Identity**: Seamless integration with Azure services

### 2.5 Monitoring and Observability
- **Azure Monitor Integration**: Comprehensive metrics and logging
- **Real-time Monitoring**: Live event delivery tracking
- **Alerting**: Proactive monitoring with custom alerts
- **Diagnostic Logs**: Detailed event delivery information
- **Performance Metrics**: Latency, throughput, and error rate tracking

## 3. Event Messaging (HTTP)

### 3.1 HTTP-based Event Delivery
>Event Grid primarily uses HTTP/HTTPS for event delivery, making it universally compatible with any system that can handle HTTP requests.

**HTTP Delivery Characteristics:**
- **Protocol**: HTTP/HTTPS only
- **Method**: POST requests to webhook endpoints
- **Headers**: Content-Type: application/json
- **Body**: JSON-formatted event data
- **Authentication**: Optional custom headers for authentication

### 3.2 HTTP Event Structure
```json
{
  "id": "unique-event-id",
  "eventType": "Microsoft.Storage.BlobCreated",
  "subject": "/blobServices/default/containers/container/blobs/file.txt",
  "eventTime": "2023-08-17T10:30:00Z",
  "data": {
    "api": "PutBlob",
    "clientRequestId": "request-id",
    "contentType": "text/plain",
    "url": "https://storage.blob.core.windows.net/container/file.txt"
  },
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "topic": "/subscriptions/sub-id/resourceGroups/rg/providers/Microsoft.Storage/storageAccounts/storage"
}
```

### 3.3 HTTP Response Handling
- **Success Response**: HTTP 200-299 status codes
- **Retry Triggers**: HTTP 408, 429, 5xx status codes
- **Permanent Failures**: HTTP 4xx status codes (except 408, 429)
- **Timeout Handling**: 30-second timeout for webhook responses
- **Connection Management**: Automatic connection pooling and reuse

### 3.4 Webhook Validation
`Event Grid implements a validation handshake for webhook endpoints:`

```json
{
  "id": "validation-event-id",
  "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
  "subject": "",
  "eventTime": "2023-08-17T10:30:00Z",
  "data": {
    "validationCode": "12345-validation-code",
    "validationUrl": "https://validation.eventgrid.azure.net/..."
  }
}
```

**Validation Response Required:**
```json
{
  "validationResponse": "12345-validation-code"
}
```

## 4. Push Delivery vs. Pull Delivery

### 4.1 Push Delivery (Default Model)
>Push delivery is Event Grid's primary delivery mechanism where events are actively pushed to subscriber endpoints.

**Push Delivery Characteristics:**
- **Immediate Delivery**: Events delivered as soon as they're published
- **No Polling**: Subscribers don't need to poll for events
- **HTTP Webhooks**: Primary delivery mechanism
- **Automatic Retry**: Built-in retry logic with exponential backoff
- **Scalable**: Handles high-volume event scenarios efficiently

**Push Delivery Process:**
1. Event publisher sends event to Event Grid topic
2. Event Grid evaluates subscription filters
3. Event Grid immediately pushes event to subscriber endpoints
4. Subscriber acknowledges receipt with HTTP 200 response
5. Failed deliveries trigger retry mechanism

**Push Delivery Advantages:**
- Real-time event processing
- No infrastructure required for polling
- Automatic load balancing across multiple endpoints
- Built-in retry and dead-lettering
- Lower operational overhead

**Push Delivery Limitations:**
- Requires publicly accessible endpoints
- Handler must be available when events are delivered
- No control over delivery timing
- Potential for overwhelming slow handlers

### 4.2 Pull Delivery
>Pull delivery allows subscribers to actively retrieve events from Event Grid, though this is less common and primarily available through specific integrations.

**Pull Delivery Characteristics:**
- **On-demand Retrieval**: Subscribers control when to retrieve events
- **Batch Processing**: Can retrieve multiple events at once
- **Flexible Timing**: Process events when convenient
- **Connection Control**: Subscriber manages connection lifecycle

**Pull Delivery Scenarios:**
- **Event Hubs Integration**: Long-term event storage with pull-based consumption
- **Batch Processing**: When real-time processing isn't required
- **Controlled Load**: When subscriber needs to manage processing rate
- **Offline Processing**: When handlers may be temporarily unavailable

**Pull Delivery Implementation:**
- Primarily through Event Hubs capture
- Azure Service Bus integration for queuing
- Custom implementations using storage-based approaches

### 4.3 Hybrid Approaches
**Event Hubs + Event Grid:**
- Event Grid pushes events to Event Hubs
- Consumers pull events from Event Hubs
- Combines real-time routing with stream processing

**Service Bus + Event Grid:**
- Event Grid pushes events to Service Bus queues/topics
- Consumers pull messages from Service Bus
- Adds message queuing capabilities

## 5. Event Subscription

### 5.1 Subscription Fundamentals
>Event subscriptions define how events are routed from topics to event handlers. They act as the configuration layer that determines which events are delivered to which destinations.

**Subscription Components:**
- **Name**: Unique identifier for the subscription
- **Topic**: Source of events (system or custom topic)
- **Endpoint**: Destination for event delivery
- **Filter**: Rules for event selection
- **Dead Letter Destination**: Storage for failed deliveries

### 5.2 Subscription Types
**System Topic Subscriptions:**
- Subscribe to events from Azure services
- Pre-defined event schemas
- Automatic topic creation

**Custom Topic Subscriptions:**
- Subscribe to application-generated events
- User-defined event schemas
- Manual topic creation required

**Domain Subscriptions:**
- Subscribe to events across multiple topics in a domain
- Multi-tenant scenarios
- Bulk subscription management

### 5.3 Subscription Configuration
```json
{
  "destination": {
    "endpointType": "WebHook",
    "properties": {
      "endpointUrl": "https://myapp.azurewebsites.net/api/webhook"
    }
  },
  "filter": {
    "subjectBeginsWith": "/blobServices/default/containers/images/",
    "subjectEndsWith": ".jpg",
    "includedEventTypes": [
      "Microsoft.Storage.BlobCreated",
      "Microsoft.Storage.BlobDeleted"
    ],
    "advancedFilters": [
      {
        "operatorType": "numberGreaterThan",
        "key": "data.contentLength",
        "value": 1000
      }
    ]
  },
  "deadLetterDestination": {
    "endpointType": "StorageBlob",
    "properties": {
      "resourceId": "/subscriptions/sub-id/resourceGroups/rg/providers/Microsoft.Storage/storageAccounts/storage",
      "blobContainerName": "deadletter"
    }
  }
}
```

### 5.4 Subscription Endpoints
**Webhook Endpoints:**
- HTTP/HTTPS endpoints
- Most common subscription type
- Supports authentication headers
- Requires validation handshake

**Azure Function Endpoints:**
- Direct integration with Azure Functions
- Automatic scaling and management
- Built-in authentication

**Logic App Endpoints:**
- Workflow-based event processing
- Visual workflow designer
- Rich connector ecosystem

**Service Bus Endpoints:**
- Queue or topic destinations
- Message queuing capabilities
- FIFO processing options

**Event Hubs Endpoints:**
- Stream processing scenarios
- High-throughput event ingestion
- Apache Kafka compatibility

**Storage Queue Endpoints:**
- Simple message queuing
- Cost-effective for low-volume scenarios
- Manual message processing

**Relay Hybrid Connections:**
- On-premises endpoint connectivity
- Secure connection through Azure Relay
- No inbound firewall rules required

### 5.5 Subscription Lifecycle Management
**Creation Process:**
1. Define subscription properties
2. Configure endpoint and filters
3. Validate endpoint (for webhooks)
4. Activate subscription
5. Monitor delivery metrics

**Modification Operations:**
- Update filters and endpoints
- Enable/disable subscriptions
- Modify retry policies
- Change dead letter destinations

**Deletion Considerations:**
- Stop event delivery immediately
- Clean up dead letter storage
- Remove monitoring configurations
- Update dependent systems

## 6. Key Concepts

### Core Components
- **Event**: A lightweight notification indicating that something happened in a system. Contains metadata about what occurred.
- **Event Source (Publisher)**: The service or application that generates events (e.g., Azure Storage, custom applications).
- **Event Handler (Subscriber)**: The destination endpoint that processes events (e.g., Azure Functions, Logic Apps, webhooks).
- **Topic**: A logical container that receives events from sources. Can be system topics or custom topics.
- **Event Subscription**: Configuration that defines which events to deliver to which handlers, including filtering rules.
- **Event Domain**: A management tool for organizing large numbers of related topics.

### Event Types
- **System Events**: Pre-defined events from Azure services
- **Custom Events**: User-defined events from applications
- **CloudEvents**: Industry-standard event format support

## 7. Architecture

### Publish-Subscribe Pattern
`Event Grid implements a robust pub-sub architecture where:`
- Publishers send events to topics without knowledge of subscribers
- Subscribers register interest in specific event types through subscriptions
- Event Grid handles routing, filtering, and delivery

### System Architecture Components
- **Event Grid Service**: Core routing engine
- **Topics**: Event ingestion endpoints
- **Subscriptions**: Event delivery configurations
- **Event Handlers**: Processing endpoints
- **Dead Letter Storage**: Failed event storage

### Regional Architecture
- Available in multiple Azure regions
- Cross-region event routing supported
- Automatic failover capabilities

## 8. Event Schema

### CloudEvents Schema (v1.0)
```json
{
  "specversion": "1.0",
  "type": "com.example.string",
  "source": "https://example.com/source",
  "id": "1234",
  "time": "2023-08-17T10:30:00Z",
  "datacontenttype": "application/json",
  "subject": "example-subject",
  "data": {
    "key": "value"
  }
}
```

### Event Grid Schema
```json
{
  "id": "unique-event-id",
  "eventType": "Microsoft.Storage.BlobCreated",
  "subject": "/blobServices/default/containers/container/blobs/file.txt",
  "eventTime": "2023-08-17T10:30:00Z",
  "data": {
    "api": "PutBlob",
    "clientRequestId": "request-id",
    "contentType": "text/plain",
    "url": "https://storage.blob.core.windows.net/container/file.txt"
  },
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "topic": "/subscriptions/sub-id/resourceGroups/rg/providers/Microsoft.Storage/storageAccounts/storage"
}
```

## 9. Event Delivery and Retry Policies

### Delivery Guarantees
- **At-least-once delivery**: Events may be delivered multiple times
- **Ordered delivery**: Not guaranteed by default
- **Idempotent handlers**: Recommended for duplicate handling

### Retry Mechanism
- **Exponential backoff**: Increases delay between retries
- **Maximum retry attempts**: Configurable (default: 30)
- **Retry duration**: Up to 24 hours
- **Retry intervals**: 30 seconds to 18 hours maximum

### Delivery Patterns
- **Push delivery**: Event Grid pushes events to handlers
- **Pull delivery**: Available for Event Hubs integration
- **Batch delivery**: Multiple events in single request

## 10. Security

### Authentication Methods
- **Azure Active Directory (AAD)**: Recommended for Azure services
- **Shared Access Signature (SAS)**: Token-based authentication
- **Access Keys**: Topic-level authentication
- **Managed Identity**: For Azure resources

### Authorization
- **Role-Based Access Control (RBAC)**: Fine-grained permissions
- **Built-in roles**: Event Grid Contributor, Event Grid Data Sender
- **Custom roles**: Tailored permissions

### Network Security
- **Private endpoints**: VNet integration
- **Service endpoints**: Secure connectivity
- **IP filtering**: Restrict access by IP ranges
- **TLS encryption**: All communications encrypted

### Webhook Security
- **Validation handshake**: Webhook endpoint verification
- **Event authentication**: Optional authentication headers
- **HTTPS requirement**: Secure webhook endpoints

## 11. Event Filtering

### Filter Types
- **Event Type Filter**: Filter by eventType field
- **Subject Filter**: Filter by subject field using wildcards
- **Advanced Filter**: Complex filtering on any event property
- **Custom Filter**: Application-specific filtering logic

### Filter Operators
- **String operators**: beginsWith, endsWith, contains, stringIn
- **Numeric operators**: numberGreaterThan, numberLessThan, numberIn
- **Boolean operators**: boolEquals
- **Array operators**: arrayContains

### Filter Examples
```json
{
  "filter": {
    "subjectBeginsWith": "/blobServices/default/containers/images/",
    "subjectEndsWith": ".jpg",
    "advancedFilters": [
      {
        "operatorType": "numberGreaterThan",
        "key": "data.contentLength",
        "value": 1000
      }
    ]
  }
}
```

## 12. Dead-lettering

### Configuration
- **Dead letter destination**: Azure Storage Blob or Service Bus Queue
- **Dead letter events**: Events that cannot be delivered
- **TTL (Time-to-Live)**: Maximum retention period

### Common Dead Letter Scenarios
- Handler endpoint unavailable
- Authentication failures
- Malformed event data
- Handler timeout
- HTTP error responses (4xx, 5xx)

### Dead Letter Event Properties
- Original event data
- Delivery attempt count
- Last delivery error
- Timestamp of last attempt

## 13. Monitoring and Diagnostics

### Azure Monitor Integration
- **Metrics**: Delivery success rate, latency, failed deliveries
- **Logs**: Detailed event delivery information
- **Alerts**: Proactive monitoring and notifications
- **Dashboards**: Visual monitoring interfaces

### Key Metrics
- **Published Events**: Total events received
- **Matched Events**: Events matching subscriptions
- **Delivered Events**: Successfully delivered events
- **Failed Events**: Delivery failures
- **Delivery Latency**: End-to-end processing time

### Diagnostic Logs
- **Event delivery logs**: Success/failure details
- **Subscription logs**: Subscription configuration changes
- **Resource logs**: Topic and domain activities

## 14. Use Cases

### Application Integration
- **Microservices communication**: Loosely coupled service interaction
- **Legacy system integration**: Modernizing existing applications
- **Cross-platform messaging**: Connecting diverse systems

### Real-time Processing
- **IoT event processing**: Device telemetry and commands
- **Real-time analytics**: Stream processing and aggregation
- **Live dashboards**: Dynamic data visualization

### Automation and Orchestration
- **Serverless workflows**: Event-driven automation
- **CI/CD pipelines**: Build and deployment triggers
- **Infrastructure automation**: Resource provisioning and management

### Business Process Integration
- **Order processing**: E-commerce workflow automation
- **Customer journey**: Personalized user experiences
- **Compliance monitoring**: Regulatory requirement tracking

## 15. Comparisons with Other Services

### Event Grid vs Logic Apps
| Feature    | Event Grid     | Logic Apps             |
| ---------- | -------------- | ---------------------- |
| Purpose    | Event routing  | Workflow orchestration |
| Triggers   | Event-based    | Multiple trigger types |
| Processing | Stateless      | Stateful workflows     |
| Complexity | Simple routing | Complex business logic |

### Event Grid vs Azure Event Hubs
| Feature   | Event Grid      | Event Hubs                |
| --------- | --------------- | ------------------------- |
| Purpose   | Discrete events | High-throughput streaming |
| Delivery  | Push-based      | Pull-based                |
| Ordering  | Not guaranteed  | Partition-based ordering  |
| Retention | 24 hours        | Up to 7 days              |
| Pricing   | Per operation   | Per throughput unit       |

### Event Grid vs Azure Service Bus
| Feature        | Event Grid | Service Bus           |
| -------------- | ---------- | --------------------- |
| Pattern        | Pub-sub    | Message queuing       |
| Delivery       | Push       | Pull                  |
| Ordering       | No         | FIFO queues available |
| Transactions   | No         | Yes                   |
| Dead lettering | Yes        | Yes                   |


## 16. Best Practices

### Design Principles
- **Idempotent handlers**: Handle duplicate events gracefully
- **Fail-fast approach**: Quick validation and error handling
- **Asynchronous processing**: Non-blocking event handling
- **Event versioning**: Plan for schema evolution

### Performance Optimization
- **Efficient filtering**: Reduce unnecessary event delivery
- **Batch processing**: Handle multiple events together
- **Connection pooling**: Reuse HTTP connections
- **Handler scaling**: Auto-scale based on event volume

### Security Best Practices
- **Use AAD authentication**: Prefer managed identity
- **Secure endpoints**: HTTPS for all webhooks
- **Network isolation**: Private endpoints where possible
- **Audit logging**: Monitor access and changes

### Operational Excellence
- **Monitor delivery metrics**: Track success rates
- **Implement alerting**: Proactive issue detection
- **Regular testing**: Validate event flows
- **Documentation**: Maintain event schemas and flows

## 17. Pricing

### Cost Components
- **Operations**: $0.60 per million operations
- **Advanced filtering**: Additional cost per filter
- **Delivery attempts**: Included in operation cost
- **Storage**: Dead letter storage costs

### Cost Optimization
- **Efficient filtering**: Reduce unnecessary operations
- **Batch delivery**: Multiple events per operation
- **Regional deployment**: Minimize cross-region costs
- **Monitoring**: Track usage patterns

### Pricing Tiers
- **Basic**: Standard operations
- **Premium**: Advanced features (private endpoints, higher SLA)

## 18. Topics and Domains

### System Topics
- **Azure service events**: Pre-configured for Azure resources
- **Automatic creation**: Created when first subscription added
- **Service-specific schemas**: Predefined event types

### Custom Topics
- **Application events**: User-defined event types
- **Manual creation**: Explicit topic creation required
- **Flexible schemas**: Custom event structure

### Event Domains
- **Multi-tenant scenarios**: Organize topics by tenant
- **Bulk operations**: Manage multiple topics together
- **Access control**: Domain-level permissions

## 19. Advanced Features

### Private Endpoints
- **VNet integration**: Secure network connectivity
- **DNS integration**: Automatic DNS configuration
- **Hybrid scenarios**: On-premises connectivity

### Event Hubs Integration
- **Capture events**: Long-term storage and analytics
- **Stream processing**: Real-time event processing
- **Apache Kafka**: Protocol compatibility

### Partner Events
- **Third-party integration**: SaaS provider events
- **Auth0, Stripe, etc.**: Popular service integrations
- **Custom partners**: Build partner integrations

## 20. Troubleshooting

### Common Issues
- **Delivery failures**: Handler endpoint problems
- **Authentication errors**: Invalid credentials
- **Filtering problems**: Incorrect filter configuration
- **Performance issues**: Handler timeout or scaling

### Debugging Tools
- **Activity logs**: Event delivery traces
- **Metrics explorer**: Performance monitoring
- **Log Analytics**: Query and analyze logs
- **Event Grid Explorer**: Visual event inspection

### Resolution Strategies
- **Retry configuration**: Adjust retry policies
- **Handler optimization**: Improve processing speed
- **Filter refinement**: Optimize event filtering
- **Scaling adjustments**: Handle load variations

## 21. Conceptual Example

### Scenario: E-commerce Order Processing
1. **Order Placed**: Customer submits order via web application
2. **Event Published**: Order service publishes "OrderCreated" event to custom topic
3. **Event Filtering**: Multiple subscriptions filter for relevant order events
4. **Event Delivery**: Event Grid delivers to multiple handlers:
   - **Payment Service**: Processes payment via Azure Function
   - **Inventory Service**: Updates stock levels via Logic App
   - **Email Service**: Sends confirmation via SendGrid webhook
   - **Analytics Service**: Records metrics in Azure Data Explorer
5. **Dead Letter Handling**: Failed deliveries stored in Azure Storage
6. **Monitoring**: Azure Monitor tracks delivery success and performance

### Implementation Flow
```
Order API → Event Grid Topic → Event Subscriptions → Event Handlers
                ↓                        ↓
        System Metrics         Dead Letter Storage
```

This comprehensive architecture enables scalable, reliable, and maintainable event-driven applications with full observability and error handling capabilities.
