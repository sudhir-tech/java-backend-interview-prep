# Spring Boot — System Design

This file covers practical system design concepts for Java/Spring Boot backend interviews.

The goal is not to memorize one architecture. The goal is to learn how to reason about requirements, scale, reliability, data, APIs, caching, messaging, and tradeoffs.

---

# 1. What Is System Design?

System design is the process of deciding how software components work together to satisfy functional and non-functional requirements.

Typical concerns:

```text
Requirements
APIs
Architecture
Database
Caching
Messaging
Scalability
Reliability
Security
Observability
Deployment
Cost
```

---

# 2. Functional vs Non-Functional Requirements

Functional:

```text
User can register
User can login
User can add product to cart
User can place order
```

Non-functional:

```text
10,000 requests/sec
99.9% availability
p95 latency < 300 ms
Secure authentication
Horizontal scalability
```

Always clarify both before designing.

---

# 3. Start With Requirements

A good interview approach:

```text
1. Clarify requirements
2. Estimate scale
3. Define APIs
4. Design high-level architecture
5. Design data model
6. Discuss scaling
7. Discuss failure handling
8. Discuss security
9. Discuss observability
10. Explain tradeoffs
```

---

# 4. Clarifying Questions

For an ecommerce system, ask:

```text
How many users?
How many products?
How many orders/day?
Read/write ratio?
Expected traffic peak?
Strong consistency required?
Payment synchronous or asynchronous?
Search requirements?
Availability target?
Data retention?
```

Good system design starts with questions, not boxes.

---

# 5. Capacity Estimation

Suppose:

```text
1,000,000 users
100,000 orders/day
```

Average orders/sec:

```text
100,000 / 86,400
≈ 1.16 orders/sec
```

But average traffic is not peak traffic.

If peak is 10x:

```text
≈ 12 orders/sec
```

This helps determine infrastructure needs.

---

# 6. Requests Per Second

Suppose:

```text
10 million API requests/day
```

Approximate average RPS:

```text
10,000,000 / 86,400
≈ 116 RPS
```

Then apply a peak multiplier based on traffic patterns.

---

# 7. Storage Estimation

Suppose:

```text
1 million orders
2 KB/order
```

Raw order data:

```text
1,000,000 × 2 KB
≈ 2 GB
```

Then account for:

```text
Indexes
Replication
Backups
Logs
Growth
Metadata
```

---

# 8. Bandwidth Estimation

If:

```text
1,000 requests/sec
10 KB average response
```

Outbound bandwidth:

```text
1,000 × 10 KB
= 10 MB/sec
```

Approximate daily transfer:

```text
10 MB × 86,400
≈ 864 GB/day
```

These estimates help guide architecture.

---

# 9. Availability

Availability describes how often the system is usable.

Examples:

```text
99%
99.9%
99.99%
99.999%
```

More availability usually increases:

```text
Infrastructure cost
Operational complexity
Redundancy requirements
```

---

# 10. Latency

Latency is the time required to complete an operation.

Important metrics:

```text
Average
p50
p95
p99
```

For user-facing APIs, tail latency such as p95/p99 is often more useful than average latency.

---

# 11. Throughput

Throughput represents the amount of work processed over time.

Examples:

```text
5,000 requests/sec
10,000 messages/sec
2,000 orders/minute
```

A system can have high throughput but poor latency, so measure both.

---

# 12. Scalability

Scalability is the ability to handle increased workload.

Two common approaches:

```text
Vertical scaling
Horizontal scaling
```

---

# 13. Vertical Scaling

Increase resources:

```text
4 CPU / 8 GB
       ↓
8 CPU / 16 GB
```

Simple but has physical and cost limits.

---

# 14. Horizontal Scaling

Add instances:

```text
1 instance
   ↓
3 instances
   ↓
10 instances
```

Works especially well for stateless application services.

---

# 15. Stateless Application

A stateless service does not depend on local instance memory for persistent user session state.

Architecture:

```text
Load Balancer
   |
+--+--+--+
|  |  |
A  B  C
```

Any instance can handle the request.

---

# 16. Load Balancer

A load balancer distributes traffic.

```text
Client
  ↓
Load Balancer
  |
+---+---+---+
|   |   |   |
A   B   C
```

Benefits:

```text
Traffic distribution
High availability
Horizontal scaling
Health checks
```

---

# 17. Reverse Proxy

A reverse proxy sits between clients and backend services.

Responsibilities can include:

```text
TLS termination
Routing
Compression
Caching
Rate limiting
Security filtering
```

Examples include:

```text
Nginx
HAProxy
Cloud load balancers
Ingress controllers
```

---

# 18. API Gateway

A gateway provides a common entry point.

```text
Client
  ↓
API Gateway
  |
+---+---+---+
|   |   |   |
User Order Product
```

Typical responsibilities:

```text
Routing
Authentication integration
Rate limiting
Request filtering
Observability
```

Don't put core business logic in the gateway.

---

# 19. Database Selection

Choose a database based on requirements.

Relational databases:

```text
PostgreSQL
MySQL
Oracle
```

Useful for:

```text
Transactions
Relationships
Strong consistency
Structured data
Complex queries
```

NoSQL databases can be useful for specific access patterns and scalability requirements.

---

# 20. Why Use a Relational Database?

For orders and payments:

