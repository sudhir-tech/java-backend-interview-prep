# System Design — File 10: Microservices Architecture & Communication

Microservices architecture breaks a large application into smaller services that can be developed, deployed and scaled independently.

A simple monolith:

```text
                 Application
        ┌──────────┼──────────┐
        ↓          ↓          ↓
      User       Order      Payment
      Logic      Logic       Logic
        └──────────┼──────────┘
                   ↓
                Database
```

A microservices architecture:

```text
              API Gateway
                   |
      ┌────────────┼────────────┐
      ↓            ↓            ↓
 User Service  Order Service  Payment Service
      |             |             |
    User DB       Order DB      Payment DB
```

The goal is not simply "many services."

The goal is:

```text
Independent ownership
Independent deployment
Independent scaling
Clear boundaries
Fault isolation
```

---

# 1. What Is a Microservice?

A microservice is a small, independently deployable service responsible for a focused business capability.

Examples:

```text
User Service
Product Service
Order Service
Payment Service
Inventory Service
Notification Service
```

A good service boundary usually follows a business capability rather than a technical layer.

---

# 2. Monolith vs Microservices

### Monolith

```text
One deployable application
One codebase
Often one database
```

Benefits:

```text
Simple deployment
Easy local development
Simple transactions
Easy debugging
Low network overhead
```

Challenges:

```text
Large codebase
Tightly coupled modules
Independent scaling is difficult
Large deployments
```

---

# 3. Microservices

```text
Multiple independently deployable services
```

Benefits:

```text
Independent scaling
Independent deployment
Team autonomy
Fault isolation
Technology flexibility
```

Costs:

```text
Network calls
Distributed transactions
Observability complexity
Deployment complexity
Data consistency challenges
Infrastructure overhead
```

---

# 4. Don't Start With Microservices Automatically

A common interview mistake:

> "I would create 20 microservices."

Better:

> "I'd start with a modular monolith unless the scale, team structure or independent deployment requirements justify splitting services."

Microservices solve organizational and scaling problems, but introduce distributed-system problems.

---

# 5. Modular Monolith

A useful intermediate architecture:

```text
Single Application
 ├── User Module
 ├── Product Module
 ├── Order Module
 ├── Payment Module
 └── Notification Module
```

The modules have clear boundaries even though they deploy together.

This can make future extraction easier.

---

# 6. Service Boundary

Good boundary:

```text
Order Service
```

owns:

```text
Order creation
Order state
Order lifecycle
```

Bad boundary:

```text
Validation Service
Logging Service
Calculation Service
```

when they don't represent meaningful business ownership.

---

# 7. Bounded Context

From Domain-Driven Design:

```text
Bounded Context
```

defines a boundary where:

```text
Domain concepts
Rules
Terminology
Models
```

have a specific meaning.

For example:

```text
Payment Service
```

may have:

```text
Payment
Transaction
Refund
```

while:

```text
Order Service
```

has:

```text
Order
OrderItem
OrderStatus
```

The models do not have to be identical.

---

# 8. Database per Service

A common microservices principle:

```text
Order Service
    ↓
Order DB

Payment Service
    ↓
Payment DB
```

The owning service controls its data.

---

# 9. Why Database-per-Service?

Benefits:

```text
Loose coupling
Independent schema evolution
Independent scaling
Clear ownership
Fault isolation
```

But it creates:

```text
Distributed transactions
Cross-service queries
Data duplication
Eventual consistency
```

---

# 10. Don't Share Tables Across Services

Avoid:

```text
Order Service ──┐
                ↓
             Same DB table
                ↑
                |
Payment Service ─┘
```

This creates tight coupling.

Prefer:

```text
Order Service → Order DB
Payment Service → Payment DB
```

and communicate through:

```text
APIs
Events
```

---

# 11. Service Communication

Two major approaches:

```text
Synchronous
Asynchronous
```

Synchronous:

```text
HTTP
REST
gRPC
```

Asynchronous:

```text
Kafka
RabbitMQ
SQS
Pub/Sub
```

---

