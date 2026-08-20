# Spring Boot — Microservices

This file covers the practical Spring Boot microservices concepts commonly asked in Java backend interviews.

The focus is on understanding the architecture, communication, resilience, data consistency, observability, and deployment concerns.

---

# 1. What Are Microservices?

Microservices architecture breaks a large application into smaller independently deployable services organized around business capabilities.

Example ecommerce system:

```text
User Service
Product Service
Cart Service
Order Service
Payment Service
Inventory Service
Notification Service
```

Each service can own a specific business responsibility.

---

# 2. Monolith vs Microservices

Monolith:

```text
One application
    |
+---+---+---+
|   |   |   |
User Order Payment
```

Microservices:

```text
User Service
     |
Order Service
     |
Payment Service
     |
Inventory Service
```

The main difference is deployment and ownership boundaries, not simply the number of classes or modules.

---

# 3. Advantages of Microservices

Common benefits:

```text
Independent deployment
Independent scaling
Business-domain separation
Team autonomy
Technology flexibility
Failure isolation
Smaller codebases
```

But these benefits come with additional operational complexity.

---

# 4. Disadvantages of Microservices

Important tradeoffs:

```text
Network failures
Distributed transactions
Eventual consistency
More deployments
More monitoring
Service discovery
API versioning
Distributed debugging
Infrastructure cost
Operational complexity
```

Interview answer:

> Microservices solve some scaling and organizational problems, but they also turn in-process calls into network calls, so reliability, observability, security, and data consistency become much more important.

---

# 5. When Should You Use Microservices?

Microservices make more sense when there is a real need for:

```text
Independent scaling
Independent deployments
Large teams
Strong domain boundaries
Different availability requirements
Different release cycles
```

For a small application, a modular monolith may be simpler.

---

# 6. Modular Monolith

A modular monolith keeps one deployable application while enforcing strong internal boundaries.

Example:

```text
Single Application

+------------------+
| User Module      |
| Product Module   |
| Order Module     |
| Payment Module   |
+------------------+
```

This can provide many organizational benefits without immediately introducing network complexity.

---

# 7. Bounded Context

A bounded context is a domain boundary where a model has a specific meaning.

Example:

```text
Order Context
    Order
    OrderItem
    Customer

Payment Context
    Payment
    Transaction
```

The same business concept may have different models in different contexts.

---

# 8. Database per Service

A common microservices principle is:

```text
Order Service
     ↓
Order DB

Payment Service
     ↓
Payment DB

Inventory Service
     ↓
Inventory DB
```

This reduces direct coupling between services.

Avoid:

```text
Multiple services
      ↓
Same tables
```

because it creates hidden coupling.

---

# 9. Why Shared Databases Are Risky

Suppose:

```text
Order Service
Payment Service
Inventory Service
```

all directly update:

```text
orders
payments
inventory
```

Now one schema change can break multiple services.

You also lose clear ownership.

---

# 10. Data Ownership

Each service should ideally own its data.

Example:

```text
Order Service
→ owns orders

Inventory Service
→ owns inventory

Payment Service
→ owns payments
```

Other services interact through:

```text
APIs
Events
Commands
```

rather than directly querying another service's database.

---

# 11. Synchronous Communication

Example:

```text
Order Service
     |
     | HTTP
     v
Payment Service
```

Common technologies:

```text
REST
HTTP
gRPC
OpenFeign
RestClient
WebClient
```

Synchronous communication is simple but couples the request to dependency availability and latency.

---

# 12. Asynchronous Communication

Example:

```text
Order Service
     |
     | OrderCreated
     v
Kafka
     |
+----+----+
|         |
Inventory Notification
```

The producer doesn't necessarily wait for consumers to finish.

---

# 13. REST Communication

Example:

```http
POST /payments
```

Request:

```json
{
  "orderId": 1001,
  "amount": 4999
}
```

Response:

```json
{
  "paymentId": "PAY-1001",
  "status": "SUCCESS"
}
```

---

# 14. OpenFeign

OpenFeign provides a declarative HTTP client.

Example:

```java
@FeignClient(
    name = "payment-service"
)
public interface PaymentClient {

    @PostMapping("/payments")
    PaymentResponse pay(
        @RequestBody PaymentRequest request
    );
}
```

The calling code can work with an interface rather than manually constructing HTTP requests.

---

# 15. RestClient

For synchronous HTTP calls, Spring provides `RestClient` in modern Spring applications.

Conceptually:

```java
PaymentResponse response =
    restClient.post()
        .uri("/payments")
        .body(request)
        .retrieve()
        .body(PaymentResponse.class);
```

Use an appropriate HTTP client based on the application architecture.

---

# 16. WebClient

`WebClient` is Spring's reactive HTTP client.

It supports:

```text
Non-blocking I/O
Reactive programming
Streaming
```