```text
Order
OrderItem
Payment
Customer
```

relationships and transactions matter.

A relational database is often a strong default.

---

# 21. Database Indexing

Indexes improve lookup performance.

Example:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

But indexes have costs:

```text
Storage
Insert/update overhead
Maintenance
```

Index based on real query patterns.

---

# 22. Database Connection Pool

Spring Boot commonly uses HikariCP.

Architecture:

```text
Application
    ↓
HikariCP
    ↓
Database
```

Connections are reused instead of created for every request.

---

# 23. Connection Pool Sizing

Suppose:

```text
10 application instances
20 DB connections each
```

Potential DB connections:

```text
10 × 20 = 200
```

Increasing every application pool independently can overload the database.

Database capacity must be considered globally.

---

# 24. Read Replicas

For read-heavy workloads:

```text
             +--> Read Replica 1
             |
Application --+--> Read Replica 2
             |
             +--> Primary
                    ↑
                  Writes
```

Benefits:

```text
Read scaling
Reduced primary load
```

Tradeoff:

```text
Replication lag
```

---

# 25. Database Sharding

Sharding distributes data across multiple databases.

Example:

```text
Shard 1 → Users 1-1M
Shard 2 → Users 1M-2M
Shard 3 → Users 2M-3M
```

Useful for very large datasets.

Challenges:

```text
Cross-shard queries
Transactions
Rebalancing
Hotspots
Operational complexity
```

Do not introduce sharding before simpler scaling options are exhausted.

---

# 26. Caching

Caching stores frequently accessed data closer to the application.

```text
Request
  ↓
Cache
  |
  +-- hit → response
  |
  +-- miss
       ↓
    Database
       ↓
     Cache
```

---

# 27. Redis

Redis is commonly used as a distributed cache.

Example:

```text
Spring Boot
     |
     v
   Redis
     |
     v
 MySQL
```

Good candidates:

```text
Product details
Reference data
Frequently accessed configuration
Short-lived computed results
```

---

# 28. Cache-Aside

Flow:

```text
Read cache
   |
   +-- hit → return
   |
   +-- miss
          ↓
       database
          ↓
        cache
          ↓
        return
```

This is one of the most common caching patterns.

---

# 29. Cache Invalidation

When data changes:

```text
Database update
      ↓
Invalidate/update cache
```

Caching creates consistency questions, so choose TTL and invalidation rules based on business requirements.

---

# 30. Cache Stampede

Popular key expires:

```text
10,000 requests
      ↓
Cache miss
      ↓
10,000 DB requests
```

Possible protections:

```text
Jittered TTL
Request coalescing
Distributed lock
Background refresh
Cache warming
```

---

# 31. CDN

A CDN caches content closer to users.

Useful for:

```text
Images
CSS
JavaScript
Videos
Static assets
Cacheable public content
```

Architecture:

```text
User
 ↓
CDN
 ↓ miss
Origin
```

---

# 32. Message Queue

Messaging decouples producers and consumers.

```text
Order Service
      ↓
    Queue
      ↓
Notification Worker
```

Benefits:

```text
Asynchronous processing
Traffic smoothing
Failure isolation
Decoupling
```

---

# 33. Kafka vs Queue

Kafka is strong for:

```text
Event streaming
Retention
Replay
High throughput
Multiple consumer groups
```

Traditional queues are often strong for:

```text
Work distribution
Task processing
Routing
```

Choose based on the workflow.

---

# 34. Asynchronous Processing

Example:

```text
Place Order
    ↓
Create order
    ↓
Return response
    ↓
Publish event
    ↓
Send email
```

Email does not need to delay the order response.

---

# 35. When Not to Use Async

Don't make everything asynchronous.

If the client needs an immediate answer:

```text
GET product
POST login
Check payment status
```

synchronous processing may be appropriate.

---

# 36. Distributed Transactions

Avoid trying to create one global ACID transaction across many services.

Instead consider:

```text
Local transactions
Saga
Outbox
Events
Compensating actions
```

---

# 37. Saga

Example:

```text
Create Order
     ↓
Reserve Inventory
     ↓
Process Payment
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

# 38. Outbox Pattern

Problem:

```text
DB update succeeds
Event publish fails
```

Solution:

```text
Business data
+
Outbox record
```

in one local transaction.

Then:

```text
Outbox
 ↓
Publisher
 ↓
Kafka
```

---

# 39. Idempotency

Distributed systems retry.

Example:

```text
Payment request
 ↓
Payment succeeds
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
One business result
```

---

# 40. Idempotency Key

Example:

```http
POST /payments
Idempotency-Key: abc-123
```

Store the key and result.

Repeated request:

```text
abc-123
```

returns the original result instead of creating another payment.

---

# 41. Timeouts

Never allow dependency calls to wait indefinitely.

Example:

```text
Order → Payment

Connect timeout = 500 ms
Response timeout = 2 sec
```

Actual values should come from requirements and measurements.

---

# 42. Retry

Retry only transient failures.

Good candidates:

```text
Temporary network failure
Temporary service unavailable
Transient database issue
```

Avoid blindly retrying:

```text
Invalid request
Business validation failure
Permanent authorization failure
```

---

# 43. Exponential Backoff

Example:

```text
100ms
200ms
400ms
800ms
```

This avoids immediately overwhelming a recovering dependency.

---

# 44. Retry + Jitter

If 10,000 instances retry simultaneously:

```text
Dependency fails
     ↓
