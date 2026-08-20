# Spring Boot — Microservices

Microservices architecture structures an application as a collection of independently deployable services, with each service focused on a specific business capability.

A typical ecommerce system might look like:

```text
                         API Gateway
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
     User Service       Product Service      Order Service
          |                   |                   |
       User DB            Product DB          Order DB
                                                  |
                                  +---------------+---------------+
                                  |                               |
                                  v                               v
                           Payment Service                 Inventory Service
```

---

# 1. What Are Microservices?

A microservice is an independently deployable application focused on a specific business capability.

Examples:

```text
User Service
Product Service
Order Service
Payment Service
Inventory Service
Notification Service
```

A good microservice generally has:

```text
Clear responsibility
Independent deployment
Well-defined APIs
Controlled dependencies
Independent scaling where useful
```

---

# 2. Monolith vs Microservices

## Monolith

```text
+--------------------------------------+
| User | Product | Order | Payment     |
| Inventory | Notification | ...       |
+--------------------------------------+
                  |
               Database
```

Advantages:

```text
Simple deployment
Simple local development
Simple debugging initially
Easy local transactions
Low network overhead
```

Disadvantages:

```text
Large codebase
Potential tight coupling
Independent scaling is harder
A deployment can affect the whole application
```

## Microservices

```text
User Service       -> User DB
Product Service    -> Product DB
Order Service      -> Order DB
Payment Service    -> Payment DB
```

Advantages:

```text
Independent deployment
Independent scaling
Team ownership
Clear domain boundaries
Failure isolation can be improved
```

Disadvantages:

```text
Network failures
Distributed transactions
More infrastructure
More operational complexity
Distributed debugging
Data consistency challenges
```

---

# 3. Are Microservices Always Better?

No.

Microservices introduce distributed-system complexity.

For a small application, a modular monolith can often be simpler and more effective.

Use microservices when the benefits justify the additional complexity.

Typical reasons include:

```text
Independent scaling
Independent deployment
Large teams
Strong domain boundaries
Different availability requirements
Different release cycles
```

---

# 4. Bounded Context

Services should normally be aligned with meaningful business boundaries.

For ecommerce:

```text
Catalog
Orders
Payments
Inventory
Users
```

Avoid splitting purely by technical layers:

```text
Controller Service
Repository Service
DTO Service
```

That creates distributed technical layers rather than meaningful business services.

---

# 5. Service Ownership

A service should own its business logic and data.

Example:

```text
Order Service
      |
      v
Order Database
```

Other services should normally access order data through the Order Service API or events rather than directly modifying the Order database.

---

# 6. Database per Service

A common microservice principle is:

```text
User Service
    |
 User DB

Order Service
    |
 Order DB

Payment Service
    |
 Payment DB
```

The important concept is data ownership.

---

# 7. Why Shared Databases Are Risky

Consider:

```text
Order Service ----+
                  |
              Shared DB
                  |
Payment Service --+
```

Both services now become coupled to:

```text
Tables
Schema
Indexes
Constraints
Transactions
Database changes
```

This makes independent evolution harder.

---

# 8. Database Technology

Database-per-service does not necessarily mean every service must use a different database technology.

For example:

```text
Order Service    -> MySQL database A
Payment Service  -> MySQL database B
```

is still a database-per-service architecture.

The important boundary is:

```text
Data ownership
```

---

# 9. Service Communication

Microservices commonly communicate using:

```text
Synchronous communication
Asynchronous communication
```

Synchronous:

```text
Order Service
     |
    HTTP
     |
Payment Service
     |
 Response
```

Asynchronous:

```text
Order Service
     |
     v
Message Broker
     |
     v
Payment Consumer
```

---

# 10. REST Communication

REST over HTTP is a common synchronous approach.

Example:

```http
POST /api/payments
```

Flow:

```text
Order Service
     |
 HTTP request
     v
Payment Service
     |
 HTTP response
     v
Order Service
```

---

# 11. RestClient

Modern Spring applications can use `RestClient` for synchronous HTTP communication.

Example:

```java
RestClient restClient =
    RestClient.builder()
        .baseUrl("http://payment-service")
        .build();

PaymentResponse response =
    restClient.post()
        .uri("/api/payments")
        .body(request)
        .retrieve()
        .body(PaymentResponse.class);
```