Do not use reactive APIs merely because they sound faster. The application should benefit from a reactive architecture and workload.

---

# 17. REST vs Messaging

REST:

```text
Request
 ↓
Wait
 ↓
Response
```

Messaging:

```text
Publish
 ↓
Continue
 ↓
Consumer processes later
```

Use REST when an immediate response is required.

Use messaging when asynchronous processing and decoupling are beneficial.

---

# 18. API Gateway

A gateway provides a common entry point.

```text
Client
  |
  v
API Gateway
  |
+---+---+---+
|   |   |   |
User Order Product
```

Responsibilities can include:

```text
Routing
Authentication integration
Rate limiting
CORS
Request filtering
Observability
```

Avoid putting core business logic in the gateway.

---

# 19. Spring Cloud Gateway

Spring Cloud Gateway is commonly used for gateway functionality in Spring ecosystems.

Example route concept:

```text
/api/orders/**
       ↓
Order Service

/api/products/**
       ↓
Product Service
```

---

# 20. Service Discovery

Service discovery allows services to find other service instances.

Conceptually:

```text
Order Service
      |
      v
Service Registry
      |
      v
Payment Service
  instance 1
  instance 2
  instance 3
```

---

# 21. Eureka

Eureka is a service-discovery technology historically used with Spring Cloud applications.

Services can register themselves and discover other services.

In Kubernetes environments, native Kubernetes service discovery is often preferred.

---

# 22. Kubernetes Service Discovery

In Kubernetes:

```text
order-service
payment-service
inventory-service
```

can be exposed through Kubernetes Services.

Applications can communicate using service names rather than hard-coded pod IP addresses.

---

# 23. Client-Side Load Balancing

Suppose:

```text
Payment Service
  instance 1
  instance 2
  instance 3
```

A client-side load balancer can select an available instance.

Spring Cloud LoadBalancer can provide client-side load-balancing capabilities.

---

# 24. Server-Side Load Balancing

Architecture:

```text
Client
  ↓
Load Balancer
  ↓
+---+---+---+
|   |   |   |
A   B   C
```

The load balancer selects the service instance.

Examples in real systems can include:

```text
Cloud load balancers
Ingress controllers
Reverse proxies
```

---

# 25. API Versioning

When an API changes incompatibly:

```text
/api/v1/orders
/api/v2/orders
```

Versioning helps consumers migrate safely.

Avoid breaking existing consumers without a migration strategy.

---

# 26. Backward Compatibility

Suppose old clients send:

```json
{
  "name": "Phone"
}
```

Adding an optional field:

```json
{
  "name": "Phone",
  "description": "..."
}
```

is generally easier to evolve than removing or changing the meaning of existing fields.

---

# 27. Distributed Transactions

In a monolith:

```text
Order
+
Inventory
+
Payment
```

could potentially be handled within one database transaction.

In microservices:

```text
Order DB
Inventory DB
Payment DB
```

a single ACID transaction normally cannot span all services in the same simple way.

---

# 28. Saga Pattern

Saga coordinates a business workflow using local transactions.

Example:

```text
Create Order
     ↓
Reserve Inventory
     ↓
Process Payment
     ↓
Confirm Order
```

If payment fails:

```text
Cancel Order
     ↓
Release Inventory
```

---

# 29. Choreography Saga

Services react to events.

Example:

```text
OrderCreated
     ↓
InventoryReserved
     ↓
PaymentRequested
     ↓
PaymentCompleted
```

There may be no central orchestrator.

Advantages:

```text
Loosely coupled
Simple for smaller workflows
```

Disadvantages:

```text
Harder to visualize
Complex event chains
Difficult debugging
```

---

# 30. Orchestration Saga

A central orchestrator coordinates the workflow.

```text
Order Saga
   |
   +--> Inventory
   |
   +--> Payment
   |
   +--> Notification
```

Advantages:

```text
Central workflow visibility
Easier to understand complex flows
```

Tradeoff:

```text
Orchestrator becomes an important component
```

---

# 31. Eventual Consistency

Microservices often use asynchronous communication.

Example:

```text
Order DB
   ↓
OrderCreated
   ↓
Inventory
```

There may be a short period where:

```text
Order = CREATED
Inventory = old state
```

Eventually the event is processed and the state converges.

---

# 32. CAP Theorem

In the presence of a network partition, a distributed system must trade off between:

```text
Consistency
Availability
```

CAP is about systems under network partition, not simply saying that every system can only choose two of C, A, and P at all times.

---

# 33. Consistency

Strong consistency means clients observe updates according to the system's consistency guarantees.

Eventual consistency means updates may become visible at different times but eventually converge if the system continues operating normally.

---

# 34. Idempotency

An operation is idempotent if repeating it produces the same intended business effect.

Example:

```text
POST /payment
Idempotency-Key: ABC123
```