Everyone retries
     ↓
Traffic spike
```

Jitter randomizes retry timing.

---

# 45. Circuit Breaker

States:

```text
CLOSED
OPEN
HALF_OPEN
```

When a dependency repeatedly fails:

```text
CLOSED
   ↓
OPEN
   ↓
Fail fast
```

After a wait:

```text
HALF_OPEN
```

A few requests test recovery.

---

# 46. Bulkhead

Isolate resources between workloads.

Example:

```text
Payment → Pool A
Recommendations → Pool B
```

A slow recommendation service should not consume every resource needed for payments.

---

# 47. Rate Limiting

Protect APIs from excessive traffic.

Example:

```text
100 requests/sec/client
```

Excess traffic:

```text
429 Too Many Requests
```

Rate limits should reflect business requirements.

---

# 48. Load Shedding

When overloaded:

```text
Accept everything
↓
Queue grows
↓
Latency explodes
↓
System fails
```

Load shedding can reject lower-priority work to protect critical functionality.

---

# 49. Backpressure

If:

```text
Producer = 10,000 events/sec
Consumer = 2,000 events/sec
```

lag grows.

Solutions:

```text
Scale consumers
Batch
Optimize processing
Control producer rate
Increase partitions where appropriate
```

---

# 50. Graceful Degradation

If recommendations fail:

```text
Product page
    ↓
Product details → available
Recommendations → fallback/empty
```

A non-critical dependency should not necessarily break the entire user experience.

---

# 51. Availability vs Consistency

Sometimes you must choose what behavior is acceptable during failures.

Example:

```text
Search index temporarily stale
```

may be acceptable.

But:

```text
Inventory says 10 units
when only 1 exists
```

may not be acceptable.

Consistency requirements depend on the business operation.

---

# 52. Strong vs Eventual Consistency

Strong consistency:

```text
Write
 ↓
Read immediately sees new state
```

Eventual consistency:

```text
Write
 ↓
Propagation
 ↓
Read eventually sees new state
```

Use eventual consistency when the business can tolerate it.

---

# 53. CAP Theorem

Under a network partition, a distributed system must trade off between:

```text
Consistency
Availability
```

CAP is specifically about behavior in the presence of partition.

Don't use CAP as a simplistic "pick any two" rule without context.

---

# 54. API Design

A good REST API should define:

```text
Resource
HTTP method
Request
Response
Status codes
Validation
Authentication
Authorization
Pagination
Versioning
Error format
```

---

# 55. REST Resource Example

```http
GET /api/products/100
```

Response:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 79999
}
```

---

# 56. HTTP Status Codes

Common:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
429 Too Many Requests
500 Internal Server Error
503 Service Unavailable
```

Use status codes consistently.

---

# 57. Pagination

Avoid:

```text
GET /products
```

returning millions of rows.

Use:

```text
GET /products?page=0&size=20
```

or cursor/keyset pagination for large datasets.

---

# 58. Cursor Pagination

For large datasets:

```text
GET /products?after=1000&limit=20
```

Database:

```sql
WHERE id > ?
ORDER BY id
LIMIT 20
```

The exact query depends on sort direction and indexed columns.

---

# 59. API Versioning

Example:

```text
/api/v1/orders
/api/v2/orders
```

Version when a breaking contract change requires it.

Prefer backward-compatible evolution where practical.

---

# 60. API Gateway Rate Limiting

Gateway can enforce:

```text
Requests/IP
Requests/user
Requests/API key
Requests/endpoint
```

This protects downstream services before traffic reaches them.

---

# 61. Security in System Design

Important:

```text
HTTPS
Authentication
Authorization
Input validation
Rate limiting
Secrets management
Least privilege
Audit logging
Dependency scanning
```

Security should be designed from the beginning.

---

# 62. JWT in Distributed Systems

A client can send:

```http
Authorization: Bearer <token>
```

Gateway/service validates:

```text
Signature
Expiration
Issuer
Audience
Authorities
```

Services then apply authorization.

---

# 63. Service-to-Service Security

Options include:

```text
OAuth2 access tokens
mTLS
Service identity
```

Don't assume internal network traffic is automatically trusted.

---

# 64. Observability

Three major pillars:

```text
Logs
Metrics
Traces
```

Use them together.

---

# 65. Metrics

Monitor:

```text
Request rate
Error rate
Latency
CPU
Memory
DB connections
Cache hit ratio
Kafka lag
Queue depth
```

---

# 66. Logs

Use structured logs:

```text
service=order-service
event=ORDER_CREATED
orderId=1001
traceId=abc123
```

Never log:

```text
Passwords
JWT tokens
API secrets
Private keys
Sensitive personal data
```

---

# 67. Distributed Tracing

Example:

```text
Trace
 |
 +-- Gateway
 |
 +-- Order Service
 |
 +-- Payment Service
 |
 +-- Database