Use the HTTP client that matches the application's requirements and Spring version.

---

# 12. WebClient

`WebClient` is Spring's non-blocking HTTP client.

Example:

```java
WebClient client =
    WebClient.builder()
        .baseUrl("http://payment-service")
        .build();
```

It is especially useful for reactive applications.

Do not introduce reactive programming merely because `WebClient` exists. Use it when the application architecture benefits from non-blocking processing.

---

# 13. OpenFeign

Spring Cloud OpenFeign provides a declarative HTTP client style.

Example:

```java
@FeignClient(
    name = "payment-service"
)
public interface PaymentClient {

    @PostMapping("/api/payments")
    PaymentResponse pay(
        @RequestBody PaymentRequest request
    );
}
```

Then:

```java
PaymentResponse response =
    paymentClient.pay(request);
```

This can make service-to-service REST clients easier to read and maintain.

---

# 14. Synchronous Communication Problem

Consider:

```text
Order Service
     |
     v
Payment Service
     |
     v
Inventory Service
```

If Payment Service becomes slow:

```text
Order request
     |
Payment waits
     |
Inventory waits
     |
Order waits
```

Latency can propagate through the system.

This is one reason distributed systems need timeouts and resilience patterns.

---

# 15. Timeouts

Network calls should have appropriate timeouts.

Without timeouts:

```text
Service A
    |
    v
Service B unavailable
    |
    v
Request waits
    |
    v
Threads remain occupied
    |
    v
System becomes unhealthy
```

Common timeout concepts:

```text
Connection timeout
Response/read timeout
Connection pool timeout
```

---

# 16. Retry

Retries can help with temporary failures.

Example:

```text
Request
   |
Failure
   |
Retry
   |
Success
```

But retries can make an outage worse.

Example:

```text
100 requests
     |
5 retries each
     |
500 requests
```

This can create a retry storm.

---

# 17. Retry Best Practices

Use retries carefully:

```text
Only transient failures
Limited attempts
Exponential backoff
Jitter
Idempotent operations
Reasonable timeouts
```

Do not blindly retry every HTTP error.

---

# 18. Circuit Breaker

A circuit breaker prevents repeated calls to an unhealthy dependency.

Typical states:

```text
CLOSED
   |
Failure threshold
   v
OPEN
   |
Wait
   v
HALF_OPEN
   |
Test request
   |
+--+----------------+
|                   |
Success            Failure
|                   |
v                   v
CLOSED              OPEN
```

---

# 19. Circuit Breaker Flow

```text
Order Service
     |
     v
Payment Service
     |
Repeated failures
     |
Circuit opens
     |
Calls fail fast
```

This protects the caller from continuously waiting for an unhealthy dependency.

---

# 20. Resilience4j

Resilience4j is commonly used in Spring applications for resilience patterns.

It provides modules for:

```text
Circuit Breaker
Retry
Rate Limiter
Bulkhead
Time Limiter
```

---

# 21. Circuit Breaker Example

Conceptually:

```java
@CircuitBreaker(
    name = "paymentService",
    fallbackMethod = "paymentFallback"
)
public PaymentResponse pay(
        PaymentRequest request) {

    return paymentClient.pay(request);
}
```

Fallback:

```java
public PaymentResponse paymentFallback(
        PaymentRequest request,
        Throwable ex) {

    return PaymentResponse.failed(
        "Payment service unavailable"
    );
}
```

A fallback should represent a safe business outcome. It should not hide every failure.

---

# 22. Bulkhead

Bulkhead isolation limits resource usage by a dependency or workload.

Example:

```text
Payment calls
    |
Payment pool

Email calls
    |
Email pool
```

If email becomes slow, payment resources can remain available.

---

# 23. Rate Limiting

Rate limiting controls request volume.

Example:

```text
100 requests/second
```

When the limit is exceeded:

```text
429 Too Many Requests
```

Rate limiting can protect:

```text
APIs
Databases
External services
Login endpoints
```

---

# 24. API Gateway

An API Gateway provides a common entry point for clients.

```text
                 Client
                   |
                   v
              API Gateway
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
     User       Product       Order
   Service      Service      Service
```

Typical responsibilities:

```text
Routing
Authentication integration
Rate limiting
CORS
Request filtering
TLS termination
Observability
```

Do not put core business logic into the gateway.