# 12. Synchronous Communication

Example:

```text
Order Service
     ↓ HTTP
Payment Service
     ↓
Response
```

Advantages:

```text
Simple request/response
Immediate result
Easy to understand
```

Disadvantages:

```text
Coupling
Latency
Timeouts
Cascading failures
```

---

# 13. Asynchronous Communication

Example:

```text
Order Service
     ↓
Kafka
     ↓
Payment Service
```

Advantages:

```text
Loose coupling
Failure isolation
Load smoothing
Independent scaling
```

Disadvantages:

```text
Eventual consistency
More complex debugging
Retries
Duplicate events
Ordering concerns
```

---

# 14. REST

REST APIs commonly use:

```text
GET
POST
PUT
PATCH
DELETE
```

Example:

```text
GET /products/101
POST /orders
GET /orders/5001
```

Good for:

```text
Public APIs
CRUD operations
Simple service communication
Browser/mobile clients
```

---

# 15. gRPC

gRPC uses:

```text
HTTP/2
Protocol Buffers
```

It is often useful for:

```text
Internal service-to-service communication
High-performance RPC
Strong contracts
Streaming
```

---

# 16. REST vs gRPC

| Requirement | REST | gRPC |
|---|---|---|
| Browser-friendly | Excellent | Less direct |
| Human-readable | Usually | No |
| Internal RPC | Good | Excellent |
| Strong schema | Optional/OpenAPI | Strong protobuf contract |
| Streaming | Possible | Strong support |
| Simple public API | Excellent | Less common |

Choose based on the use case.

---

# 17. API Gateway

A gateway provides a common entry point.

```text
Client
  ↓
API Gateway
  ├── User Service
  ├── Product Service
  ├── Order Service
  └── Payment Service
```

---

# 18. API Gateway Responsibilities

Potential responsibilities:

```text
Routing
Authentication
Authorization
Rate limiting
TLS termination
Request validation
Response transformation
Observability
```

Don't put business logic into the gateway unnecessarily.

---

# 19. Gateway vs Load Balancer

### Load Balancer

Primarily distributes traffic:

```text
Requests
   ↓
Instance 1
Instance 2
Instance 3
```

### API Gateway

Understands API/service routing:

```text
/orders → Order Service
/products → Product Service
/users → User Service
```

They can coexist.

---

# 20. Service Discovery

When services scale dynamically:

```text
Order Service
```

may have:

```text
10 instances
```

Other services need to find them.

Service discovery provides:

```text
Service name
→ Available instances
```

Examples include:

```text
Kubernetes Service
Consul
Eureka
Cloud service discovery
```

---

# 21. Client-Side Discovery

The client discovers service instances and chooses one.

Conceptually:

```text
Order Service
     ↓
Service Registry
     ↓
Instances
     ↓
Choose instance
```

The client owns more routing responsibility.

---

# 22. Server-Side Discovery

The client calls:

```text
Service endpoint
```

and infrastructure routes the request.

Example:

```text
Order Service
      ↓
Load Balancer
      ↓
Payment Service instance
```

This is common in container orchestration platforms.

---

# 23. Service Registry

A registry can maintain:

```text
Service name
IP/address
Port
Health
Metadata
```

Services register and discover each other.

---

# 24. Health Checks

Two common concepts:

### Liveness

```text
Is the process alive?
```

### Readiness

```text
Can this instance receive traffic?
```

An application can be alive but not ready.

Example:

```text
Application running
Database unavailable
```

It may be:

```text
Liveness = healthy
Readiness = unhealthy
```

---

# 25. Timeouts

Every network call should have a sensible timeout.

Bad:

```text
Order Service
 ↓
Payment Service
 ↓
Wait forever
```

Good:

```text
Payment call
 ↓
Timeout
 ↓
Fallback/retry/error handling
```

Timeouts prevent resources from being held indefinitely.

---

# 26. Retry

Retries are useful for transient failures.

Example:

```text
Network timeout
```

Retry may succeed.

But don't retry every failure.

---

# 27. Retryable vs Non-Retryable

