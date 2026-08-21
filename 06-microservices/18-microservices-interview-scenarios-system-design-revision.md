# Microservices — Interview Scenarios & System Design Revision

This is the final Microservices interview-preparation file.

Use it as a revision guide after studying the previous files.

Core topics:

```text
Microservices Design
Service Boundaries
DDD Basics
API Gateway
Service Discovery
Communication
REST
Messaging
Kafka
Transactions
Saga
Outbox
Idempotency
Consistency
Caching
Database per Service
Observability
Security
Resilience
Docker
Kubernetes
CI/CD
Scaling
Failure Handling
System Design
Production Scenarios
Interview Questions
```

---

# 1. What Is a Microservice?

Interview answer:

> "A microservice is a small, independently deployable service that owns a specific business capability. It communicates with other services through well-defined APIs or events and can be developed, deployed and scaled independently."

Important:

```text
Small
Business-focused
Independently deployable
Loosely coupled
Owns its data
Communicates through contracts
```

---

# 2. Microservices vs Monolith

Monolith:

```text
+-----------------------------+
| Order | Payment | Inventory |
|        One Application       |
+-----------------------------+
            |
          One DB
```

Microservices:

```text
Order       Payment       Inventory
  |            |             |
 DB           DB            DB
```

Microservices provide independence but add distributed-system complexity.

---

# 3. When Should You Use Microservices?

Good reasons:

```text
Independent scaling
Independent deployment
Large teams
Clear business boundaries
Different availability requirements
Different technology requirements
```

Don't use microservices simply because:

```text
"Everyone uses microservices."
```

A modular monolith can be a better starting point.

---

# 4. Microservices Trade-Off

Benefits:

```text
Independent deployment
Independent scaling
Fault isolation
Team autonomy
Technology flexibility
```

Costs:

```text
Network failures
Distributed transactions
Observability complexity
Deployment complexity
Data consistency
Operational overhead
```

---

# 5. How Do You Identify Service Boundaries?

Start from business capabilities.

For e-commerce:

```text
User Management
Catalog
Cart
Order
Inventory
Payment
Notification
```

Avoid splitting purely by technical layers:

```text
Controller Service
Database Service
Utility Service
```

unless there is a real architectural reason.

---

# 6. Domain-Driven Design

DDD can help identify boundaries around business domains.

Example:

```text
Ordering
Inventory
Payments
Shipping
```

Each can become a bounded context.

---

# 7. Bounded Context

A bounded context defines the boundary within which a particular domain model and terminology apply.

Example:

```text
Customer
```

may mean different things in:

```text
Sales
Billing
Support
```

Don't force every service to share the exact same model.

---

# 8. Shared Database Problem

Bad:

```text
Order Service ─┐
Payment Service├→ Same DB
Inventory ─────┘
```

Problems:

```text
Tight coupling
Schema coupling
Deployment coupling
Cross-service queries
Unclear ownership
```

Prefer:

```text
Order → Order DB
Payment → Payment DB
Inventory → Inventory DB
```

---

# 9. Service Communication

Two major styles:

```text
Synchronous
Asynchronous
```

Synchronous:

```text
Order
 ↓
Inventory API
```

Asynchronous:

```text
OrderCreated
 ↓
Kafka
 ↓
Inventory
```

---

# 10. Synchronous Communication

Examples:

```text
REST
HTTP
gRPC
```

Advantages:

```text
Simple request/response
Immediate result
Easy to understand
```

Risks:

```text
Latency
Timeouts
Cascading failures
Dependency availability
```

---

# 11. Asynchronous Communication

Examples:

```text
Kafka
RabbitMQ
Cloud messaging
```

Advantages:

```text
Loose coupling
Better buffering
Independent processing
Event-driven architecture
```

Risks:

```text
Eventual consistency
Duplicate messages
Ordering issues
Debugging complexity
```

---

# 12. How Do You Choose REST vs Messaging?

Ask:

```text
Does caller need immediate response?
```

If yes:

```text
REST/gRPC
```

If no:

```text
Event/message
```