---

# 25. Spring Cloud Gateway

Spring Cloud Gateway is commonly used as an API gateway in Spring-based systems.

Example routing:

```text
/api/users/**      -> User Service
/api/products/**   -> Product Service
/api/orders/**     -> Order Service
```

---

# 26. Service Discovery

In dynamic environments, service locations can change.

Instead of hardcoding:

```text
http://10.0.0.12:8081
```

a service can discover another service dynamically.

Conceptually:

```text
Service A
    |
    v
Service Registry
    ^
    |
Service B
```

---

# 27. Eureka

Netflix Eureka is a service discovery technology historically used with Spring Cloud.

Services can register:

```text
Service name
Host
Port
Instance information
```

Consumers discover:

```text
PAYMENT-SERVICE
```

and receive available instances.

In modern cloud-native environments, platform-native service discovery such as Kubernetes Services may be preferred.

---

# 28. Client-Side Load Balancing

Suppose:

```text
PAYMENT-SERVICE
    |
    +-- Instance 1
    +-- Instance 2
    +-- Instance 3
```

A client-side load balancer can distribute calls among available instances.

Spring Cloud LoadBalancer is one option in the Spring ecosystem.

---

# 29. Server-Side Load Balancing

Another architecture:

```text
Client
  |
  v
Load Balancer
  |
  +-- Instance 1
  +-- Instance 2
  +-- Instance 3
```

Examples:

```text
AWS Load Balancer
NGINX
Kubernetes Service
Cloud load balancers
```

The exact responsibility depends on the deployment platform.

---

# 30. Configuration Management

Microservices require configuration such as:

```text
Database URLs
Service URLs
Timeouts
Feature flags
Environment settings
```

Avoid hardcoding environment-specific values.

Use:

```text
application.yml
Environment variables
Secret managers
Configuration services
Platform configuration
```

---

# 31. Spring Cloud Config

Spring Cloud Config provides centralized configuration management.

Conceptually:

```text
             Config Server
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
     User        Order       Payment
   Service      Service      Service
```

Each service can retrieve environment-specific configuration.

Cloud-native platforms may also provide configuration and secret-management solutions.

---

# 32. Secrets

Never commit:

```text
Database passwords
JWT signing keys
API keys
Cloud credentials
```

to Git.

Use appropriate secret management:

```text
Environment variables
HashiCorp Vault
AWS Secrets Manager
Azure Key Vault
Kubernetes Secrets
```

---

# 33. Distributed Transactions

In a monolith:

```text
Order
Payment
Inventory
```

may potentially use one local database transaction.

In microservices:

```text
Order DB
Payment DB
Inventory DB
```

a single local transaction does not automatically span all databases.

This is a major microservices challenge.

---

# 34. Saga Pattern

Saga breaks a distributed business transaction into local transactions.

Example:

```text
Create Order
     |
     v
Reserve Inventory
     |
     v
Process Payment
     |
     v
Confirm Order
```

If Payment fails:

```text
Cancel Order
     |
     v
Release Inventory
```

These are compensating actions.

---

# 35. Choreography Saga

Services react to events.

Example:

```text
OrderCreated
     |
     v
Inventory Service
     |
InventoryReserved
     |
     v
Payment Service
     |
PaymentCompleted
```

There is no central coordinator.

---

# 36. Orchestration Saga

A central orchestrator manages the workflow.

```text
             Order Orchestrator
              /       |                    v        v        v
        Inventory   Payment    Order
```

The orchestrator decides:

```text
Next action
Retry
Compensation
Failure handling
```

---

# 37. Saga Tradeoff

Choreography:

```text
Less central coordination
Good for event-driven workflows
Can become difficult to understand as workflows grow
```

Orchestration:

```text
Central visibility
Clear workflow
Orchestrator becomes an important component
```

---

# 38. Event-Driven Architecture

Services can communicate using events.

Example:

```text
Order Service
     |
     v
OrderCreated
     |
     v
Message Broker
     |
     +------------------+
     |                  |
     v                  v
Inventory Service   Notification Service
```

This reduces synchronous coupling.

---

# 39. Message Brokers

Common technologies include:

```text
Apache Kafka
RabbitMQ
AWS SQS/SNS
Azure Service Bus
```

Choose based on:

```text
Throughput
Ordering requirements
Delivery guarantees
Routing
Operational requirements
```