Potentially retryable:

```text
Temporary network error
503
Connection reset
```

Usually not useful to retry blindly:

```text
400 validation error
401 authentication failure
403 authorization failure
```

The exact retry policy depends on the operation.

---

# 28. Idempotency and Retries

Suppose:

```text
POST /payment
```

times out.

The client doesn't know whether payment succeeded.

Retrying could create:

```text
Two payments
```

Use:

```text
Idempotency key
```

for operations where duplicate execution is dangerous.

---

# 29. Circuit Breaker

Suppose:

```text
Order Service
 ↓
Payment Service
```

Payment is down.

Without a circuit breaker:

```text
Thousands of requests
 ↓
Payment
 ↓
Timeout
 ↓
Retry
 ↓
Timeout
```

This can overload the system.

---

# 30. Circuit Breaker States

Typical states:

```text
CLOSED
OPEN
HALF_OPEN
```

### Closed

Requests flow normally.

### Open

Requests fail fast without calling the dependency.

### Half-open

A limited number of test requests determine whether recovery occurred.

---

# 31. Circuit Breaker

Conceptually:

```text
Payment failures
      ↓
Threshold reached
      ↓
Circuit OPEN
      ↓
Fail fast
      ↓
Wait
      ↓
HALF_OPEN
      ↓
Success → CLOSED
```

---

# 32. Bulkhead Pattern

Bulkheads isolate resources.

Example:

```text
Payment calls → Thread Pool A
Search calls  → Thread Pool B
```

If payment becomes slow:

```text
Payment pool exhausted
```

search can continue using:

```text
Search pool
```

This prevents one dependency from consuming all resources.

---

# 33. Cascading Failure

Example:

```text
API
 ↓
Order
 ↓
Payment
 ↓
External Payment Provider
```

Payment slows down.

Then:

```text
Order threads blocked
 ↓
API threads blocked
 ↓
Traffic queue grows
 ↓
System becomes unhealthy
```

This is cascading failure.

Use:

```text
Timeouts
Circuit breakers
Bulkheads
Rate limits
Async processing
```

---

# 34. Rate Limiting

Protect services from excessive traffic.

Example:

```text
100 requests/min/user
```

Possible algorithms:

```text
Token Bucket
Leaky Bucket
Fixed Window
Sliding Window
```

---

# 35. Token Bucket

A bucket holds tokens.

Each request consumes a token.

```text
Bucket
[● ● ● ● ●]
   ↓
Request consumes token
```

Tokens refill over time.

It allows controlled bursts.

---

# 36. Distributed Rate Limiting

If there are multiple API instances:

```text
Instance 1
Instance 2
Instance 3
```

a local counter is insufficient if the limit applies globally.

Use a shared mechanism such as:

```text
Redis
```

or a gateway/load-balancer capability.

---

# 37. Distributed Tracing

A request can cross:

```text
Gateway
 ↓
Order
 ↓
Payment
 ↓
Inventory
```

A trace ID lets you follow the complete request.

Example:

```text
traceId = abc123
```

Every service logs:

```text
traceId
```

---

# 38. Correlation ID

A correlation ID can associate related logs across services.

Example:

```text
Request
 ↓
X-Correlation-ID: 12345
```

Services include:

```text
12345
```

in logs.

---

# 39. Observability

Three major pillars:

```text
Logs
Metrics
Traces
```

### Logs

What happened?

### Metrics

How much/how often?

### Traces

Where did the request spend time?

---

# 40. Metrics

Useful service metrics:

```text
Request rate
Error rate
Latency
CPU
Memory
Thread pool utilization
Connection pool utilization
Queue depth
Dependency latency
```

A useful starting framework is:

```text
RED
```

---

# 41. RED Method

For request-driven services:

```text
Rate
Errors
Duration
```

Example:

```text
Rate = 2K RPS
Errors = 1.2%
Duration = p95 180ms
```

---

# 42. USE Method

For infrastructure resources:

```text
Utilization
Saturation
Errors
```

Useful for:

```text
CPU
Disk
Network
```

---

# 43. Centralized Logging

With many services:

```text
Service 1 logs
Service 2 logs
Service 3 logs
```

centralize logs:

```text
Applications
   ↓
Log Collector
   ↓
Central Logging
```

Examples:

```text
ELK
OpenSearch
Cloud logging platforms
```

---

# 44. API Versioning

APIs evolve.

Example:

```text
/api/v1/orders
/api/v2/orders
```

or header/media-type based versioning.

Avoid breaking existing clients without a migration plan.

---

# 45. Backward Compatibility

Suppose old clients expect:

```json
{
  "name": "Laptop"
}
```

New API should not suddenly remove:

```text
name
```

unless clients are migrated.

Prefer additive changes when possible.

---

# 46. Synchronous Chain Problem

Suppose:

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

Latency roughly accumulates:

```text
L = L1 + L2 + L3 + L4
```

and failures can propagate through the chain.

Keep synchronous dependency chains short where possible.

---

# 47. Asynchronous Decoupling

Instead:

```text
Order
 ↓
Kafka
 ↓
Payment
```

Payment processing can happen asynchronously.

This improves:

```text
Failure isolation
Load handling
Independent scaling
```

but introduces eventual consistency.

---

# 48. Data Consistency Across Services

Suppose:

```text
Order DB
Payment DB
Inventory DB
```

A single transaction cannot normally cover all three.

Use:

```text
Saga
Events
Compensating actions
Outbox
```

depending on requirements.

---

# 49. Saga Example

```text
Create Order
    ↓
Reserve Inventory
    ↓
Process Payment
    ↓
Create Shipment
```

If payment fails:

```text
Release Inventory
    ↓
Cancel Order
```

---

# 50. Choreography

Services communicate through events:

```text
OrderCreated
    ↓
Inventory Service
    ↓
InventoryReserved
    ↓
Payment Service
    ↓
PaymentCompleted
```

No central coordinator.

---

# 51. Orchestration

Central workflow manager:

```text
Order Saga
    |
    +→ Reserve Inventory
    |
    +→ Process Payment
    |
    +→ Create Shipment
```

Useful for complex workflows where centralized state visibility is valuable.

---

# 52. Service Resilience

For every dependency ask:

```text
What if it is slow?
What if it is down?
What if it returns errors?
What if it returns duplicates?
What if network fails?
```

Then design:

```text
Timeout
Retry
Circuit breaker
Fallback
Idempotency
Bulkhead
```

where appropriate.

---

# 53. Fallback

If a dependency is unavailable, return a degraded result when possible.

Example:

```text
Recommendation Service unavailable
```

Instead of failing product page:

```text
Show product page
without recommendations
```

This is graceful degradation.

---

# 54. Graceful Degradation

Critical functionality:

```text
Checkout
Payment
Order creation
```

should have higher availability priority.

Non-critical functionality:

```text
Recommendations
Analytics
Personalization
```

may be degraded during failures.

---

# 55. Service Ownership

Each service should ideally have:

```text
Clear business responsibility
Clear API
Clear data ownership
Clear team ownership
```

This reduces architectural ambiguity.

---

# 56. Shared Libraries

Shared libraries can help with:

```text
Logging
Authentication utilities
Common API clients
Observability
```

But too much shared code can recreate coupling.

Avoid:

```text
One giant shared business library
```

used by every service.

---

# 57. Configuration

Don't hard-code:

```text
Database URLs
Passwords
API keys
Timeouts
```

Use configuration management and secret management.

Typical separation:

```text
Application configuration
+
Secrets
```

---

# 58. Secrets

Secrets include:

```text
Database passwords
JWT signing keys
API tokens
Cloud credentials
```

Store them in:

```text
Secret manager
Vault
Cloud secret service
Kubernetes Secret
```

with appropriate security controls.

---

# 59. Deployment

Each microservice should ideally be independently deployable.

Example:

```text
Order Service v2
```

can be deployed without redeploying:

```text
Payment Service
```

This is one of the major benefits of microservices.