```

Tracing helps locate where latency is introduced.

OpenTelemetry is a common instrumentation standard.

---

# 68. Correlation ID

A request gets:

```text
correlationId = ABC
```

Every downstream service logs:

```text
ABC
```

This makes cross-service debugging easier.

---

# 69. Health Checks

Two important concepts:

```text
Liveness
Readiness
```

Liveness:

```text
Should this process be restarted?
```

Readiness:

```text
Should this instance receive traffic?
```

---

# 70. Caching Strategy

Ask:

```text
What data is read frequently?
How stale can it be?
How large is it?
How often does it change?
What happens on cache failure?
```

Don't cache everything.

---

# 71. Cache Failure

If Redis goes down:

```text
Application
   ↓
Cache unavailable
```

A possible fallback:

```text
Database
```

But if all requests suddenly bypass Redis:

```text
Database overload
```

Therefore cache failures should be considered in capacity planning.

---

# 72. Database Failure

Possible strategies:

```text
Primary/replica architecture
Failover
Retries for transient failures
Connection timeouts
Circuit breakers
Graceful degradation
```

Do not retry database operations blindly when transaction semantics make retries unsafe.

---

# 73. Database Connection Failure

If all connections are exhausted:

```text
Requests wait
↓
Latency increases
↓
Timeouts
```

Investigate:

```text
Slow queries
Long transactions
Connection leaks
Pool sizing
Database capacity
```

---

# 74. N+1 Query Problem

Example:

```text
1 query → orders
100 queries → customers
```

Total:

```text
101 queries
```

Solutions:

```text
Fetch join
Projection
Batch fetching
Purpose-built queries
```

---

# 75. Read-Heavy System

Suppose:

```text
95% reads
5% writes
```

Potential architecture:

```text
Load Balancer
      |
Multiple API instances
      |
    Redis
      |
Read replicas
      |
Primary DB
```

The exact architecture depends on consistency and workload.

---

# 76. Write-Heavy System

For high writes:

```text
API
 ↓
Queue/Kafka
 ↓
Workers
 ↓
Database
```

can smooth bursts when the business operation can be asynchronous.

But critical synchronous writes still need an appropriate transaction boundary.

---

# 77. High Traffic Product Catalog

Requirements:

```text
Millions of products
High read traffic
Search
Filtering
Sorting
```

Possible design:

```text
CDN where applicable
      ↓
API Gateway
      ↓
Product Service
      ↓
Redis
      ↓
Database
      +
Search Engine
```

Search engine should be introduced when search requirements justify it.

---

# 78. Search Engine

Search engines are useful for:

```text
Full-text search
Fuzzy search
Faceting
Ranking
Complex filtering
```

Keep the primary database as the source of truth where appropriate.

---

# 79. Search Event Flow

```text
Product DB
   ↓
ProductUpdated
   ↓
Kafka
   ↓
Search Indexer
   ↓
Search Engine
```

The search index is eventually consistent.

---

# 80. High Traffic Checkout

Possible architecture:

```text
Client
  ↓
Load Balancer
  ↓
API Gateway
  ↓
Order Service
  |
  +--> Redis
  |
  +--> MySQL
  |
  +--> Kafka
         |
         +--> Payment
         +--> Inventory
         +--> Notification
```

Use strong consistency where the business requires it and eventual consistency where acceptable.

---

# 81. Prevent Overselling

Suppose:

```text
Stock = 1
```

Two users buy simultaneously.

Use database-level concurrency control such as:

```text
Atomic update
Optimistic locking
Pessimistic locking
Reservation
```

Example:

```sql
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = ?
  AND quantity >= 1;
```

Then check affected rows.

---

# 82. Optimistic Locking

Entity:

```java
@Version
private Long version;
```

If two transactions modify the same row:

```text
Transaction A → version 1
Transaction B → version 1
```

One successful update increments the version.

The other detects a version conflict.

---

# 83. Pessimistic Locking

A transaction can lock the row while processing.

Conceptually:

```text
SELECT ... FOR UPDATE
```

This can protect critical inventory operations but may reduce concurrency.

Use only when justified.

---

# 84. Order State Machine

Instead of random status changes:

```text
PENDING
  ↓
CONFIRMED
  ↓
SHIPPED
  ↓
DELIVERED
```

Failure states:

```text
PAYMENT_FAILED
CANCELLED
REFUNDED
```

Define valid transitions explicitly.

---

# 85. Payment State

Example:

```text
INITIATED
   ↓
PENDING
   ↓
SUCCESS
```

or:

```text
PENDING
   ↓
FAILED
```

A clear state machine makes retries and reconciliation easier.

---

# 86. Reconciliation

Distributed systems can reach ambiguous states.

Example:

```text
Payment provider says SUCCESS
Order service says PENDING
```

A reconciliation process can compare systems and repair the state.

This is especially important for payments.

---

# 87. Scheduled Reconciliation

A background job can:

```text
Find PENDING payments older than threshold
      ↓
Query payment provider
      ↓
Update local state
```

Use idempotency and controlled concurrency.

---

# 88. Distributed Scheduler

If multiple application instances run the same scheduled task:

```text
Instance A → job
Instance B → job
Instance C → job
```

you may accidentally execute it multiple times.

Options:

```text
Distributed lock
Leader election
Dedicated worker
External scheduler
```

---

# 89. File Upload System

For large files, avoid routing the entire file through the application server when possible.

Architecture:

```text
Client
  ↓
Pre-signed upload URL
  ↓
Object Storage
```

Application stores metadata:

```text
fileId
ownerId
objectKey
size
contentType
```

---

# 90. Notification System

Architecture:

```text
Business Service
      ↓