---

# 40. Kafka

Kafka is commonly used for:

```text
Event streaming
High-throughput messaging
Event-driven architectures
Data pipelines
```

Basic flow:

```text
Producer
    |
    v
Kafka Topic
    |
    v
Consumer
```

---

# 41. Kafka Topic

Example topic:

```text
orders
```

Possible events:

```text
OrderCreated
OrderPaid
OrderCancelled
```

Consumers subscribe to topics.

---

# 42. Kafka Consumer Groups

Example:

```text
orders topic
     |
+----+----+
|         |
v         v
Consumer A Consumer B
```

Partitions can be distributed across consumers in the same consumer group.

This allows horizontal scaling.

---

# 43. Eventual Consistency

Distributed systems often use eventual consistency.

Example:

```text
Order created
     |
     v
Event published
     |
     v
Inventory processes event
     |
     v
Inventory updated
```

For a short period:

```text
Order DB       -> updated
Inventory DB   -> not yet updated
```

Eventually the services converge.

---

# 44. Strong vs Eventual Consistency

Strong consistency:

```text
Reads reflect the latest committed state according to the system's consistency guarantees.
```

Eventual consistency:

```text
Updates propagate asynchronously and different services can temporarily have different views.
```

Choose the consistency model based on business requirements.

---

# 45. Outbox Pattern

Problem:

```text
Save order
     |
     v
Publish event
```

What if:

```text
Database save succeeds
Event publishing fails
```

The database contains the order but the event is lost.

The Outbox Pattern addresses this by storing the business change and an event record transactionally.

---

# 46. Outbox Flow

```text
Order Service
     |
     +-- Save Order
     |
     +-- Save Outbox Event
             |
             v
       Same DB transaction
             |
             v
       Outbox Publisher
             |
             v
       Message Broker
```

This reduces the gap between database state and event publication.

---

# 47. Idempotency

Distributed systems can deliver messages more than once.

Example:

```text
PaymentCompleted
PaymentCompleted
```

The consumer should safely handle duplicates.

Example:

```text
eventId = ABC123
```

Before processing:

```text
Already processed ABC123?
```

If yes:

```text
Ignore duplicate
```

---

# 48. At-Least-Once Delivery

Many messaging systems provide at-least-once delivery.

This means:

```text
Message should not be lost
but duplicate delivery may happen
```

Therefore consumers should generally be designed to be idempotent.

---

# 49. Dead Letter Queue

Repeatedly failing messages can be moved to a dead-letter destination.

```text
Main Queue
    |
Retry
    |
Retry
    |
Retry
    |
Dead Letter Queue
```

This allows investigation without blocking normal processing indefinitely.

---

# 50. Retry vs DLQ

Retry is appropriate for:

```text
Temporary failures
Network problems
Transient dependency issues
```

DLQ is appropriate for:

```text
Repeated failures
Invalid messages
Messages requiring manual investigation
```

Do not retry permanently invalid messages forever.

---

# 51. Distributed Logging

Microservices produce logs independently.

```text
Order Service
Payment Service
Inventory Service
```

Use:

```text
Centralized logging
Structured logs
Correlation IDs
```

to reconstruct request flows.

---

# 52. Correlation ID

Example:

```text
X-Correlation-Id: ABC123
```

Logs:

```text
Order Service   -> ABC123
Payment Service -> ABC123
Inventory       -> ABC123
```

This makes distributed troubleshooting easier.

---

# 53. Distributed Tracing

A trace represents one request across multiple services.

Example:

```text
Trace ABC123
 |
 +-- Gateway span
 |
 +-- Order span
 |
 +-- Payment span
 |
 +-- Inventory span
```

Common technologies:

```text
OpenTelemetry
Jaeger
Zipkin
APM platforms
```

---

# 54. OpenTelemetry

OpenTelemetry provides standardized observability instrumentation for:

```text
Traces
Metrics
Logs
```

It can propagate trace context between services.

---

# 55. Service-to-Service Authentication

Internal services should not automatically trust every caller.

Possible approaches:

```text
OAuth2
JWT
mTLS
Service identity
Cloud IAM
```

Validate where appropriate:

```text
Caller identity
Audience
Permissions
```

---

# 56. mTLS

mTLS means:

```text
Mutual TLS
```