---

# 60. Rolling Deployment

Gradually replace old instances:

```text
v1 v1 v1 v1
 ↓
v2 v1 v1 v1
 ↓
v2 v2 v1 v1
 ↓
v2 v2 v2 v1
 ↓
v2 v2 v2 v2
```

Requires backward compatibility.

---

# 61. Blue-Green Deployment

Maintain:

```text
Blue → current
Green → new
```

Switch traffic:

```text
Blue
 ↓
Green
```

Rollback can be fast.

---

# 62. Canary Deployment

Send a small percentage of traffic to the new version:

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

Then gradually increase traffic.

---

# 63. Health-Aware Routing

Don't send traffic to an unhealthy instance.

Use:

```text
Readiness checks
Health checks
Load balancer
Service discovery
```

---

# 64. Containerization

Microservices are commonly packaged as containers.

```text
Order Service
   ↓
Docker Image
   ↓
Container
```

Containers provide:

```text
Consistent runtime
Isolation
Portable deployment
```

---

# 65. Kubernetes

Kubernetes can orchestrate containers.

Conceptually:

```text
Kubernetes Cluster
 ├── Order Pods
 ├── Payment Pods
 ├── Product Pods
 └── Inventory Pods
```

It can handle:

```text
Scheduling
Scaling
Service discovery
Rolling deployments
Health checks
```

---

# 66. Horizontal Scaling

If Order Service receives more traffic:

```text
Order Pod × 2
```

can become:

```text
Order Pod × 10
```

The service should be stateless where practical.

---

# 67. Stateless Service

A stateless service does not depend on local instance memory for critical session state.

Instead:

```text
Shared DB
Redis
Token
```

can hold state.

This makes:

```text
Horizontal scaling
Failover
Load balancing
```

easier.

---

# 68. Stateful Service

State lives on the instance or local storage.

Example:

```text
Local session
Local uploaded file
```

This makes scaling more difficult.

Prefer externalized state for scalable APIs where appropriate.

---

# 69. Authentication

A gateway can validate:

```text
JWT
```

but services should still enforce authorization for resources they own.

Example:

```text
User can access order?
```

should ultimately be determined by the service responsible for the order.

---

# 70. JWT in Microservices

Typical flow:

```text
Client
 ↓
Login
 ↓
Auth Service
 ↓
JWT
 ↓
Client
 ↓
API Gateway
 ↓
Services
```

Services can validate token claims.

Be careful with:

```text
Token expiry
Revocation
Key rotation
Permissions
```

---

# 71. Service-to-Service Authentication

Don't assume internal traffic is automatically trusted.

Use:

```text
mTLS
Service identity
OAuth2/client credentials
Signed tokens
Network policies
```

as appropriate.

---

# 72. Zero Trust Concept

A service should not automatically trust another service just because it is inside the network.

Validate:

```text
Identity
Authorization
Network access
```

according to security requirements.

---

# 73. API Gateway Rate Limiting

Gateway can enforce:

```text
100 requests/sec per client
```

before requests reach services.

This protects downstream systems.

---

# 74. Circuit Breaker Location

Circuit breakers can exist:

```text
Client
API Gateway
Service client
Service mesh
```

The right layer depends on the architecture.

---

# 75. Service Mesh

A service mesh can provide infrastructure-level capabilities such as:

```text
Traffic management
mTLS
Retries
Observability
Service-to-service policy
```

Examples:

```text
Istio
Linkerd
```

A service mesh adds infrastructure complexity, so don't introduce it without a reason.

---

# 76. Distributed Transactions

Avoid:

```text
2-phase commit everywhere
```

when possible.

Prefer business workflows using:

```text
Saga
Outbox
Events
Compensation
```

for many microservice use cases.

---

# 77. Distributed Lock

Sometimes multiple service instances need coordination.

Redis or a database may be used for distributed locking.

But locks should be:

```text
Short-lived
Renewable where necessary
Safe under failure
```

and not used as a substitute for proper data modeling.

---

# 78. Service Discovery Failure