Example:

```text
Get product price
→ REST

Send order confirmation email
→ Event
```

---

# 13. API Gateway

External flow:

```text
Client
 ↓
API Gateway
 ↓
Services
```

Gateway can handle:

```text
Routing
Authentication
Rate limiting
TLS
API policies
Observability
```

Don't put core business logic there.

---

# 14. Service Discovery

Don't hardcode:

```text
10.20.30.40
```

Use:

```text
Service name
DNS
Service registry
```

Kubernetes commonly provides service discovery through Services and DNS.

---

# 15. Database per Service

Core rule:

> A service owns its data.

Example:

```text
Order Service
 ↓
Order DB

Payment Service
 ↓
Payment DB
```

Other services should use:

```text
API
or
Events
```

rather than directly querying the owner's database.

---

# 16. Distributed Transactions

Avoid trying to make:

```text
Order DB
+
Inventory DB
+
Payment DB
```

behave like one local transaction unless there is a very specific reason.

Use:

```text
Saga
+
Compensation
```

for many business workflows.

---

# 17. Saga

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

Failure:

```text
Payment fails
 ↓
Release Inventory
 ↓
Cancel Order
```

---

# 18. Saga Orchestration

Central coordinator:

```text
Saga Orchestrator
 ↓
Order
 ↓
Inventory
 ↓
Payment
```

Good when workflows are complex and centralized visibility is valuable.

---

# 19. Saga Choreography

Event-driven:

```text
OrderCreated
 ↓
InventoryReserved
 ↓
PaymentCompleted
 ↓
OrderConfirmed
```

Good for loosely coupled workflows, but event chains can become harder to understand.

---

# 20. Outbox Pattern

Problem:

```text
DB update succeeds
Event publish fails
```

Solution:

```text
Transaction
 ├→ Business data
 └→ Outbox event
       ↓
     COMMIT
       ↓
Outbox Publisher
       ↓
Kafka
```

---

# 21. Idempotency

Messages can be delivered more than once.

Consumer:

```text
PaymentCompleted
PaymentCompleted
```

should not:

```text
Charge customer twice
```

Use:

```text
Event ID
Unique constraint
Processed-event table
Business idempotency key
```

---

# 22. Eventual Consistency

Services may temporarily disagree:

```text
Order = CREATED
Payment = PENDING
Inventory = RESERVED
```

Eventually:

```text
Order = CONFIRMED
Payment = SUCCESS
Inventory = RESERVED
```

Design intermediate states explicitly.

---

# 23. Retry

Only retry:

```text
Transient
Safe
Retryable
```

Examples:

```text
Temporary network failure
HTTP 503
Temporary connection failure
```

Avoid blindly retrying:

```text
Validation errors
Authentication failures
Permanent business failures
```

---

# 24. Retry Storm

Bad:

```text
1000 requests
 ×
5 retries
```

Potentially:

```text
5000 requests
```

Use:

```text
Bounded retries
Exponential backoff
Jitter
Circuit breaker
```

---

# 25. Circuit Breaker

States:

```text
CLOSED
OPEN
HALF_OPEN
```

Purpose:

> Stop repeatedly calling an unhealthy dependency.

---

# 26. Bulkhead

Isolate resources:

```text
Payment → Pool A
Inventory → Pool B
```

Payment failure shouldn't consume all resources required by Inventory.

---

# 27. Timeout

Every remote call needs a bounded timeout.

Bad:

```text
wait forever
```

Better:

```text
Inventory timeout = 500ms
```

Use an overall request/deadline budget where possible.

---

# 28. Caching

Common pattern:

```text
Application
 ↓
Redis
 ↓
Database
```

Cache-aside:

```text
Check cache
 ↓
Hit → return
Miss
 ↓
DB
 ↓
Cache
```

---

# 29. Cache Invalidation

When data changes:

```text
Update DB
 ↓
Invalidate/update cache
```

Consider:

```text
TTL
Staleness tolerance
Invalidation events
```

---

# 30. Cache Stampede

Many requests miss the same key:

```text
10,000 requests
 ↓
Cache miss
 ↓
10,000 DB requests
```

Mitigation:

```text
Request coalescing
Locking
Jittered expiration
Prewarming
Stale-while-revalidate
```

---

# 31. CQRS

Separate:

```text
Write Model
Read Model
```

Useful when:

```text
Read workload is very high
Queries are complex
Read structure differs from write structure
```

Don't introduce it unnecessarily.

---

# 32. Event Sourcing

Store state changes:

```text
OrderCreated
PaymentStarted
PaymentCompleted
OrderConfirmed
```

rather than only the current state.

Benefits:

```text
History
Replay
Auditability
```

Costs:

```text
Complexity
Event schema evolution
Storage
Replay management
```

---

# 33. Observability

Three primary signals:

```text
Logs
Metrics
Traces
```

Mental model:

```text
Metrics → Detect
Traces → Locate
Logs → Explain
```

---

# 34. Metrics

Important API metrics:

```text
Request rate
Error rate
p95 latency
p99 latency
Timeouts
Active requests
```

---

# 35. Distributed Tracing

Example:

```text
Trace abc123

Gateway
 ↓
Order
 ↓
Inventory
 ↓
Payment
```

Find the slow span.

---

# 36. Correlation ID

Use a request identifier across logs:

```text
X-Correlation-ID: abc123
```

Trace context can also provide standardized trace identifiers.

---

# 37. Centralized Logging

Instead of searching each server:

```text
Services
 ↓
Log Collector
 ↓
Central Log Platform
 ↓
Kibana/Grafana/etc.
```

Use structured logging.

---

# 38. Security

Layers:

```text
Gateway
 ↓
Authentication
 ↓
Authorization
 ↓
Network Policy
 ↓
Service Identity
 ↓
Database Permissions
```

Use defense in depth.

---

# 39. JWT

JWT commonly contains:

```text
Header
Payload
Signature
```

Remember:

> A signed JWT is not automatically encrypted.

Don't put secrets into the payload.

---

# 40. OAuth2

OAuth2 is an authorization framework.

For service-to-service:

```text
Client Credentials Flow
```

can be used.

---

# 41. mTLS

Mutual TLS means:

```text
Client authenticates Server
Server authenticates Client
```

Useful for service identity and encrypted service-to-service traffic.

---

# 42. Zero Trust

Don't assume:

```text
Internal network = trusted
```

Instead:

```text
Authenticate
Authorize
Encrypt
Monitor
```

---

# 43. API Gateway vs Service Mesh

Gateway:

```text
External/client → system
```

Service mesh:

```text
Service → service
```

They can coexist.

---

# 44. Docker

Deployment flow:

```text
Source
 ↓
Build
 ↓
Docker Image
 ↓
Registry
 ↓
Deployment
```

Prefer immutable versioned images.

---

# 45. Kubernetes

Important objects:

```text
Pod
Deployment
Service
Ingress
ConfigMap
Secret
HPA
```

---

# 46. Pod

Smallest deployable Kubernetes unit.

Usually:

```text
1 Pod
1 application container
```

---

# 47. Deployment

Maintains desired replicas and manages rollout.

```text
Deployment
 ↓
ReplicaSet
 ↓
Pods
```

---

# 48. Service

Provides stable networking to Pods.

```text
Service
 ↓
Pod A
Pod B
Pod C
```

---

# 49. Readiness vs Liveness

```text
Readiness
→ Should receive traffic?

Liveness
→ Should be restarted?
```

Startup probes can protect slow-starting applications.

---

# 50. Horizontal Scaling

```text
3 Pods
 ↓
6 Pods
```

Useful for stateless services.

But scaling doesn't automatically fix:

```text
Database bottlenecks
External rate limits
Single-threaded bottlenecks
```

---

# 51. Deployment Strategies

Rolling:

```text
Gradual replacement
```

Blue-Green:

```text
Old environment
+
New environment
```

Canary:

```text
Small percentage → new version
```

---

# 52. CI/CD

Typical:

```text
Git Push
 ↓
Build
 ↓
Unit Tests
 ↓
Integration Tests
 ↓
Static Analysis
 ↓
Security Scan
 ↓
Docker Build
 ↓
Registry
 ↓
Deploy
 ↓
Smoke Test
 ↓
Monitor
```

---

# 53. Production Design Scenario

## Design an E-Commerce System

Requirements:

```text
Users
Products
Cart
Orders
Inventory
Payments
Notifications
```

Possible services:

```text
User Service
Product Service
Cart Service
Order Service
Inventory Service
Payment Service
Notification Service
```

---

# 54. E-Commerce Request Flow

```text
Client
 ↓
API Gateway
 ↓
Order Service
 ↓
Inventory Service
 ↓
Payment Service
```

Then:

```text
OrderCreated
 ↓
Kafka
 ↓
Notification Service
```

---

# 55. E-Commerce Database Design

```text
User Service
 → User DB

Product Service
 → Product DB

Order Service
 → Order DB

Inventory Service
 → Inventory DB

Payment Service
 → Payment DB
```

No direct cross-service DB joins.

---

# 56. Order Placement Workflow

Possible flow:

```text
1. Create Order = PENDING
2. Reserve Inventory
3. Process Payment
4. Confirm Order
5. Publish OrderConfirmed
6. Send Notification
```

---

# 57. Payment Failure

```text
Order = PENDING
 ↓
Payment FAILED
 ↓
Release Inventory
 ↓
Order = CANCELLED
```

Use Saga/compensation.

---

# 58. Payment Provider Timeout

Do not immediately charge again.

Use:

```text
Idempotency key
Provider status query
PENDING state
Reconciliation
```

---

# 59. Duplicate Payment Event

```text
PaymentCompleted
PaymentCompleted
```

Consumer:

```text
Check event ID
 ↓
Already processed?
 ↓
Yes → ignore
```

---

# 60. Inventory Concurrency

Two users buy the last product.

Possible solution:

```text
Optimistic locking
```

Example:

```text
stock = 1
version = 10
```

First update succeeds.

Second detects version conflict.

---

# 61. High Product Traffic

If millions of users read product details:

```text
CDN where appropriate
+
Redis
+
Read replicas
+
Indexes
```

Don't blindly scale the application while ignoring the database.

---

# 62. Checkout Is Slow

Investigation:

```text
Metrics
 ↓
p95 latency increased
 ↓
Trace
 ↓
Payment span slow
 ↓
Payment logs
 ↓
DB connection pool exhausted
```

Root cause is identified through multiple observability signals.

---

# 63. One Service Is Down

Suppose:

```text
Notification Service DOWN
```

Order creation should ideally continue if notification isn't part of the critical synchronous path:

```text
Order Created
 ↓
Event
 ↓
Notification later
```

---

# 64. Kafka Is Down

If an event is critical:

```text
Business transaction
+
Outbox
```

The event can remain in the outbox until publishing succeeds.

Don't lose the business state because the broker is temporarily unavailable.

---

# 65. Database Is Down

For a critical write:

```text
Fail safely
 ↓
Return appropriate error
 ↓
Retry only when safe
```

Don't return:

```text
SUCCESS
```

when the transaction wasn't committed.

---

# 66. External Payment API Is Down

Use:

```text
Timeout
Circuit breaker
Bounded retry
PENDING state
```

Depending on business rules, allow the customer to check status later.

---

# 67. Deployment Failure

Use:

```text
Canary
 ↓
Observe
 ↓
Detect regression
 ↓
Stop rollout
 ↓
Rollback
```

---

# 68. Database Migration Failure

Prefer:

```text
Backward-compatible migration
```

Use:

```text
Expand
 ↓
Compatible application
 ↓
Backfill
 ↓
Switch
 ↓
Contract
```

---

# 69. Service Security

For internal communication:

```text
mTLS
or
OAuth2
```

and:

```text
Authorization
+
Network Policy
```

---

# 70. Microservices Failure Model

Always assume:

```text
Network can fail
Service can be slow
Message can duplicate
Message can arrive late
Database can fail
Deployment can fail
Configuration can be wrong
```

This mindset is essential.

---

# 71. Design for Failure

Instead of:

```text
Everything always works
```

design:

```text
If dependency fails
→ timeout

If repeated failures
→ circuit opens

If message duplicates
→ idempotency

If event publishing fails
→ outbox

If instance dies
→ replacement

If deployment fails
→ rollback
```

---

# 72. Most Important Patterns

Memorize these:

```text
Database per Service
API Gateway
Service Discovery
Saga
Outbox
Idempotency
Circuit Breaker
Retry + Backoff + Jitter
Bulkhead
Caching
CQRS
Observability
mTLS
CI/CD
Docker
Kubernetes
Canary
```

---

# 73. Top 20 Interview Questions

## 1. Why microservices?

> "They allow independently deployable and scalable business capabilities, but introduce distributed-system complexity."

## 2. How do you define service boundaries?

> "I start from business capabilities and bounded contexts rather than technical layers."

## 3. How do services communicate?

> "Synchronously through APIs when an immediate response is required, and asynchronously through events/messages when decoupling is more valuable."

## 4. How do you manage distributed transactions?

> "Use local transactions and a Saga with compensating actions for workflows that span services."

## 5. How do you avoid DB/event dual writes?

> "Use the Outbox Pattern."

## 6. How do you handle duplicate messages?

> "Make consumers idempotent using event IDs, unique constraints or processed-event records."

## 7. How do you handle service failure?

> "Use timeouts, bounded retries, circuit breakers, bulkheads and graceful degradation where appropriate."

## 8. How do you monitor microservices?

> "Centralized logs, metrics and distributed tracing."

## 9. How do you secure service-to-service communication?

> "Use workload identity with mechanisms such as mTLS or OAuth2, plus authorization and network policies."

## 10. Why API Gateway?

> "To provide a controlled entry point for routing, authentication, rate limiting and other cross-cutting policies."

## 11. What is service discovery?

> "A mechanism that lets services locate healthy instances without hardcoding addresses."

## 12. What is eventual consistency?

> "Services may temporarily have different states but converge after asynchronous updates are processed."

## 13. What is CQRS?

> "Separating the write model from the read model when their requirements differ."

## 14. What is circuit breaker?

> "It prevents repeated calls to an unhealthy dependency and fails fast while allowing controlled recovery checks."

## 15. What is bulkhead?

> "It isolates resources so failure in one dependency doesn't consume resources needed by other operations."

## 16. What is mTLS?

> "Mutual TLS authenticates both sides of a connection and encrypts communication."

## 17. What is Kubernetes Service?

> "It provides stable networking and service discovery for a set of Pods."

## 18. Rolling vs Canary?

> "Rolling gradually replaces instances. Canary sends a small percentage of traffic to the new version before increasing it."

## 19. How do you scale a service?

> "First identify the bottleneck, then use horizontal scaling, caching, database optimization, replicas or other targeted solutions."

## 20. What happens if Kafka is down?

> "For critical events, I'd persist the event using an Outbox in the same local transaction and publish it when Kafka recovers."

---

# 74. Rapid-Fire Revision

### REST or Kafka?

```text
Immediate response → REST
Async event → Kafka
```

### Saga or 2PC?

```text
Usually Saga in microservices
```

### Shared DB or DB per service?

```text
Prefer DB ownership per service
```

### Retry or Circuit Breaker?

```text
Retry transient failures
Circuit breaker repeated dependency failures
```

### JWT or OAuth2?

```text
JWT → token format
OAuth2 → authorization framework
```

### Gateway or Service Mesh?

```text
Gateway → external traffic
Mesh → service-to-service traffic
```

### Logs or Metrics?

```text
Logs → detailed events
Metrics → trends/alerts
```

### Metrics or Traces?

```text
Metrics → detect
Traces → locate
```

### Readiness or Liveness?

```text
Readiness → traffic
Liveness → restart
```

### Rolling or Canary?