Both services authenticate each other using certificates.

```text
Service A  <----TLS---->  Service B
```

It is useful when strong service identity is required.

---

# 57. API Versioning

APIs evolve.

Examples:

```text
/api/v1/products
/api/v2/products
```

or header/media-type based versioning.

Consider:

```text
Backward compatibility
Deprecation
Consumer migration
Contract testing
```

---

# 58. Backward Compatibility

Suppose an API changes:

```text
Old:
amount

New:
paymentAmount
```

Existing consumers may break.

Prefer additive changes when possible:

```text
Add new field
Keep old field temporarily
Migrate consumers
Remove old field later
```

---

# 59. Service Contract

A service contract defines:

```text
Request
Response
Status codes
Headers
Schema
Error behavior
```

Contract testing helps detect breaking changes before deployment.

---

# 60. API Gateway vs Service Discovery

They solve different problems.

API Gateway:

```text
Client entry point
Routing
Cross-cutting API policies
```

Service discovery:

```text
Locate service instances
```

They can be used together.

---

# 61. API Gateway vs Load Balancer

Load balancer:

```text
Distribute traffic
```

API Gateway:

```text
Routing
Authentication integration
Rate limiting
API policies
Observability
```

The exact responsibilities depend on the platform.

---

# 62. Health Checks

Each service should expose health information.

Typical endpoints:

```text
/actuator/health
/actuator/health/liveness
/actuator/health/readiness
```

Monitoring can determine:

```text
Service alive?
Service ready?
Important dependency healthy?
```

---

# 63. Resilience Example

```text
Order Service
     |
     v
Payment Service
     |
  timeout
     |
     v
Retry with backoff
     |
 failure
     |
     v
Circuit Breaker
     |
     v
Safe failure / fallback
```

This prevents a dependency failure from consuming resources indefinitely.

---

# 64. Timeouts First

A common resilience strategy is:

```text
Timeout
   |
Retry if appropriate
   |
Circuit breaker
   |
Fallback
```

A retry without a timeout can still wait too long.

---

# 65. Failure Isolation

Suppose Notification Service is down.

Ideally:

```text
Order creation
     |
     v
OrderCreated event
     |
     v
Notification Service
```

Order creation should not necessarily fail simply because email delivery failed.

This separates critical business operations from non-critical asynchronous work.

---

# 66. Avoid Distributed Monolith

A distributed monolith may look like:

```text
Service A
   |
Service B
   |
Service C
   |
Service D
```

where every request depends on all services.

Warning signs:

```text
Tight coupling
Synchronized deployments
Many synchronous calls
Shared database
Difficult local development
```

You get microservice infrastructure without real service independence.

---

# 67. Caching

Microservices can use caching for frequently accessed data.

Example:

```text
Product Service
     |
     v
Redis
     |
     v
Database
```

Good candidates:

```text
Product catalog
Reference data
Frequently requested information
Short-lived data
```

Caching introduces:

```text
Invalidation
Stale data
Consistency
Memory management
```

---

# 68. Redis

Redis can be used for:

```text
Caching
Distributed locks
Rate limiting
Short-lived data
Session-related use cases
```

Do not automatically treat Redis as the durable source of truth.

---

# 69. Scheduled Jobs

Suppose a service has:

```java
@Scheduled(...)
```

and three instances:

```text
Instance A
Instance B
Instance C
```

The scheduled task may run on multiple instances unless the architecture prevents duplicate execution.

Possible solutions:

```text
Distributed lock
Dedicated scheduler
Platform scheduler
Idempotent job design
```

---

# 70. Containerization

Microservices are commonly packaged as containers.

```text
Docker Image
     |
     v
Container
     |
     v
Kubernetes / Cloud
```

Benefits:

```text
Consistent runtime
Isolation
Easy deployment
Scalability
```

---

# 71. Kubernetes

Kubernetes can provide:

```text
Deployment
Scaling
Service discovery
Health probes
Rolling updates
Configuration
Secrets
```

Example:

```text
Deployment
    |
    v
Pods
    |
    v
Kubernetes Service
```

---

# 72. Horizontal Scaling

If Order Service has high traffic:

```text
1 instance
    |
    v
3 instances
```

A load-balancing layer distributes traffic.

Stateless services are generally easier to scale horizontally.

---