Kafka
      ↓
Notification Service
      |
 +----+----+
 |         |
Email      SMS
```

Notifications can be retried independently.

---

# 91. Email Reliability

Don't send email inside a long database transaction:

```text
BEGIN
 ↓
Update DB
 ↓
Call Email Provider
 ↓
COMMIT
```

Instead:

```text
DB transaction
 ↓
Outbox
 ↓
Notification worker
 ↓
Email provider
```

---

# 92. Rate Limiting Architecture

Possible:

```text
Client
  ↓
Gateway
  ↓
Redis counter
  ↓
Service
```

Redis can coordinate rate limits across multiple application instances.

---

# 93. Distributed Rate Limiting

If each instance maintains its own counter:

```text
Instance A → 100
Instance B → 100
Instance C → 100
```

a global limit can be violated.

A shared store or gateway-level limiter can enforce a coordinated limit.

---

# 94. Idempotent Order Creation

Client sends:

```http
POST /orders
Idempotency-Key: ORDER-ABC
```

Server:

```text
Check key
 ↓
Existing?
 ├─ yes → return previous order
 └─ no → create order + store key
```

Use a unique constraint to protect against concurrent duplicate requests.

---

# 95. Distributed Lock vs Database Constraint

For uniqueness:

```text
Database unique constraint
```

is often simpler and more reliable than creating a distributed lock.

Use distributed locks only when the problem truly requires coordination beyond database constraints.

---

# 96. Hot Key

Suppose:

```text
product:popular-100
```

receives millions of requests.

One cache key becomes extremely hot.

Possible approaches:

```text
Local caching
Key replication
Request coalescing
CDN
Traffic distribution
```

Choose based on the access pattern.

---

# 97. Hot Partition

In Kafka:

```text
orderId = same value
```

for a huge number of messages can overload one partition.

Monitor partition-level lag and key distribution.

---

# 98. Thundering Herd

A popular cache entry expires:

```text
Many requests
     ↓
All miss cache
     ↓
All call database
```

Solutions:

```text
Request coalescing
Staggered expiration
Background refresh
Locking
```

---

# 99. Database Hot Row

Example:

```text
One inventory row
```

receives thousands of updates.

Possible solutions:

```text
Atomic operations
Partitioning
Inventory reservation
Queueing
Sharding by stock bucket where justified
```

Don't blindly increase database connections.

---

# 100. Queue-Based Load Leveling

Traffic spike:

```text
10,000 requests/sec
```

Worker capacity:

```text
2,000/sec
```

Queue:

```text
Requests → Queue → Workers
```

The queue absorbs bursts.

But the system must define:

```text
Maximum queue size
Latency limits
Expiration
Dead-letter behavior
Backpressure
```

---

# 101. Graceful Overload

When capacity is exceeded:

```text
Rate limit
Load shed
Queue
Return 429/503
```

It's better to fail predictably than collapse unpredictably.

---

# 102. Disaster Recovery

Important concepts:

```text
RPO
RTO
Backups
Replication
Failover
Recovery testing
```

---

# 103. RPO

Recovery Point Objective:

```text
How much data can we afford to lose?
```

Example:

```text
RPO = 5 minutes
```

The system should be designed so recoverable data loss is within that target.

---

# 104. RTO

Recovery Time Objective:

```text
How quickly must the system recover?
```

Example:

```text
RTO = 30 minutes
```

---

# 105. Backup Strategy

Consider:

```text
Full backups
Incremental backups
Retention
Encryption
Off-site storage
Restore testing
```

A backup that has never been restored is not enough evidence of recoverability.

---

# 106. Multi-AZ Deployment

For high availability:

```text
Zone A
  App 1
  DB replica

Zone B
  App 2
  DB replica

Zone C
  App 3
```

Avoid putting all critical instances in one failure domain.

---

# 107. Multi-Region

For stronger disaster tolerance:

```text
Region A
    |
    +---- Region B
```

But multi-region introduces:

```text
Replication complexity
Latency
Consistency challenges
Cost
Operational complexity
```

Use it when the availability/disaster requirements justify it.

---

# 108. Deployment Strategy

Common strategies:

```text
Rolling
Blue-Green
Canary
```

Choose based on:

```text
Risk
Rollback needs
Infrastructure
Traffic
Database compatibility
```

---

# 109. Zero-Downtime Deployment

Requirements:

```text
Multiple instances
Health checks
Backward-compatible APIs
Backward-compatible DB migrations
Graceful shutdown
Load balancer
```

---

# 110. Database Migration Strategy

Safe rollout:

```text
1. Add new nullable column
2. Deploy code supporting old + new
3. Backfill
4. Start using new field
5. Remove old field later
```

This avoids breaking old and new application versions during deployment.

---

# 111. Docker

Package Spring Boot application:

```text
Spring Boot
    ↓
JAR
    ↓
Docker Image
    ↓
Container
```

Benefits:

```text
Consistent runtime
Portable deployment
Isolation
```

---

# 112. Kubernetes

Kubernetes manages:

```text
Pods
Deployments
Services
Ingress
ConfigMaps
Secrets
Autoscaling
Health checks
```

---

# 113. Horizontal Pod Autoscaling

Example:

```text
CPU/metrics increase
       ↓