```text
Rolling → gradual replacement
Canary → gradual traffic exposure
```

---

# 75. System Design Framework

When given a microservices system-design problem, follow this order.

```text
1. Requirements
2. Business capabilities
3. Service boundaries
4. APIs/events
5. Data ownership
6. Transactions
7. Consistency
8. Caching
9. Scaling
10. Resilience
11. Security
12. Observability
13. Deployment
14. Failure scenarios
```

---

# 76. Step 1 — Requirements

Ask:

```text
Who are the users?
What operations exist?
What is the expected traffic?
What is latency requirement?
What availability is required?
What data must be strongly consistent?
```

---

# 77. Step 2 — Business Capabilities

Example:

```text
Catalog
Orders
Payments
Inventory
Users
Notifications
```

---

# 78. Step 3 — APIs and Events

Example APIs:

```http
GET /products/{id}
POST /orders
GET /orders/{id}
POST /payments
```

Events:

```text
OrderCreated
PaymentCompleted
InventoryReserved
OrderConfirmed
```

---

# 79. Step 4 — Data Ownership

```text
Order → Order DB
Payment → Payment DB
Inventory → Inventory DB
```

---

# 80. Step 5 — Consistency

Ask:

```text
Which operations require immediate consistency?
Which can be eventually consistent?
```

Example:

```text
Payment authorization
→ Stronger consistency requirements

Email notification
→ Eventual consistency acceptable
```

---

# 81. Step 6 — Scaling

Identify hot paths.

Example:

```text
Product reads
→ Cache

Order writes
→ Scale Order Service

Payment
→ Protect external provider limits
```

---

# 82. Step 7 — Resilience

For every remote dependency:

```text
Timeout?
Retry?
Circuit breaker?
Fallback?
Bulkhead?
```

Don't apply all patterns blindly.

---

# 83. Step 8 — Security

Define:

```text
Authentication
Authorization
Service identity
Secrets
Encryption
Network boundaries
Audit
```

---

# 84. Step 9 — Observability

Define:

```text
Logs
Metrics
Traces
Alerts
Dashboards
Business metrics
```

---

# 85. Step 10 — Deployment

Define:

```text
Docker
Registry
Kubernetes
CI/CD
Rolling/Canary
Health checks
Rollback
```

---

# 86. Step 11 — Failure Scenarios

Always discuss:

```text
Service unavailable
Database unavailable
Broker unavailable
Network timeout
Duplicate message
Slow dependency
Bad deployment
Schema mismatch
```

---

# 87. Strong System Design Answer Structure

Use this interview flow:

> "First, I'd identify the business boundaries. Then I'd define service ownership and APIs/events. Each service would own its data and use local transactions. For cross-service workflows I'd use Saga and Outbox where appropriate. I'd add caching and horizontal scaling based on actual bottlenecks. For resilience I'd use timeouts, targeted retries, circuit breakers and bulkheads. Security would include authentication, authorization and service identity. Finally, I'd add centralized logs, metrics and distributed tracing and deploy through automated CI/CD with safe rollout and rollback."

---

# 88. Common Mistakes

Avoid saying:

```text
"Every service must have its own physical database server."
```

Better:

```text
"Each service should own its data and schema boundary."
```

---

# 89. Common Mistake

Don't say:

```text
"Use Kafka for everything."
```

Instead:

```text
Use messaging where asynchronous decoupling provides value.
```

---

# 90. Common Mistake

Don't say:

```text
"Retry every failure."
```

Say:

```text
Retry only safe transient failures.
```

---

# 91. Common Mistake

Don't say:

```text
"JWT provides encryption."
```

Correct:

```text
JWT can be signed.
A signed JWT is not automatically encrypted.
```

---

# 92. Common Mistake

Don't say:

```text
"Kubernetes fixes application failures."
```

Correct:

```text
Kubernetes can restart/reschedule workloads, but application bugs still need fixing.
```

---

# 93. Common Mistake

Don't say:

```text
"API Gateway handles all security."
```

Better:

```text
Gateway provides an important security layer, while services still enforce appropriate authorization.
```

---

# 94. Common Mistake

Don't say:

```text
"Microservices always improve performance."
```

Correct:

```text
Microservices improve independent scaling, but network calls and distributed coordination can add latency.
```

---

# 95. Common Mistake

Don't say:

```text
"Microservices have no transactions."
```

Correct:

```text
Each service can have normal local database transactions. The challenge is coordinating workflows across service boundaries.
```

---

# 96. Scenario: Design a Payment System

Services:

```text
Payment API
Payment Processor
Ledger
Notification
```

Important requirements:

```text
Idempotency
Audit
Strong consistency for local payment state
External provider reconciliation
Security
Observability
```

---

# 97. Payment Idempotency

Client sends:

```text
Idempotency-Key: abc123
```

If the request is repeated:

```text
abc123
```

return the existing result rather than creating a second payment.

---

# 98. Payment Reconciliation

Possible state:

```text
Local = PENDING
Provider = SUCCESS
```

A reconciliation process can:

```text
Query provider
 ↓
Confirm result
 ↓
Update local state
```

---

# 99. Scenario: Inventory Reservation

Requirement:

```text
Stock = 1
Two users purchase simultaneously
```

Use appropriate concurrency control:

```text
Optimistic locking
or
Pessimistic locking
```

depending on contention and transaction design.

---

# 100. Scenario: Notification

Notification is usually a good asynchronous use case:

```text
OrderConfirmed
 ↓
Kafka
 ↓
Notification Service
```

Order creation doesn't need to wait for email delivery.

---

# 101. Scenario: Search

For complex search:

```text
Product DB
 ↓
Search index
```

Events can update the search index asynchronously.

This creates:

```text
Eventual consistency
```

but can provide much faster search.

---

# 102. Scenario: Reporting

Don't run heavy analytics against critical transactional databases.

Prefer:

```text
Events/CDC
 ↓
Analytics platform
 ↓
Reports
```

---

# 103. Scenario: Multi-Region

Consider:

```text
Latency
Data residency
Replication
Failover
Consistency
Conflict resolution
Traffic routing
```

Don't assume multi-region automatically improves every system.

---

# 104. Scenario: Disaster Recovery

Think about:

```text
Backups
Replication
RPO
RTO
Failover
Restore testing
```

---

# 105. RPO

Recovery Point Objective:

> How much data loss is acceptable after a disaster?

Example:

```text
RPO = 5 minutes
```

means the organization may tolerate losing up to approximately five minutes of recent data, depending on implementation.

---

# 106. RTO

Recovery Time Objective:

> How quickly must the service be restored?

Example:

```text
RTO = 30 minutes
```

---

# 107. RPO vs RTO

```text
RPO
→ Data loss tolerance

RTO
→ Recovery time tolerance
```

---

# 108. Final E-Commerce Architecture

```text
                         Clients
                            |
                            ↓
                       API Gateway
                            |
       +--------------------+--------------------+
       |         |          |         |          |
       ↓         ↓          ↓         ↓          ↓
     User     Product      Cart      Order    Payment
     Service  Service      Service   Service  Service
       |         |          |         |          |
       ↓         ↓          ↓         ↓          ↓
      DB        DB         DB        DB         DB
                                      |
                                      ↓
                              Inventory Service
                                      |
                                      ↓
                                  Inventory DB

Order Service
      |
      ↓
   Outbox
      |
      ↓
    Kafka
      |
      +──────────────→ Notification
      |
      +──────────────→ Analytics
      |
      +──────────────→ Search Projection
```

Cross-cutting:

```text
Observability
Security
Configuration
Secrets
CI/CD
Kubernetes
```

---

# 109. Complete Microservices Mental Model

```text
BUSINESS
   ↓
Service Boundaries
   ↓
APIs + Events
   ↓
Data Ownership
   ↓
Local Transactions
   ↓
Saga / Outbox
   ↓
Eventual Consistency
   ↓
Caching + Scaling
   ↓
Resilience
   ↓
Security
   ↓
Observability
   ↓
Deployment
   ↓
Failure Recovery
```