First request:

```text
Payment created
```

Second request:

```text
Existing result returned
```

No duplicate payment.

---

# 35. Why Idempotency Matters

Distributed systems can retry requests.

Example:

```text
Payment created
     ↓
Response lost
     ↓
Client retries
```

Without idempotency:

```text
Two payments
```

With idempotency:

```text
One payment
```

---

# 36. Idempotency Implementation

Possible design:

```text
Request
   ↓
Idempotency key
   ↓
Check storage
   |
   +-- exists → return previous result
   |
   +-- absent → process
                 ↓
              store result
```

Use a unique constraint where appropriate.

---

# 37. Outbox Pattern

Problem:

```text
Update database
     ↓
Publish event
     X
Broker unavailable
```

Now the database changed but the event was lost.

Outbox solution:

```text
Business data
+
Outbox event
```

are written in the same local transaction.

---

# 38. Outbox Flow

```text
Order Service
     |
     +--> orders table
     |
     +--> outbox table
              |
              v
       Outbox Publisher
              |
              v
            Kafka
```

---

# 39. Outbox Status

An outbox record can contain:

```text
event_id
aggregate_id
event_type
payload
status
created_at
processed_at
```

Possible states:

```text
PENDING
PROCESSED
FAILED
```

Exact states depend on implementation.

---

# 40. Inbox Pattern

The Inbox Pattern helps consumers handle duplicate messages.

Conceptually:

```text
Message
  ↓
Check event ID
  ↓
Already processed?
  ├─ yes → ignore/reuse result
  └─ no → process + record ID
```

This supports idempotent consumers.

---

# 41. Kafka

Kafka is a distributed event-streaming platform.

Core concepts:

```text
Topic
Partition
Producer
Consumer
Consumer group
Offset
Broker
```

---

# 42. Kafka Topic

A topic is a logical stream of records.

Example:

```text
order-events
```

Events can include:

```text
OrderCreated
OrderCancelled
OrderPaid
```

---

# 43. Kafka Partition

A topic is divided into partitions.

Example:

```text
order-events

Partition 0
Partition 1
Partition 2
```

Partitions enable:

```text
Parallelism
Scalability
Ordering within a partition
```

---

# 44. Kafka Ordering

Kafka guarantees ordering within a partition.

It does not automatically provide global ordering across all partitions.

If ordering matters for an entity, choose a suitable message key.

Example:

```text
key = orderId
```

This can keep events for the same order in the same partition.

---

# 45. Consumer Group

A consumer group allows multiple consumers to share partitions.

Example:

```text
Topic
P0 P1 P2 P3

Consumer A → P0 P1
Consumer B → P2 P3
```

This provides parallel processing.

---

# 46. Consumer Scaling

If:

```text
4 partitions
```

then a consumer group can have at most:

```text
4 actively consuming consumers
```

for those partitions at a time.

Adding more consumers than partitions does not increase parallelism for that topic/group.

---

# 47. Kafka Offset

A consumer tracks its position using offsets.

Conceptually:

```text
Partition
0 1 2 3 4 5 6
        ↑
     offset
```

Offset management determines where consumption resumes.

---

# 48. At-Least-Once Delivery

Many practical Kafka consumer designs use at-least-once processing.

This means:

```text
Message may be processed more than once
```

Therefore consumers should be designed to handle duplicates safely.

---

# 49. Exactly-Once

Exactly-once semantics are more complex than simply setting one configuration option.

The actual guarantee depends on:

```text
Kafka processing model
Producer configuration
Consumer behavior
External database effects
Transaction boundaries
```

Don't claim exactly-once for an entire distributed business workflow without explaining the complete design.

---

# 50. Dead Letter Topic

If a message repeatedly fails:

```text
Main Topic
    ↓
Consumer
    ↓
Failure
    ↓
Retry
    ↓
Failure
    ↓
Dead Letter Topic
```

This prevents a permanently bad message from blocking normal processing indefinitely.

---

# 51. Retry Topic

A retry mechanism can use:

```text
Main Topic
    ↓
Retry Topic
    ↓
Consumer
```

with controlled delays.

The exact implementation depends on the messaging platform and Spring Kafka setup.

---

# 52. Poison Message

A poison message consistently fails processing.

Example:

```text
Invalid payload
```

If the consumer retries forever:

```text
Consumer stuck
```

Use:

```text
Bounded retries
Dead letter handling
Monitoring
Manual remediation
```

---

# 53. Service-to-Service Authentication

Microservices should authenticate calls between services.

Options include:

```text
OAuth2 access tokens
mTLS
Service identity
API keys in limited scenarios
```

For modern distributed systems, strong service identity and short-lived credentials are preferable to static shared secrets where practical.

---

# 54. mTLS

mTLS means:

```text
Mutual TLS
```

Both sides authenticate each other using certificates.