What if registry is unavailable?

You need:

```text
Cached service information
Health checks
Retry/fallback
Infrastructure-level discovery
```

Avoid making the registry a single point of failure.

---

# 79. API Gateway Failure

Gateway can become a bottleneck.

Use:

```text
Multiple gateway instances
Load balancer
Autoscaling
Health checks
```

---

# 80. Cascading Timeout

Suppose:

```text
Gateway timeout = 5s
Order timeout = 5s
Payment timeout = 5s
```

This can create bad behavior.

Timeouts should be designed as a hierarchy.

For example:

```text
Gateway
  5s

Order
  4s

Payment
  2s
```

with enough time for the outer layer to respond cleanly.

Exact values depend on workload.

---

# 81. Retry Multiplication

Suppose:

```text
Gateway retries 3x
Order retries 3x
Payment retries 3x
```

One original request can create many downstream attempts.

This can amplify load dramatically.

Centralize and carefully limit retry behavior.

---

# 82. Synchronous vs Asynchronous Decision

Ask:

```text
Does the caller need an immediate result?
```

If yes:

```text
Synchronous
```

If no:

```text
Asynchronous
```

Also consider:

```text
Consistency
Latency
Failure isolation
Workload
```

---

# 83. Example: Payment

User may need:

```text
Immediate payment authorization
```

So the initial payment authorization may be synchronous.

But:

```text
Email receipt
Analytics
Recommendation updates
```

can be asynchronous.

---

# 84. Example: Image Processing

Upload:

```text
POST /images
```

Return:

```text
202 Accepted
```

Then:

```text
Queue
 ↓
Image Worker
 ↓
Resize
 ↓
Store
```

The client doesn't need to wait for image processing.

---

# 85. HTTP Status for Async Work

A useful pattern:

```text
202 Accepted
```

means:

```text
Request accepted for processing
```

The API can return:

```text
jobId
```

Client can later query:

```text
GET /jobs/{jobId}
```

---

# 86. Service Communication Checklist

For every synchronous call define:

```text
Timeout
Retry policy
Circuit breaker
Authentication
Authorization
Idempotency
Observability
Fallback
```

For every asynchronous event define:

```text
Event schema
Partition key
Ordering
Delivery semantics
Retry
DLQ
Idempotency
Retention
```

---

# 87. Interview Question

### Why use microservices?

Answer:

> "I'd use microservices when independent deployment, scaling, team ownership or fault isolation justify the additional distributed-system complexity. I wouldn't split a system into microservices just because it's considered modern."

---

# 88. Interview Question

### What are the disadvantages of microservices?

Answer:

> "They introduce network failures, distributed transactions, eventual consistency, observability challenges, deployment complexity and additional infrastructure."

---

# 89. Interview Question

### How do services communicate?

Answer:

> "For immediate request-response operations I'd use REST or gRPC. For asynchronous workflows and events I'd use messaging such as Kafka or a queue. The choice depends on latency, consistency and coupling requirements."

---

# 90. Interview Question

### How do you handle service failure?

Answer:

> "I'd use timeouts, limited retries with backoff, circuit breakers and bulkheads where appropriate. For non-critical dependencies I'd also design graceful degradation."

---

# 91. Interview Question

### What is a circuit breaker?

Answer:

> "It prevents repeated calls to an unhealthy dependency. After failures cross a threshold, the circuit opens and requests fail fast. Later, a small number of requests can test whether the dependency has recovered."

---

# 92. Interview Question

### What is service discovery?

Answer:

> "It's the mechanism that allows services to find healthy instances of another service dynamically rather than hard-coding instance addresses."

---

# 93. Interview Question

### Why database-per-service?

Answer:

> "It gives each service ownership of its data and allows independent schema and scaling decisions. The trade-off is that cross-service queries and transactions become distributed problems."

---

# 94. Interview Question

### How do you handle distributed transactions?

Answer:

> "I first try to redesign the workflow so each service owns a local transaction. For multi-step business workflows, I'd consider Saga, events, outbox and compensating actions instead of relying on a global database transaction."