3 pods → 6 pods
```

Autoscaling should account for:

```text
CPU
Memory
Request rate
Queue depth
Custom metrics
```

where appropriate.

---

# 114. Autoscaling Doesn't Fix Bottlenecks Automatically

Suppose:

```text
10 API instances
      ↓
One database
```

Adding:

```text
100 API instances
```

may overload the database.

Always identify the bottleneck.

---

# 115. System Design Tradeoffs

Every design has tradeoffs.

Examples:

```text
Cache → speed vs staleness
Replica → read scaling vs lag
Async → decoupling vs eventual consistency
Microservices → autonomy vs complexity
Sharding → scale vs query complexity
Multi-region → resilience vs cost
```

A strong interview answer explains the tradeoff.

---

# 116. E-Commerce High-Level Architecture

```text
                         Users
                           |
                           v
                     CDN / WAF
                           |
                           v
                    Load Balancer
                           |
                           v
                     API Gateway
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
       Product           Order            User
       Service           Service          Service
          |                |                |
        Redis            MySQL            MySQL
          |                |
          |              Outbox
          |                |
          |                v
          |              Kafka
          |         +------+------+------+
          |         |      |      |      |
          |         v      v      v      v
          |      Payment Inventory Notification
          |      Service  Service   Service
          |         |       |          |
          |         v       v          v
          |       DB      DB       Email/SMS
          |
       Search
       Engine
```

---

# 117. E-Commerce Request Flow

Product read:

```text
Client
 ↓
Gateway
 ↓
Product Service
 ↓
Redis
 ↓ hit
Response
```

Cache miss:

```text
Redis
 ↓ miss
Database
 ↓
Redis
 ↓
Response
```

---

# 118. E-Commerce Order Flow

```text
Client
 ↓
Gateway
 ↓
Order Service
 ↓
Create PENDING order
 ↓
Outbox
 ↓
Kafka
 ├── Inventory
 ├── Payment
 └── Notification
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

---

# 119. E-Commerce Failure Flow

Payment fails:

```text
PaymentFailed
     ↓
Order Service
     ↓
CANCELLED
     ↓
InventoryReleased
```

Notification failure should not necessarily cancel a successful order.

---

# 120. Product Search Flow

```text
Product DB
   ↓
ProductUpdated
   ↓
Kafka
   ↓
Search Indexer
   ↓
Search Engine
```

The search index can be eventually consistent.

---

# 121. Product Update Consistency

Source of truth:

```text
Product Database
```

Search index:

```text
Derived/read model
```

If indexing fails:

```text
Product DB = correct
Search = temporarily stale
```

A retryable indexing workflow can repair the search state.

---

# 122. High-Traffic Product Read

Optimize with:

```text
CDN
Redis
Database indexes
Read replicas
DTOs
Pagination
Compression
```

Measure before adding every layer.

---

# 123. High-Traffic Checkout

Protect:

```text
Order Service
Payment
Inventory
Database
```

using:

```text
Rate limiting
Idempotency
Timeouts
Circuit breakers
Bulkheads
Queueing where appropriate
```

---

# 124. Payment Safety

Payment operations should have:

```text
Idempotency
Clear states
Timeouts
Controlled retries
Reconciliation
Audit logs
Secure credentials
```

Never assume a network timeout means payment definitely failed.

---

# 125. Inventory Safety

Inventory updates should consider:

```text
Concurrency
Atomic updates
Optimistic locking
Reservations
Consistency
Idempotency
```

Avoid allowing stock to become negative.

---

# 126. Notification Safety

Notifications should be:

```text
Asynchronous
Retryable
Idempotent where possible
Rate limited
Observable
```

A notification failure should not necessarily affect the core transaction.

---

# 127. API Gateway Failure

If the gateway is a single instance:

```text
Gateway down
↓
Entire API unavailable
```

Use multiple instances:

```text
Load Balancer
   |
+--+--+
A     B
```

and appropriate health checks.

---

# 128. Redis Failure

Possible fallback:

```text
Redis down
 ↓
Database
```

But protect the database from a cache stampede using:

```text
Rate limiting
Request coalescing
Fallback controls
```

---

# 129. Kafka Failure

If Kafka is temporarily unavailable:

```text
Order transaction
```

should not necessarily disappear.

Outbox:

```text
Order DB
+
Outbox
```

can preserve the event for later publishing.

---

# 130. Database Failure

If the primary database fails:

```text
Failover/replica
```

may restore service.

But application design must account for:

```text
Connection errors
Retries
Transactions
Data consistency
Recovery
```

---

# 131. System Design Interview Structure

Use this order:

```text
1. Requirements
2. Scale estimates
3. APIs
4. Data model
5. High-level architecture
6. Detailed components
7. Scaling
8. Reliability
9. Security
10. Observability
11. Tradeoffs
```

This keeps the discussion organized.

---

# 132. Example: Design an Order Service

Requirements:

```text
Create order
Get order
Cancel order
View order history
```

APIs:

```http
POST /orders
GET /orders/{id}
POST /orders/{id}/cancel
GET /users/{id}/orders
```

Data:

```text
orders
order_items
```

Reliability:

```text
Idempotency
Transactions
Outbox
Events
```

---

# 133. Order Database Model

```text
orders
------
id
user_id
status
total_amount
created_at
updated_at

order_items
-----------
id
order_id
product_id
quantity
unit_price
```