```text
Service A ←→ Service B
   cert       cert
```

It provides strong transport-level service identity.

---

# 55. Zero Trust

Zero Trust generally means:

```text
Do not automatically trust network location
```

Every request should be evaluated according to:

```text
Identity
Authentication
Authorization
Context
Policy
```

---

# 56. Centralized Configuration

Microservices often have configuration such as:

```text
Database URL
Kafka settings
Feature flags
External service URLs
Timeouts
```

Use appropriate configuration management rather than hardcoding values.

Never store production secrets in source control.

---

# 57. Spring Cloud Config

Spring Cloud Config can centralize application configuration.

Conceptually:

```text
Config Server
      |
+-----+-----+
|     |     |
Order Payment Inventory
```

Modern cloud-native environments may also use platform-native configuration and secret systems.

---

# 58. Configuration vs Secrets

Configuration:

```text
Timeout = 2s
Feature enabled = true
```

Secret:

```text
Database password
Private key
API credential
```

Secrets need stronger protection and access control.

---

# 59. Distributed Logging

Each service should produce structured logs containing useful context.

Example:

```json
{
  "service": "order-service",
  "traceId": "abc123",
  "orderId": "1001",
  "event": "ORDER_CREATED"
}
```

Avoid logging sensitive credentials or tokens.

---

# 60. Correlation ID

A correlation ID helps connect logs from multiple services.

```text
Request
 ↓
Gateway
 ↓ correlationId=ABC
Order
 ↓ correlationId=ABC
Payment
```

All services log:

```text
ABC
```

---

# 61. Distributed Tracing

Tracing provides a view of a request across services.

```text
Trace
 |
 +-- Gateway
 |
 +-- Order Service
 |
 +-- Payment Service
 |
 +-- Inventory Service
```

OpenTelemetry is a common standard for instrumentation.

---

# 62. Metrics

Important microservice metrics:

```text
Request rate
Error rate
p95/p99 latency
CPU
Memory
DB pool usage
Kafka lag
Cache hit ratio
External API latency
```

---

# 63. Health Checks

Expose health information appropriate for the environment.

Typical concepts:

```text
Liveness
Readiness
```

Readiness should reflect whether the instance should receive traffic, not simply whether the process exists.

---

# 64. Circuit Breaker

A circuit breaker protects services from repeatedly calling unhealthy dependencies.

States:

```text
CLOSED
   ↓ failures
OPEN
   ↓ wait
HALF_OPEN
   ↓ success
CLOSED
```

---

# 65. Retry with Backoff

Instead of:

```text
Retry immediately
Retry immediately
Retry immediately
```

use:

```text
1st retry → short delay
2nd retry → longer delay
3rd retry → longer delay
```

Add jitter to reduce synchronized retry bursts.

---

# 66. Timeout

Every network call should have an appropriate timeout.

Example:

```text
Order → Payment
Connect timeout = 500 ms
Response timeout = 2 sec
```

Exact values should come from measured service-level requirements.

---

# 67. Bulkhead

Bulkhead isolation prevents one dependency from consuming all shared resources.

Example:

```text
Payment requests → dedicated pool
Recommendation requests → separate pool
```

If recommendations become slow, payment traffic can remain protected.

---

# 68. Rate Limiting

Rate limiting controls incoming traffic.

Example:

```text
100 requests/second/client
```

When the limit is exceeded:

```text
429 Too Many Requests
```

Use limits appropriate to the business and endpoint.

---

# 69. Load Shedding

When a service is overloaded:

```text
Accept everything
→
Queue grows
→
Latency explodes
→
System fails
```

Load shedding can reject or defer lower-priority work to protect critical operations.

---

# 70. Backpressure

Backpressure prevents a producer from overwhelming a consumer.

Example:

```text
Producer
   ↓
Queue
   ↓
Consumer
```

If consumer capacity is limited:

```text
Bounded queue
Rate control
Scaling
Load shedding
```

can help.

---

# 71. Failure Isolation

A failure in one service should not automatically bring down the entire system.

Example:

```text
Recommendation Service DOWN
```

The product page may still work:

```text
Products → available
Recommendations → unavailable/fallback
```

This is graceful degradation.

---

# 72. Graceful Degradation

Example:

```text
Payment service unavailable
```

Instead of:

```text
Entire application crashes
```

the system may:

```text
Mark payment as pending
Queue request
Return controlled response
```

depending on business requirements.

---

# 73. Distributed Cache

Redis can be shared across service instances.

```text
Order Instance A
Order Instance B
Order Instance C
        |
        v
      Redis
```

This is useful when cached state must be shared.

---

# 74. Distributed Lock

Sometimes only one instance should perform a particular operation.

Example:

```text
Scheduled job
   ↓
3 service instances
   ↓
Only one should execute
```

A distributed lock can coordinate this.