# 73. Stateless Service

A stateless service does not depend on local in-memory session state between requests.

Example:

```text
Request 1 -> Instance A
Request 2 -> Instance B
```

Both instances can process requests because required state is externalized or carried by the request/token.

---

# 74. Deployment Strategies

Common strategies:

```text
Rolling deployment
Blue-green deployment
Canary deployment
```

---

# 75. Rolling Deployment

Example:

```text
Old: A A A
New: N

Old: A A
New: N N

Old: A
New: N N N

Old: -
New: N N N
```

Instances are gradually replaced.

---

# 76. Blue-Green Deployment

```text
Blue  -> Current
Green -> New
```

Initially:

```text
Client -> Blue
```

After validation:

```text
Client -> Green
```

Rollback can switch traffic back to Blue.

---

# 77. Canary Deployment

Send a small percentage of traffic to the new version:

```text
95% -> old
5%  -> new
```

Monitor:

```text
Errors
Latency
Business metrics
```

Then gradually increase new-version traffic.

---

# 78. Distributed Failure Modes

Expect:

```text
Network timeout
Service unavailable
Partial failure
Duplicate messages
Out-of-order events
Database failure
Message broker failure
Deployment mismatch
```

Microservices should be designed with failure in mind.

---

# 79. Partial Failure

A distributed system can have:

```text
Order Service     -> UP
Payment Service   -> DOWN
Inventory Service -> UP
```

The system is not simply:

```text
UP
```

or:

```text
DOWN
```

This is why resilience and observability are important.

---

# 80. CAP Theorem

CAP discusses a distributed system's tradeoffs involving:

```text
Consistency
Availability
Partition tolerance
```

When a network partition occurs, a distributed system cannot simultaneously guarantee both strong consistency and availability under the classical CAP framing.

In interviews, always clarify the consistency model and failure assumptions.

---

# 81. Event Ordering

Suppose events are:

```text
OrderCreated
OrderPaid
OrderShipped
```

If:

```text
OrderShipped
```

arrives before:

```text
OrderPaid
```

the consumer needs a strategy.

Possible approaches:

```text
Partitioning/keying
Sequence numbers
State validation
Retry
Buffering
Idempotency
```

---

# 82. Exactly Once

Be careful with:

```text
Exactly once
```

End-to-end exactly-once business effects are difficult in distributed systems.

A practical architecture often uses:

```text
At-least-once delivery
+
Idempotent consumers
+
Transactional boundaries
```

to achieve effectively-once business behavior where appropriate.

---

# 83. Ecommerce Microservices Design

A reasonable starting point:

```text
                       API Gateway
                            |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
   User Service       Product Service      Order Service
        |                   |                   |
     User DB            Product DB          Order DB
                                                |
                                  +-------------+-------------+
                                  |                           |
                                  v                           v
                           Payment Service             Inventory Service
                                  |                           |
                              Payment DB                 Inventory DB
```

Events:

```text
OrderCreated
InventoryReserved
PaymentCompleted
OrderCancelled
```

---

# 84. Ecommerce Order Flow

Example:

```text
Client
  |
  v
API Gateway
  |
  v
Order Service
  |
  v
Create Pending Order
  |
  v
OrderCreated Event
  |
  v
Inventory Service
  |
  v
Reserve Stock
  |
  v
Payment Service
  |
  v
Process Payment
  |
  v
Confirm Order
```

A production design must also define:

```text
Timeouts
Retries
Compensation
Idempotency
Failure states
Observability
```

---

# 85. Payment Failure

Suppose:

```text
Order created
     |
Inventory reserved
     |
Payment fails
```

Possible compensation:

```text
Payment failed
     |
Cancel order
     |
Release inventory
```

This is a Saga-style workflow.

---

# 86. Notification Failure

Suppose:

```text
Order created
     |
Email notification fails
```

The order normally should not be rolled back only because email failed.

Better:

```text
OrderCreated
     |
     v
Notification Service
     |
     +-- Retry
     |
     +-- DLQ
```

---

# 87. Microservices Security

Security exists at multiple boundaries:

```text
Client -> Gateway
Gateway -> Services
Service -> Service
Service -> Database
```

Possible mechanisms:

```text
OAuth2/JWT
mTLS
Network policies
Service identity
Secret management
Authorization
```

---

# 88. Microservices Observability