Indexes:

```text
orders(user_id)
orders(status)
order_items(order_id)
```

Exact indexes depend on query patterns.

---

# 134. Create Order Transaction

Local transaction:

```text
BEGIN
 ↓
Validate request
 ↓
Create order
 ↓
Create order items
 ↓
Create outbox event
 ↓
COMMIT
```

Then:

```text
Outbox Publisher
 ↓
Kafka
```

---

# 135. Cancel Order

Validate:

```text
Order exists
User authorized
Order state allows cancellation
```

Then:

```text
Update status
Create event
Commit
```

Possible event:

```text
OrderCancelled
```

---

# 136. Prevent Duplicate Create Order

Use:

```text
Idempotency-Key
```

and a unique database constraint.

Example:

```text
(user_id, idempotency_key)
```

This protects against concurrent retries.

---

# 137. Order History

Query:

```sql
SELECT *
FROM orders
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT ?;
```

For very large histories, use keyset pagination.

---

# 138. Order Search

If requirements include:

```text
Search by product
Search by customer
Search by status
Date range
Full-text search
```

evaluate whether relational indexes are enough before introducing a dedicated search engine.

---

# 139. API Error Model

A consistent response:

```json
{
  "timestamp": "2026-08-20T10:00:00Z",
  "status": 409,
  "code": "ORDER_ALREADY_CANCELLED",
  "message": "Order cannot be cancelled",
  "traceId": "abc123"
}
```

Avoid exposing stack traces.

---

# 140. Correlation and Trace IDs

Every request can carry:

```text
traceId
correlationId
```

Example:

```text
Gateway
 ↓ ABC
Order
 ↓ ABC
Payment
 ↓ ABC
Inventory
```

This is invaluable during production debugging.

---

# 141. Logging Strategy

Log:

```text
Request ID
Business entity ID
Event
Result
Latency
Error category
```

Avoid:

```text
Passwords
Tokens
Secrets
Sensitive payloads
```

---

# 142. Monitoring Dashboard

Useful dashboard:

```text
Request rate
p50
p95
p99
5xx
4xx
CPU
Memory
DB pool
DB latency
Redis hit ratio
Kafka lag
External API latency
```

---

# 143. Alerting

Alert on meaningful conditions:

```text
p99 latency high
5xx rate high
DB connections exhausted
Kafka lag growing
Disk nearly full
Memory pressure
Service unavailable
```

Avoid noisy alerts that operators learn to ignore.

---

# 144. Performance Bottleneck Analysis

If API is slow:

```text
Trace request
 ↓
Gateway
 ↓
Service
 ↓
Database
```

Suppose:

```text
Service = 100 ms
DB = 1.5 sec
```

Optimize database first.

Don't optimize Java code that contributes only 10ms.

---

# 145. Capacity Planning

Consider:

```text
Current traffic
Expected growth
Peak multiplier
CPU
Memory
Database capacity
Cache capacity
Kafka partitions
Network bandwidth
Storage
```

Plan for headroom rather than running every component at maximum capacity.

---

# 146. Cost Optimization

Ask:

```text
Can caching reduce DB load?
Can autoscaling reduce idle resources?
Can storage tiers reduce cost?
Can CDN reduce origin traffic?
Can async processing reduce synchronous infrastructure?
```

Cost is a system-design constraint too.

---

# 147. Reliability Budget

Don't design every component for extreme requirements if the business doesn't need them.

Example:

```text
99.9% availability
```

may be enough for one feature, while payment processing may require much stronger operational guarantees.

Match architecture to business value.

---

# 148. Simplicity First

Start with:

```text
Load Balancer
Spring Boot
MySQL
Redis if justified
```

Then add:

```text
Kafka
Search engine
Read replicas
Sharding
Multi-region
```

only when requirements justify them.

---

# 149. Avoid Premature Microservices

A modular monolith can be a good starting point:

```text
Single Spring Boot app
  |
+-- User module
+-- Product module
+-- Order module
+-- Payment module
```

Split services when independent deployment/scaling or domain boundaries provide real value.

---

# 150. System Design Interview: Strong Answer

Question:

> How would you design a scalable Spring Boot ecommerce backend?

Answer:

> I would start by clarifying traffic, consistency, availability, and business requirements. At a high level, I'd use a load balancer and stateless Spring Boot services. I'd use MySQL for transactional data, Redis for carefully selected read-heavy data, and Kafka for asynchronous workflows such as notifications and downstream order processing. For order creation, I'd use a local database transaction with an Outbox Pattern and idempotency key. I'd add timeouts, controlled retries, circuit breakers, observability, and resource-level authorization. Finally, I'd scale horizontally and introduce read replicas, search infrastructure, or sharding only when measurements and requirements justify them.

---

# 151. System Design Interview: How Do You Scale a Spring Boot API?

> I first identify the bottleneck. If the application is stateless, I can horizontally scale instances behind a load balancer. Then I look at database capacity, connection pools, caching, external dependencies, and network latency. I use metrics and p95/p99 latency to verify improvements rather than simply adding more instances.

---

# 152. System Design Interview: How Do You Handle Millions of Requests?