---

# 110. Final 30-Second Interview Answer

If the interviewer says:

> "Explain how you would build a production-ready microservices system."

Say:

> "I'd start by identifying clear business boundaries and create independently deployable services around those capabilities. Each service would own its data and expose APIs or events rather than sharing databases. I'd use local transactions and Saga for workflows spanning services, with Outbox and idempotent consumers for reliable event processing. I'd add caching and horizontal scaling based on workload, and use timeouts, targeted retries, circuit breakers and bulkheads for resilience. Security would include authentication, authorization, service identity and secret management. Finally, I'd use centralized logs, metrics and distributed tracing, package services as immutable Docker images, deploy through CI/CD and Kubernetes, and use safe rollout and rollback strategies."

---

# 111. Final Revision Checklist

```text
□ Microservice definition
□ Monolith vs microservices
□ Service boundaries
□ Business capabilities
□ DDD
□ Bounded contexts
□ API Gateway
□ Service discovery
□ REST
□ Messaging
□ Kafka
□ Database per service
□ Local transactions
□ Distributed transactions
□ Saga
□ Outbox
□ Idempotency
□ Eventual consistency
□ CQRS
□ Event sourcing
□ Caching
□ Retry
□ Timeout
□ Circuit breaker
□ Bulkhead
□ Rate limiting
□ Logs
□ Metrics
□ Traces
□ Correlation IDs
□ OpenTelemetry
□ Security
□ JWT
□ OAuth2
□ mTLS
□ Zero Trust
□ Secrets
□ Docker
□ Kubernetes
□ Pods
□ Deployments
□ Services
□ Ingress
□ ConfigMaps
□ Secrets
□ HPA
□ CI/CD
□ Rolling deployments
□ Blue-green
□ Canary
□ GitOps
□ Feature flags
□ Disaster recovery
□ RPO
□ RTO
□ Production failure scenarios
□ System design
```

---

# 112. What You Should Be Able to Explain Now

After completing the Microservices section, you should be comfortable answering:

```text
Why microservices?
How do you identify boundaries?
How do services communicate?
How do you manage distributed transactions?
What is Saga?
What is Outbox?
How do you handle duplicate events?
How do you handle service failures?
How do you design retries?
What is circuit breaker?
What is bulkhead?
How do you monitor services?
How does distributed tracing work?
How do you secure service communication?
What is mTLS?
What is OAuth2?
How does Kubernetes deploy services?
How do you safely release a new version?
How do you scale services?
How do you design an e-commerce system?
How do you handle payment failures?
How do you handle database failures?
How do you handle Kafka failures?
How do you handle duplicate payments?
How do you recover from a bad deployment?
```

---

# 113. Final Microservices Rule

When designing any distributed system, ask:

```text
What happens when this dependency is slow?
What happens when it is down?
What happens when the message is duplicated?
What happens when the network fails?
What happens when the database commits but the event doesn't publish?
What happens when two users update the same data?
What happens when the new deployment is broken?
What happens when configuration is wrong?
What happens when credentials expire?
```

If you can answer those questions clearly, you are thinking like a production backend engineer rather than only designing the happy path.

---

# 114. Microservices Section Complete

```text
01 → Fundamentals
02 → Architecture & Service Boundaries
03 → REST & API Communication
04 → Messaging & Kafka
05 → Service Discovery & Gateway
06 → Resilience Patterns
07 → Security
08 → Data Management
09 → Distributed Transactions
10 → Saga & Outbox
11 → Caching & Performance
12 → Event-Driven Architecture
13 → Inter-Service Communication
14 → Database & Distributed Data
15 → Observability
16 → Docker, Kubernetes & CI/CD
17 → Advanced Config, Security & Service Mesh
18 → Interview Scenarios & System Design
```

## Final takeaway

> **Microservices are not mainly about splitting one application into many applications. They are about defining clear business boundaries, owning data properly, communicating through contracts, and designing every interaction for failure, observability, security and independent deployment.**