Use carefully because distributed locks add complexity and failure modes.

---

# 75. Scheduled Jobs in Microservices

If every instance runs:

```text
@Scheduled
```

then every instance may execute the job.

Possible approaches:

```text
Distributed lock
Leader election
Dedicated worker
Queue-based scheduling
Platform scheduler
```

Choose based on the requirement.

---

# 76. API Gateway vs Service Mesh

API Gateway:

```text
North-south traffic
Client → Services
```

Service mesh:

```text
East-west traffic
Service ↔ Service
```

A service mesh can provide:

```text
mTLS
Traffic management
Retries
Observability
Policy
```

depending on the platform.

---

# 77. Docker

Microservices are commonly packaged as containers.

Conceptually:

```text
Docker Image
     ↓
Container
     ↓
Spring Boot Application
```

Benefits:

```text
Consistent environment
Easy deployment
Isolation
Portable packaging
```

---

# 78. Kubernetes

Kubernetes can manage:

```text
Containers
Deployments
Services
Scaling
Rolling updates
Health checks
Configuration
Secrets
```

Example:

```text
Order Deployment
     ↓
Order Pods
     ↓
Order Service
```

---

# 79. Horizontal Pod Autoscaling

Kubernetes can scale pods based on metrics.

Example:

```text
CPU > threshold
     ↓
3 pods → 6 pods
```

Autoscaling should consider application behavior and bottlenecks, not just CPU.

---

# 80. Rolling Deployment

A rolling deployment gradually replaces old instances with new ones.

```text
Old Old Old
   ↓
New Old Old
   ↓
New New Old
   ↓
New New New
```

This reduces downtime when the application supports safe rolling upgrades.

---

# 81. Canary Deployment

Release the new version to a small percentage of traffic first.

```text
95% → v1
5%  → v2
```

Monitor:

```text
Errors
Latency
Business metrics
```

Then increase traffic if healthy.

---

# 82. Blue-Green Deployment

Two environments:

```text
Blue → current
Green → new
```

Traffic switches from:

```text
Blue
```

to:

```text
Green
```

This can simplify rollback at the cost of additional infrastructure.

---

# 83. Database Migration

Use migration tools such as:

```text
Flyway
Liquibase
```

Avoid manually changing production schemas without versioned migration scripts.

---

# 84. Backward-Compatible Database Changes

A safe migration often follows:

```text
Add new column
↓
Deploy code that supports old + new
↓
Backfill data
↓
Switch reads/writes
↓
Remove old column later
```

This is useful for rolling deployments.

---

# 85. Distributed System Failure Model

Assume:

```text
Network can fail
Service can be slow
Message can be duplicated
Message can be delayed
Request can time out
Instance can restart
```

Design accordingly.

---

# 86. Why Timeouts Are Important

Without a timeout:

```text
Service A
   ↓
Service B hangs
   ↓
A waits
   ↓
Threads/requests accumulate
   ↓
A becomes unhealthy
```

Timeouts prevent indefinite resource consumption.

---

# 87. Why Retries Are Dangerous

If a dependency is already overloaded:

```text
Requests
   ↓
Retries
   ↓
More requests
   ↓
More overload
```

Use:

```text
Limited retries
Backoff
Jitter
Circuit breaker
```

---

# 88. Why Synchronous Chains Are Risky

Example:

```text
Gateway
 ↓
Order
 ↓
Payment
 ↓
Inventory
 ↓
Shipping
```

If each call takes 300ms:

```text
Latency accumulates
```

And if one dependency fails:

```text
Entire chain may fail
```

Use asynchronous processing where business requirements allow it.

---

# 89. Event-Driven Order Flow

Possible architecture:

```text
Client
  ↓
Order Service
  ↓
OrderCreated
  ↓
Kafka
  |
  +--> Inventory
  |
  +--> Payment
  |
  +--> Notification
```

Each service processes the event according to its responsibility.

---

# 90. Event Design

A useful event can contain:

```json
{
  "eventId": "evt-1001",
  "eventType": "OrderCreated",
  "aggregateId": "order-500",
  "occurredAt": "2026-08-20T10:00:00Z",
  "version": 1,
  "payload": {
    "orderId": 500
  }
}
```

Keep event contracts explicit and versionable.

---

# 91. Event Versioning

Events may evolve.

Example:

```text
OrderCreated v1
OrderCreated v2
```

Consumers should be able to handle compatible changes.

Avoid removing required fields without a migration strategy.

---

# 92. Event Schema Registry

For Kafka environments, a schema registry can help manage structured event schemas.

Possible formats include:

```text
Avro
Protobuf
JSON Schema
```

Benefits:

```text
Schema validation
Compatibility checks
Version management
```

---

# 93. Synchronous vs Asynchronous Order Processing

Synchronous:

```text
POST /orders
 ↓
Inventory
 ↓
Payment
 ↓
Response
```

Asynchronous:

```text
POST /orders
 ↓
Create PENDING order
 ↓
Publish event
 ↓
Return
 ↓
Workers process
```

The correct choice depends on whether the user needs an immediate final result.

---

# 94. Distributed Transaction Alternative

Instead of trying to create one global transaction:

```text
Order DB
Inventory DB
Payment DB
```

use:

```text
Local transactions
+
Events
+
Saga
+
Compensation
```

This is a common distributed-systems approach.

---

# 95. Compensation

A compensation reverses or offsets a previous successful operation.

Example:

```text
Inventory reserved
Payment failed
        ↓
Release inventory
```

Compensation is not always a literal database rollback.

It is a new business action that restores an acceptable state.

---

# 96. Saga Failure Scenario

```text
Order Created
     ↓
Inventory Reserved
     ↓
Payment Failed
```

Compensation:

```text
Release Inventory
     ↓
Cancel Order
```

The workflow must be designed so these actions are reliable and idempotent.

---

# 97. Service Ownership

A team should ideally own:

```text
Code
Database
Deployment
Monitoring
Operational responsibility
```

for its service.

This creates clearer accountability.

---

# 98. Microservice Size

A microservice should not be defined by:

```text
Exactly 100 classes
```

or:

```text
One database table
```

A better principle is:

> A service should represent a meaningful business capability with a clear ownership boundary.

---

# 99. Too Many Microservices

If an application has:

```text
100 tiny services
```

the system can become difficult to operate.

Problems:

```text
Network overhead
Deployment complexity
Monitoring complexity
Versioning
Debugging
Infrastructure cost
```

Prefer meaningful boundaries.

---

# 100. Shared Libraries

Shared libraries can reduce duplication but create coupling.

If:

```text
10 services
    ↓
Shared library v1
```

and changing the library requires coordinated upgrades, independent deployment becomes harder.

Keep shared libraries focused on stable cross-cutting concerns.

---

# 101. Service Contracts

A service contract defines:

```text
Endpoint
Request
Response
Errors
Authentication
Versioning
Timeout expectations
```

Treat contracts as first-class artifacts.

---

# 102. Contract Testing

Contract tests verify that consumers and providers agree on an API contract.

Useful for:

```text
Microservice APIs
Independent deployments
Breaking-change detection
```

---

# 103. Consumer-Driven Contracts

A consumer describes what it expects from a provider.

Conceptually:

```text
Order Service
   ↓
expects Payment API contract

Payment Service
   ↓
verifies contract
```

This can catch incompatible changes before deployment.

---

# 104. Observability Stack

A typical setup:

```text
Application
   |
+-- Logs → ELK/OpenSearch
|
+-- Metrics → Prometheus
|
+-- Traces → OpenTelemetry
|
+-- Dashboards → Grafana
```

Exact tooling varies by organization.

---

# 105. ELK

ELK commonly refers to:

```text
Elasticsearch
Logstash
Kibana
```

It can be used for centralized log collection, search, and visualization.

---

# 106. Structured Logging

Instead of:

```text
Order created
```

use structured fields:

```text
event=ORDER_CREATED
orderId=1001
customerId=200
service=order-service
traceId=abc123
```

This makes searching and aggregation easier.

---

# 107. Metrics vs Logs vs Traces

Metrics:

```text
What is happening?
```

Logs:

```text
What happened?
```

Traces:

```text
Where did the request spend time?
```

Use all three together.

---

# 108. Distributed Debugging

Example:

```text
Customer reports checkout failed
```

Use:

```text
Trace ID
 ↓
Gateway
 ↓
Order
 ↓
Payment
 ↓
Inventory
```

Then inspect each service's logs and metrics.

---

# 109. Microservice Security

Important areas:

```text
Authentication
Authorization
Service-to-service identity
TLS/mTLS
Secrets
API gateway
Rate limiting
Audit logs
Dependency scanning
```

Never assume internal network traffic is automatically trusted.

---

# 110. Microservice Database Security

Each service should receive only the database permissions it needs.

Example:

```text
Order DB user
→ SELECT/INSERT/UPDATE orders

Not:
→ DROP DATABASE
```

Use least privilege.

---

# 111. Microservices and Caching

Caching can reduce cross-service calls.

Instead of:

```text
Order
 ↓
Product Service
 ↓
Product DB
```

every time, some stable/read-heavy data can potentially be cached.

But consider:

```text
Staleness
Invalidation
Consistency
Memory
```

---

# 112. Cache Stampede in Microservices

If a popular cache entry expires:

```text
Many service instances
      ↓
Cache miss
      ↓
Database overload
```

Solutions:

```text
Jitter
Locking
Request coalescing
Background refresh
Pre-warming
```

---

# 113. Resilience4j