> I would start with capacity estimation and identify read/write patterns. Then I'd use horizontal application scaling, load balancing, caching, database indexing, read replicas where appropriate, asynchronous processing for suitable workloads, and rate limiting. For extreme scale, I would evaluate partitioning or sharding, but only after simpler bottlenecks are addressed.

---

# 153. System Design Interview: How Do You Make a System Highly Available?

> I avoid single points of failure by running multiple application instances across failure domains, using health checks and load balancing, and designing dependencies for failover. For data, I use replication and backups appropriate to the RPO and RTO. I also add monitoring, alerting, graceful degradation, and tested recovery procedures.

---

# 154. System Design Interview: How Do You Design for Failure?

> I assume networks can fail, services can become slow, messages can be duplicated, and instances can restart. I use timeouts, carefully selected retries with backoff and jitter, circuit breakers, bulkheads, idempotency, dead-letter handling, and observability. The goal is to prevent a local failure from becoming a cascading failure.

---

# 155. System Design Interview: Why Redis?

> Redis is useful when frequently accessed data can be served faster than querying the database repeatedly. I use cache-aside for appropriate data and define TTL and invalidation behavior. I also plan for Redis failure because sending every cache miss directly to the database can create a thundering herd.

---

# 156. System Design Interview: Why Kafka?

> Kafka is useful when I need durable asynchronous event streaming, multiple independent consumers, high throughput, or replay. I would not introduce it for simple synchronous CRUD without a real requirement because it adds operational and consistency complexity.

---

# 157. System Design Interview: How Do You Avoid Duplicate Payments?

> I use an idempotency key backed by a unique constraint and store the result of the payment operation. If the client retries because the original response was lost, the same logical request returns the existing result rather than charging again. I also use controlled retries and reconciliation for ambiguous provider responses.

---

# 158. System Design Interview: How Do You Prevent Overselling?

> I enforce inventory consistency at the database or reservation layer rather than relying only on application checks. An atomic conditional update or optimistic locking can ensure that two concurrent requests cannot both consume the same final unit of stock.

---

# 159. System Design Interview: How Do You Handle a Slow Payment Service?

> I'd configure a timeout so requests don't wait indefinitely. For transient failures I might use limited retries with backoff and jitter, but only if the payment operation is idempotent. A circuit breaker can prevent repeated calls when the dependency is unhealthy, and the business flow may move the payment into a PENDING state if asynchronous processing is acceptable.

---

# 160. System Design Interview: How Do You Handle Database Overload?

> First I identify the cause using slow-query analysis, connection-pool metrics, execution plans, and database CPU/IO metrics. Then I address query and index problems, reduce unnecessary reads, add caching, consider read replicas for read-heavy workloads, and tune connection pools carefully. I wouldn't simply add more application instances because that could increase database pressure.

---

# 161. System Design Interview: How Do You Handle a Traffic Spike?

> I use horizontal scaling and autoscaling for stateless services, caching for read-heavy traffic, rate limiting and load shedding to protect the system, and queues for workloads that can be asynchronous. I also make sure downstream systems such as the database and payment provider can handle the increased load.

---

# 162. System Design Interview: What Makes a Good Architecture?

A good architecture is:

```text
Simple enough to operate
Scalable enough for requirements
Reliable enough for business needs
Secure
Observable
Cost-aware
Easy to evolve
```

There is no universally perfect architecture.

---

# 163. Final System Design Checklist

```text
□ Requirements
□ Functional requirements
□ Non-functional requirements
□ Scale estimation
□ RPS
□ Storage
□ Bandwidth
□ API design
□ Database
□ Indexes
□ Connection pooling
□ Caching
□ Load balancing
□ Horizontal scaling
□ Messaging
□ Idempotency
□ Transactions
□ Saga
□ Outbox
□ Timeouts
□ Retry
□ Circuit breaker
□ Bulkhead
□ Rate limiting
□ Backpressure
□ Security
□ Logs
□ Metrics
□ Tracing
□ Health checks
□ Deployment
□ Backups
□ RPO
□ RTO
□ Disaster recovery
□ Cost
□ Tradeoffs
```

---

# 164. Final Mental Model

```text
                    SYSTEM DESIGN
                          |
        +-----------------+-----------------+
        |                 |                 |
    Requirements        Scale            Data
        |                 |                 |
     APIs              RPS/Storage      DB/Cache
        |                 |                 |
        +-----------------+-----------------+
                          |
                     Architecture
                          |
          +---------------+---------------+
          |               |               |
       Services        Messaging       Security
          |               |               |
       Spring Boot      Kafka          Auth/JWT
          |               |               |
          +---------------+---------------+
                          |
                     Reliability
                          |
       Timeout + Retry + Circuit Breaker
       Idempotency + Outbox + Backpressure
                          |
                     Observability
                          |
             Logs + Metrics + Traces
                          |
                       Deploy
                          |
               Scale + Monitor + Recover
```

---

# 165. Final Rule

> **Good system design is not about adding the most technologies. It is about choosing the simplest architecture that satisfies the requirements, then explaining how it will scale, fail, recover, and evolve.**

---

# 166. Quick Interview Formula

When the interviewer gives you a system-design problem, remember:

```text
R → Requirements
S → Scale
A → APIs
D → Data
C → Cache
M → Messaging
R → Reliability
S → Security
O → Observability
T → Tradeoffs
```

Then walk through the architecture logically instead of jumping directly into implementation details.