---

# 95. Interview Question

### What is graceful degradation?

Answer:

> "It means keeping critical functionality available even when a non-critical dependency fails. For example, a product page could remain available even if recommendations are temporarily unavailable."

---

# 96. Interview Question

### REST vs gRPC?

Answer:

> "REST is usually convenient for public and browser-facing APIs. gRPC is attractive for internal service-to-service communication when strong contracts, efficient serialization and streaming are useful."

---

# 97. Interview Question

### What is a stateless service?

Answer:

> "A stateless service doesn't rely on local instance memory for critical user state, which makes horizontal scaling and failover much easier."

---

# 98. Interview Question

### How would you design microservices for e-commerce?

Answer:

```text
API Gateway
     |
     +── User Service
     +── Product Service
     +── Order Service
     +── Payment Service
     +── Inventory Service
     +── Notification Service
             |
           Kafka
```

Each service owns its data.

Use:

```text
Redis → caching
Kafka → asynchronous events
MySQL → transactional data
Search → product search
Object Storage → media
```

where justified by requirements.

---

# 99. E-commerce Order Flow

```text
Client
 ↓
API Gateway
 ↓
Order Service
 ↓
MySQL
 ↓
Outbox
 ↓
Kafka
 ├── Inventory
 ├── Payment
 ├── Notification
 └── Analytics
```

Use:

```text
Idempotency
Retries
Timeouts
Circuit breakers
```

where appropriate.

---

# 100. Final Architecture

A production-oriented system may look like:

```text
                         Clients
                            |
                     Load Balancer
                            |
                      API Gateway
                            |
          +-----------------+----------------+
          |                 |                |
       User             Product           Order
       Service           Service          Service
          |                 |                |
       User DB          Product DB        Order DB
                            |                |
                         Redis             Outbox
                            |                |
                         Search             Kafka
                                             |
                       +---------------------+----------------+
                       |          |            |              |
                   Inventory   Payment     Notification    Analytics
                       |          |            |
                    Inv DB    Payment DB      Email
```

The important part is not the number of boxes.

The important part is:

```text
Clear ownership
Appropriate communication
Failure isolation
Correct consistency
Independent scaling
Good observability
```

---

# 101. Final Checklist

You should be able to explain:

```text
□ Microservices
□ Monolith
□ Modular monolith
□ Service boundaries
□ Bounded contexts
□ Database-per-service
□ Synchronous communication
□ Asynchronous communication
□ REST
□ gRPC
□ API Gateway
□ Load balancer
□ Service discovery
□ Health checks
□ Liveness
□ Readiness
□ Timeouts
□ Retries
□ Idempotency
□ Circuit breaker
□ Bulkhead
□ Cascading failures
□ Rate limiting
□ Distributed tracing
□ Correlation IDs
□ Observability
□ API versioning
□ Graceful degradation
□ Saga
□ Choreography
□ Orchestration
□ Stateless services
□ Service-to-service authentication
□ Zero trust
□ Service mesh
□ Rolling deployment
□ Blue-green deployment
□ Canary deployment
□ Kubernetes basics
□ Distributed transactions
```

---

# 102. One-Minute Interview Answer

### "How would you design a microservices architecture for an e-commerce application?"

> "I'd start by defining services around business capabilities such as users, products, orders, payments and inventory. Each service would own its data. I'd use synchronous REST or gRPC when the caller needs an immediate response and Kafka for asynchronous events such as order creation and notifications. An API Gateway would handle routing and cross-cutting concerns such as authentication and rate limiting. For resilience I'd use timeouts, controlled retries, circuit breakers and idempotency. I'd also add centralized logs, metrics and distributed tracing so failures can be diagnosed across services. I would avoid microservices if the scale and organizational requirements don't justify the added complexity."

---

# 103. Key Takeaway

> **Microservices are not about creating many small applications. They are about creating independently owned business capabilities with clear boundaries, controlled communication and independent scaling. The difficult part is managing the distributed-system problems that appear after the split.**

**File 10 complete.**