At minimum:

```text
Centralized logs
Metrics
Distributed traces
Correlation IDs
Health checks
Alerts
```

Without observability, distributed debugging becomes very difficult.

---

# 89. Microservices Testing

Test at multiple levels:

```text
Unit
Integration
Contract
Component
End-to-end
```

Especially test:

```text
Service contracts
Failure handling
Retries
Idempotency
Event processing
Security
```

---

# 90. Consumer-Driven Contract Testing

A consumer can define expectations about a provider API.

Example:

```text
Order Service
     |
expects
     |
Payment Service
     |
POST /payments
     |
Specific response schema
```

Contract testing can detect breaking provider changes before deployment.

---

# 91. Distributed Tracing Example

```text
Trace ABC123

Gateway         20 ms
Order Service   80 ms
Payment        500 ms
Inventory       40 ms
```

This immediately suggests Payment Service is contributing most of the latency.

---

# 92. Production Checklist

```text
□ Clear service boundaries
□ Service-owned data
□ API contracts
□ Timeouts
□ Controlled retries
□ Circuit breakers where appropriate
□ Rate limiting
□ Bulkheads where useful
□ Idempotency
□ Consistency strategy
□ Saga for distributed workflows
□ Outbox for reliable events
□ Dead-letter handling
□ Centralized logs
□ Metrics
□ Distributed tracing
□ Correlation IDs
□ Health checks
□ Secret management
□ API versioning
□ Contract testing
□ CI/CD
□ Containerization
□ Horizontal scaling
□ Failure testing
```

---

# 93. Interview: Why Microservices?

> I would use microservices when independent deployment, scaling, team ownership, or clear domain boundaries justify the additional distributed-system complexity. I wouldn't split a small application into microservices just for the sake of using them.

---

# 94. Interview: What Is Service Discovery?

> Service discovery allows services to locate available instances dynamically instead of relying on hardcoded host and port information.

---

# 95. Interview: What Is an API Gateway?

> An API Gateway provides a common entry point for clients and can handle routing and cross-cutting concerns such as authentication integration, rate limiting, and observability.

---

# 96. Interview: What Is a Circuit Breaker?

> A circuit breaker stops repeatedly calling an unhealthy dependency. After enough failures it opens and fails fast, then later allows limited test calls to determine whether the dependency has recovered.

---

# 97. Interview: What Is a Saga?

> A Saga coordinates a distributed business transaction as a sequence of local transactions. If a later step fails, compensating actions are performed for earlier steps.

---

# 98. Interview: Choreography vs Orchestration

> In choreography, services react to events without a central coordinator. In orchestration, a coordinator explicitly manages the workflow. Choreography can be simpler initially, while orchestration gives clearer control for complex workflows.

---

# 99. Interview: What Is Eventual Consistency?

> Eventual consistency means different services may temporarily have different views of data, but after asynchronous updates propagate, the system converges to a consistent state.

---

# 100. Interview: What Is the Outbox Pattern?

> The Outbox Pattern stores the business change and the event to publish in the same local database transaction. A separate publisher then sends the event to the broker, reducing the chance of saving data while losing the corresponding event.

---

# 101. Interview: Why Is Idempotency Important?

> Distributed systems can retry requests or deliver messages more than once. Idempotent processing makes repeated delivery safe and prevents duplicate business effects.

---

# 102. Final Mental Model

```text
                         Client
                           |
                           v
                      API Gateway
                           |
       +-------------------+-------------------+
       |                   |                   |
       v                   v                   v
   User Service       Product Service      Order Service
       |                   |                   |
    User DB            Product DB          Order DB
                                               |
                                  +------------+------------+
                                  |                         |
                                  v                         v
                             Payment                    Inventory
                             Service                     Service
                                  |                         |
                              Payment DB               Inventory DB

                    Message Broker / Events

                 Logs + Metrics + Traces + Alerts
```

---

# 103. Final Interview Rule

> **I design microservices around business capabilities, keep each service responsible for its own data, use APIs or events for communication, and design explicitly for distributed failures. In production, I focus on timeouts, controlled retries, circuit breakers, idempotency, observability, and clear data-consistency strategies rather than assuming the network is reliable.**

Next:

```text
13 Spring Boot Microservices
      ↓
14 Spring Cloud & Distributed Systems
```