Resilience4j provides resilience patterns commonly used with Java applications.

Patterns include:

```text
Circuit breaker
Retry
Rate limiter
Bulkhead
Time limiter
```

Use only the patterns that solve actual reliability problems.

---

# 114. Circuit Breaker Example Concept

```java
@CircuitBreaker(
    name = "paymentService",
    fallbackMethod = "paymentFallback"
)
public PaymentResponse pay(...) {
    ...
}
```

The exact configuration should define:

```text
Failure threshold
Sliding window
Wait duration
Half-open behavior
```

---

# 115. Retry Example Concept

```java
@Retry(
    name = "paymentService"
)
public PaymentResponse pay(...) {
    ...
}
```

Retry only operations that are safe to retry.

---

# 116. Fallback

A fallback should provide a meaningful degraded behavior.

Bad:

```text
Return fake successful payment
```

Good:

```text
Mark payment as PENDING
```

if the business workflow supports it.

Never hide critical failures by pretending the operation succeeded.

---

# 117. Dead Letter Handling

A failed event can be moved to a dead-letter destination.

Then operators can:

```text
Inspect
Fix data/problem
Replay
Discard
```

depending on the business policy.

---

# 118. Replay

Events stored in Kafka can sometimes be replayed.

Useful for:

```text
Recovering from bugs
Rebuilding read models
Reprocessing failed events
```

Consumers should be designed carefully before replaying production events.

---

# 119. Event Replay Safety

Replay can cause:

```text
Duplicate side effects
Repeated emails
Repeated payments
Repeated inventory changes
```

Therefore event handlers should be:

```text
Idempotent
```

or have appropriate replay controls.

---

# 120. Microservices Interview Question

## What is the biggest challenge with microservices?

Strong answer:

> The biggest challenge is distributed-system complexity. Once services communicate over the network, we have to deal with timeouts, retries, partial failures, eventual consistency, observability, security, and versioned contracts. That's why I wouldn't choose microservices unless the business and scaling requirements justify that complexity.

---

# 121. Interview: Monolith vs Microservices

> A monolith is simpler to develop and operate because components share a process and usually a transaction boundary. Microservices allow independent deployment and scaling, but introduce network communication, distributed data, observability, and operational complexity. I choose based on business and organizational requirements rather than assuming microservices are always better.

---

# 122. Interview: How Do Microservices Communicate?

> They can communicate synchronously through REST, gRPC, or HTTP clients such as OpenFeign, or asynchronously through messaging systems such as Kafka. I use synchronous communication when an immediate response is required and asynchronous communication when decoupling and eventual processing are acceptable.

---

# 123. Interview: How Do You Handle Distributed Transactions?

> I generally avoid trying to create one global database transaction across services. Instead, I use local transactions combined with patterns such as Saga, Outbox, events, and compensating actions.

---

# 124. Interview: Explain Saga

> Saga breaks a distributed business transaction into a sequence of local transactions. If a later step fails, compensating actions undo or offset the earlier successful steps. It can be implemented through choreography or orchestration.

---

# 125. Interview: Explain Outbox Pattern

> The Outbox Pattern solves the problem of updating a database and publishing an event reliably. I store the business change and an outbox record in the same local transaction, then a separate publisher sends the event to the broker.

---

# 126. Interview: How Do You Prevent Duplicate Events?

> I make consumers idempotent. I typically use a unique event ID and record processed IDs or use a business-level idempotency key so that repeated delivery doesn't create duplicate business effects.

---

# 127. Interview: How Do You Handle Service Failure?

> I use timeouts first, then carefully chosen retries with exponential backoff and jitter for transient failures. For dependencies that remain unhealthy, circuit breakers and bulkheads can prevent cascading failures. For suitable workflows, asynchronous processing can also reduce coupling.

---

# 128. Interview: Why Is Idempotency Important?

> Network failures can cause clients or services to retry requests after the original operation may already have succeeded. Idempotency ensures that processing the same logical request multiple times doesn't create multiple business effects, which is especially important for payments and orders.

---

# 129. Interview: How Do You Design an Ecommerce Microservices System?

A practical high-level design:

```text
                    Client
                      |
                 API Gateway
                      |
        +-------------+-------------+
        |             |             |
      User         Product        Order
      Service      Service        Service
                                  |
                    +-------------+-------------+
                    |                           |
                Inventory                    Payment
                 Service                     Service
                    |
                    +-------------+
                                  |
                               Kafka
                                  |
                         Notification
                           Service
```

Each service owns its data.

---

# 130. Ecommerce Order Flow

A possible event-driven flow:

```text
POST /orders
      ↓
Order Service
      ↓
Create PENDING order
      ↓
OrderCreated
      ↓
Kafka
      |
      +------> Inventory
      |
      +------> Payment
      |
      +------> Notification
```

Then:

```text
InventoryReserved
PaymentCompleted
      ↓
Order Service
      ↓
CONFIRMED
```

If payment fails:

```text
PaymentFailed
      ↓
Order Service
      ↓
CANCELLED
```

The exact workflow should be designed around business requirements.

---

# 131. Ecommerce Failure Scenario

Payment succeeds but response is lost:

```text
Payment Service
    ↓
Payment successful
    ↓
Response lost
```

Order Service retries.

Without idempotency:

```text
Duplicate payment
```

With idempotency:

```text
Same payment result returned
```

This is a classic distributed-system problem.

---

# 132. Ecommerce Inventory Race Condition

Initial:

```text
Stock = 1
```

Two requests:

```text
User A → buy 1
User B → buy 1
```

A correct design must ensure:

```text
Only one succeeds
Stock never becomes negative
```

Possible techniques:

```text
Optimistic locking
Pessimistic locking
Atomic database update
Reservation model
```

---

# 133. Atomic Inventory Update

A database-level approach can be:

```sql
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = ?
  AND quantity >= 1;
```

Then check:

```text
Rows updated = 1
→ success

Rows updated = 0
→ insufficient stock
```

This can avoid some application-level race conditions.

---

# 134. Service Failure Example

Suppose:

```text
Recommendation Service
```

is unavailable.

Product API should ideally still provide:

```text
Product details
Price
Stock
```

and handle recommendations gracefully:

```text
Recommendations unavailable
```

instead of failing the entire request if recommendations are non-critical.

---

# 135. Microservices Deployment

A typical pipeline:

```text
Git
 ↓
Build
 ↓
Unit tests
 ↓
Integration tests
 ↓
Security scan
 ↓
Docker image
 ↓
Registry
 ↓
Kubernetes
 ↓
Rolling deployment
```

---

# 136. Microservices CI/CD

Each service should ideally be independently buildable and deployable.

Example:

```text
order-service pipeline
payment-service pipeline
inventory-service pipeline
```

This supports independent releases.

---

# 137. Service Health During Deployment

A new instance should not receive traffic until it is ready.

Conceptually:

```text
Start pod
 ↓
Readiness check
 ↓
Ready
 ↓
Receive traffic
```

If unhealthy:

```text
Do not send traffic
```

---

# 138. Graceful Shutdown

During deployment:

```text
Instance receives shutdown
 ↓
Stop accepting new traffic
 ↓
Finish/handle in-flight work
 ↓
Close resources
 ↓
Exit
```

This reduces dropped requests.

---

# 139. Backward-Compatible Deployment

Suppose:

```text
v1 instances
v2 instances
```

are running simultaneously.

The API and event contract should support both during the transition.

This is why backward compatibility matters in distributed deployments.

---

# 140. Schema Evolution

Database changes should be compatible with rolling deployments.

Example:

```text
Old code expects:
name

New code supports:
name + description
```

First add:

```text
description nullable
```

Then deploy code.

Later enforce stricter constraints after old versions are gone.

---

# 141. Microservices and Testing

Testing levels:

```text
Unit
 ↓
Service integration
 ↓
Contract tests
 ↓
End-to-end
```

Don't rely only on E2E tests because they are slower and harder to diagnose.

---

# 142. Contract Testing Example

```text
Order Service
expects:
POST /payments

Payment Service
must support:
request
response
error contract
```

Contract tests catch breaking changes before production.

---

# 143. Microservices Observability Checklist

```text
□ Structured logs
□ Correlation ID
□ Distributed tracing
□ Metrics
□ Health checks
□ Error tracking
□ Database metrics
□ Kafka lag
□ External dependency latency
□ Alerts
```

---

# 144. Microservices Security Checklist

```text
□ HTTPS
□ Authentication
□ Authorization
□ Service identity
□ Secret management
□ Least privilege
□ Input validation
□ Rate limiting
□ Audit logging
□ Dependency scanning
```

---

# 145. Microservices Reliability Checklist

```text
□ Timeouts
□ Retry policy
□ Exponential backoff
□ Jitter
□ Circuit breaker
□ Bulkhead
□ Idempotency
□ Dead-letter handling
□ Backpressure
□ Graceful degradation
```

---

# 146. Final Mental Model

```text
                 MICROSERVICES
                      |
       +--------------+--------------+
       |              |              |
     Services       Data           Network
       |              |              |
   Boundaries      Ownership      Failures
   Deployment      Events         Timeouts
   Scaling         Saga           Retries
       |              |              |
       +--------------+--------------+
                      |
              Observability
                      |
             Logs + Metrics + Traces
```

---

# 147. Final Interview Rule

> **Microservices are not simply small Spring Boot applications. They are distributed systems. The important part is defining clear business boundaries and then designing for network failures, independent data ownership, eventual consistency, observability, security, and independent deployment.**
