# Spring Boot — Real-World Scenarios

This file focuses on practical Spring Boot backend situations that commonly appear in experienced Java interviews.

The goal is not just to know annotations, but to explain how you would design, debug, secure, optimize, and operate a real application.

---

# 1. Production API Is Suddenly Slow

Start with:

```text
Impact
↓
Metrics
↓
Logs
↓
Tracing
↓
Database
↓
External dependencies
↓
Recent changes
```

Check:

```text
Request rate
Error rate
p95/p99 latency
CPU
Memory
GC
Thread pools
Database latency
Connection pool
External API latency
```

Interview answer:

> I would first confirm the scope and impact using metrics. Then I would use logs and traces to identify where the latency is introduced. I would check database queries, connection-pool usage, external dependencies, recent deployments, and infrastructure before making a change.

---

# 2. Database Connection Pool Is Exhausted

Symptoms:

```text
Requests waiting
High latency
Connection timeout
Hikari pool warnings
```

Possible causes:

```text
Slow queries
Long transactions
Too many concurrent requests
Connection leak
Pool too small
Database overloaded
```

Investigation:

```text
Active connections
Idle connections
Pending threads
Query latency
Transaction duration
Database health
```

Do not immediately increase the pool size.

> Increasing the pool can hide the real bottleneck and may overload the database further.

---

# 3. Application Has High CPU

Check:

```text
CPU metrics
Thread dumps
GC activity
Recent deployments
Hot methods
Infinite loops
Large computations
Serialization
Regex processing
```

Interview answer:

> I would confirm whether the CPU is application CPU or GC-related. Then I would inspect thread activity and profiling data to find the expensive code path instead of guessing.

---

# 4. Application Has High Memory Usage

Possible causes:

```text
Large collections
Unbounded caches
Memory leaks
Large API responses
Large database result sets
Thread growth
Excessive object creation
```

Approach:

```text
Monitor heap
↓
Check GC
↓
Take heap dump if required
↓
Identify retained objects
↓
Find ownership/reference path
↓
Fix
```

---

# 5. OutOfMemoryError

Do not simply increase:

```text
-Xmx
```

First determine why memory is growing.

Useful investigation:

```text
Heap dump
GC logs
Memory metrics
Allocation profiling
Thread count
Cache size
Large collections
```

Interview answer:

> Increasing heap can be a temporary mitigation, but I would first identify whether the problem is a memory leak, excessive workload, oversized objects, or an incorrectly configured cache.

---

# 6. API Returns 500

First determine:

```text
Which endpoint?
Which requests?
When did it start?
How many users?
What exception?
```

Then:

```text
Correlation ID
↓
Application logs
↓
Stack trace
↓
Trace
↓
Recent deployment
↓
Database/external dependency
```

Never assume every 500 is a code bug.

It could be:

```text
Database outage
Dependency timeout
Configuration problem
Infrastructure issue
Unexpected application exception
```

---

# 7. API Returns 400 Unexpectedly

Check:

```text
Request JSON
Content-Type
DTO validation
Required fields
Type conversion
Query parameters
Path variables
Custom validators
```

Example:

```json
{
  "price": "abc"
}
```

when the DTO expects:

```java
BigDecimal price
```

This is a binding/conversion issue rather than a normal `@Positive` validation failure.

---

# 8. API Returns 401

Likely causes:

```text
Missing token
Expired token
Invalid token
Wrong issuer
Wrong audience
Invalid signature
Authentication configuration
```

Check:

```text
Authorization header
Token expiration
Issuer
Signing key
Security filter configuration
```

---

# 9. API Returns 403

The user is usually authenticated but lacks permission.

Example:

```text
USER
 ↓
DELETE /api/products/101
 ↓
ADMIN required
 ↓
403
```

Check:

```text
Authorities
Roles
@PreAuthorize
Security configuration
Resource ownership
```

---

# 10. Product Endpoint Is Returning Sensitive Data

Suppose the entity contains:

```text
id
name
price
costPrice
supplierId
internalStatus
```

Do not return the entity directly.

Use:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {
}
```

DTOs control the public contract.

---

# 11. Two Users Buy the Last Product

Initial stock:

```text
1
```

Requests:

```text
User A → buy 1
User B → buy 1
```

A simple:

```text
read stock
↓
check
↓
decrease
```

can suffer from a race condition.

Possible solutions:

```text
Optimistic locking
Pessimistic locking
Atomic database update
Inventory reservation
```

---

# 12. Atomic Stock Update

Instead of:

```text
SELECT stock
↓
Java checks stock
↓
UPDATE stock
```

consider an atomic operation:

```sql
UPDATE products
SET stock = stock - 1
WHERE id = ?
  AND stock >= 1;
```

Then verify:

```text
affected rows == 1
```

This can prevent overselling for the specific operation.

---

# 13. Optimistic Locking for Inventory

Entity:

```java
@Version
private Long version;
```

Two transactions read:

```text
version = 5
```

First update:

```text
5 → 6
```

Second update:

```text
5 → conflict
```

The application can retry or return an appropriate conflict response depending on the business workflow.

---

# 14. Duplicate Order Creation

Problem:

```text
User clicks Pay
↓
Request succeeds
↓
Network response is lost
↓
User clicks again
↓
Second request
```

Possible duplicate order/payment.

Solution:

```text
Idempotency key
```

Example:

```http
Idempotency-Key: checkout-abc-123
```

Store the key with the result.

---

# 15. Payment API Times Out

Bad approach:

```text
Retry payment indefinitely
```

Better:

```text
Timeout
↓
Determine whether operation is safely retryable
↓
Use idempotency key
↓
Controlled retry if appropriate
↓
Circuit breaker
↓
Async reconciliation if necessary
```

Never assume a timeout means the payment definitely failed.

The request may have succeeded while the response was lost.

---

# 16. Payment Request Timed Out

Important distinction:

```text
Timeout
≠
Business operation definitely failed
```

Possible states:

```text
Payment succeeded
Payment failed
Payment still processing
Response was lost
```

A robust payment workflow may require:

```text
Payment status API
Webhook
Reconciliation job
Idempotency
```

---

# 17. External Service Is Down

Suppose:

```text
Order Service
      ↓
Shipping Service
      X
```

Do not allow every request to wait indefinitely.

Use:

```text
Timeout
Circuit breaker
Fallback where meaningful
Async processing
Retry with backoff
```

---

# 18. Circuit Breaker Scenario

If Payment Service repeatedly fails:

```text
CLOSED
  ↓
Failures increase
  ↓
OPEN
  ↓
Fail fast
  ↓
Wait
  ↓
HALF_OPEN
  ↓
Test request
  ↓
Recovered → CLOSED
```

This protects the calling service from repeatedly spending resources on a failing dependency.

---

# 19. Retry Scenario

Retry is appropriate for some transient failures:

```text
Temporary network failure
503 Service Unavailable
Transient infrastructure issue
```

Avoid retrying blindly for:

```text
400
401
403
Business validation failure
Non-idempotent payment operation
```

The exact retry policy should be based on the dependency contract.

---

# 20. N+1 Query in Production

Endpoint:

```text
GET /api/orders
```

Code loads:

```text
Orders
↓
For each order
↓
Customer
↓
Items
```

Database:

```text
1 query for orders
+
N queries for customers
+
N queries for items
```

This can become expensive quickly.

Fix using:

```text
Fetch joins
Entity graphs
DTO projections
Batch fetching
Purpose-built queries
```

---

# 21. Large API Response

Bad:

```text
GET /api/products
→ 500,000 products
```

Problems:

```text
Database load
Memory usage
Network bandwidth
Serialization time
Client processing
```

Use:

```text
Pagination
Filtering
Sorting
Field selection where appropriate
```

---

# 22. Pagination Design

Example:

```http
GET /api/products?page=0&size=20
```

For very large datasets, offset pagination can become expensive.

Consider cursor/keyset pagination:

```text
GET /api/products?after=product_1000&limit=20
```

when the use case benefits from it.

---

# 23. Slow Search Query

Approach:

```text
Measure query
↓
EXPLAIN / execution plan
↓
Check indexes
↓
Check predicates
↓
Check joins
↓
Check returned columns
↓
Optimize
```

Don't add indexes blindly.

Indexes improve reads but add:

```text
Storage
Write cost
Maintenance
```

---

# 24. Database Index Scenario

Query:

```sql
SELECT *
FROM products
WHERE sku = ?;
```

If SKU is frequently searched:

```sql
CREATE UNIQUE INDEX idx_products_sku
ON products(sku);
```

A unique constraint may be preferable when uniqueness is a business requirement because it also enforces integrity.

---

# 25. Transaction Is Too Long

Bad:

```text
BEGIN
↓
Update DB
↓
Call payment API
↓
Wait 10 seconds
↓
Call shipping API
↓
COMMIT
```

Problems:

```text
Connection held
Locks held
Reduced concurrency
Higher timeout risk
```

Better:

```text
Keep local DB transactions short
Use events/Saga for distributed workflows
```

---

# 26. Transaction Rollback Doesn't Happen

Possible cause:

```text
Exception caught and swallowed
```

Example:

```java
try {
    repository.save(order);
} catch (Exception e) {
    log.error("failed", e);
}
```

The transaction may not see an exception to trigger rollback.

Depending on the desired behavior, rethrow a suitable exception or configure rollback rules explicitly.

---

# 27. Checked Exception and Rollback

By default, Spring declarative transaction management generally rolls back for unchecked exceptions.

If required:

```java
@Transactional(
    rollbackFor = IOException.class
)
```

Use this intentionally.

---

# 28. Redis Cache Is Returning Stale Data

Possible causes:

```text
Missing eviction
Long TTL
Write/update path didn't invalidate cache
Multiple cache writers
Event processing delay
```

Possible solutions:

```text
TTL
Explicit eviction
Cache-aside discipline
Event-based invalidation
Versioned cache keys
```

Choose based on consistency requirements.

---

# 29. Cache Stampede

Suppose a popular cache entry expires:

```text
10,000 requests
      ↓
Cache miss
      ↓
10,000 DB queries
```

This can overload the database.

Possible techniques:

```text
Jittered TTL
Request coalescing
Locking
Background refresh
Pre-warming
```

---

# 30. Redis Is Down

Do not automatically make Redis a single point of application failure.

For non-critical cache data:

```text
Redis unavailable
↓
Fallback to database
```

For critical Redis-backed functionality, the failure strategy must be explicit.

---

# 31. Kafka Consumer Receives the Same Event Twice

Example:

```text
OrderPaid event
OrderPaid event
```

Use idempotent processing:

```text
eventId
↓
Check processed-events
↓
Already processed?
    ├── yes → ignore
    └── no  → process + record
```

---

# 32. Kafka Consumer Fails Repeatedly

Potential strategy:

```text
Retry
↓
Retry limit
↓
Dead Letter Topic/Queue
↓
Alert
↓
Investigate
```

Do not allow one permanently bad message to block the entire consumer workflow indefinitely.

---

# 33. Kafka Consumer Is Too Slow

Check:

```text
Consumer lag
Processing time
Partition count
Consumer count
Database latency
External API latency
Batch size
```

Scaling consumers is limited by partitioning.

```text
3 partitions
→ at most 3 actively consuming consumers
```

within one consumer group for that topic partition set.

---

# 34. Event Is Published but Database Update Fails

This is one reason event ordering matters.

Suppose:

```text
Publish OrderCreated
↓
Database transaction fails
```

Now consumers believe the order exists.

A common solution for database-to-event consistency is:

```text
Outbox Pattern
```

---

# 35. Outbox Scenario

Within one database transaction:

```text
Create Order
+
Insert Outbox Event
```

Both succeed or roll back together.

Then:

```text
Outbox Publisher
↓
Kafka
```

This separates:

```text
Local consistency
```

from:

```text
Event delivery
```

---

# 36. Service Has Circular Dependency

Example:

```text
OrderService
    ↓
PaymentService
    ↓
OrderService
```

Don't immediately solve it with lazy injection.

Ask:

```text
Why do both services need each other?
```

Possible solution:

```text
Extract shared responsibility
Change ownership
Use events
Redesign service boundary
```

---

# 37. Microservice Shares Another Service's Database

Bad:

```text
Order Service
      ↓
Payment DB
```

This creates tight coupling.

Better:

```text
Order Service
      ↓
Payment API/Event
      ↓
Payment Service
      ↓
Payment DB
```

Each service should normally own its data.

---

# 38. Service Discovery Failure

If a service cannot discover:

```text
PAYMENT-SERVICE
```

check:

```text
Registry health
Service registration
Network/DNS
Service name
Configuration
Credentials
Kubernetes Service
```

In Kubernetes, prefer platform-native service discovery where appropriate rather than automatically adding Eureka.

---

# 39. API Gateway Is Down

If all traffic goes:

```text
Client
 ↓
Gateway
 ↓
Services
```

the gateway is critical infrastructure.

Use:

```text
Multiple gateway instances
Load balancer
Health checks
Horizontal scaling
```

Avoid a single gateway instance.

---

# 40. Gateway Has Too Much Logic

Bad:

```text
Gateway
 ↓
Business rules
 ↓
Database
 ↓
Payment decisions
```

Better:

```text
Gateway
 ↓
Routing/security/cross-cutting concerns
 ↓
Business service
```

Business logic should remain in the appropriate service.

---

# 41. Security Issue: User Can Access Another User's Order

Endpoint:

```text
GET /api/orders/101
```

User B requests User A's order.

Authentication alone is not enough.

The service must check:

```text
Authenticated user
+
Order ownership/authorization
```

Example concept:

```java
if (!order.belongsTo(userId)) {
    throw new AccessDeniedException(...);
}
```

---

# 42. JWT Expired

The backend should reject an expired token.

```text
JWT
↓
Expiration check
↓
Expired
↓
401 Unauthorized
```

Don't simply trust the token because it is structurally valid.

---

# 43. JWT Key Rotation

Production systems may rotate signing keys.

Design for:

```text
Key IDs
Multiple active verification keys
Grace period
JWKS where applicable
```

Do not hardcode a single forever-key without a rotation strategy.

---

# 44. Password Brute Force

For login endpoints consider:

```text
Rate limiting
Account protection
Monitoring
Strong password hashing
MFA where appropriate
```

Avoid storing plaintext passwords.

---

# 45. SQL Injection

Bad:

```java
String sql =
    "SELECT * FROM products WHERE name = '"
    + name
    + "'";
```

Better:

```text
Prepared statements
JPA parameters
Repository query parameters
```

Never concatenate untrusted input into SQL.

---

# 46. Mass Assignment

Don't blindly bind API JSON to an entity with sensitive fields.

Bad:

```text
User entity
+
isAdmin
+
passwordHash
```

A malicious request could attempt:

```json
{
  "isAdmin": true
}
```

Use dedicated request DTOs.

---

# 47. CORS Misconfiguration

Bad:

```text
Allow every origin
Allow every method
Allow credentials
```

without understanding the security implications.

Configure only required origins and methods.

---

# 48. Sensitive Actuator Endpoints

Do not expose every actuator endpoint publicly.

Especially protect:

```text
Environment information
Configuration details
Heap dumps
Mappings
Beans
```

Expose only what operations actually require.

---

# 49. Logs Contain Passwords

If logs contain:

```text
password=secret123
```

this is a security incident.

Fix:

```text
Redaction
Structured logging
Safe DTO logging
Code review
Secret scanning
```

Never log credentials.

---

# 50. Production Database Migration

Never manually edit production schema without a controlled process.

Use:

```text
Flyway
Liquibase
```

Migration:

```text
V1
↓
V2
↓
V3
```

Run migrations through deployment automation.

---

# 51. Breaking Database Change

Suppose old application expects:

```text
customer_name
```

and new application expects:

```text
name
```

Don't immediately rename/drop the old column during a rolling deployment.

Safer:

```text
Add new column
↓
Deploy backward-compatible application
↓
Write both
↓
Backfill
↓
Switch reads
↓
Remove old column later
```

---

# 52. Zero-Downtime Deployment

A common rolling deployment:

```text
Version 1
+ Version 2
     ↓
Traffic gradually moves
     ↓
Version 1 removed
```

The application should remain compatible during the transition.

This applies to:

```text
API
Database schema
Events
Configuration
```

---

# 53. Deployment Fails

Good deployment process:

```text
Build
↓
Tests
↓
Security checks
↓
Deploy
↓
Health checks
↓
Smoke tests
↓
Monitor
```

Have a rollback strategy.

---

# 54. Health Check Is Green but Application Is Broken

A simple:

```text
/health → UP
```

doesn't guarantee every business operation works.

Use appropriate:

```text
Liveness
Readiness
Dependency health
Synthetic checks
Business metrics
```

Do not make liveness depend on every external system or the application may restart unnecessarily.

---

# 55. Graceful Shutdown

When an instance receives shutdown:

```text
Stop accepting new traffic
↓
Finish active requests
↓
Close resources
↓
Exit
```

This prevents unnecessary request failures during deployments.

---

# 56. Thread Pool Exhaustion

Symptoms:

```text
Requests waiting
High latency
Timeouts
CPU may be normal
```

Possible causes:

```text
Slow database
Slow external API
Blocking I/O
Too many concurrent tasks
Thread leak
```

Check:

```text
Thread dump
Executor metrics
Connection pools
Dependency latency
```

---

# 57. Async Task Queue Is Growing

If:

```text
Produced tasks > Consumed tasks
```

queue depth grows.

Check:

```text
Consumer capacity
Processing latency
Error rate
External dependencies
Partitioning
Thread pools
```

Don't simply increase consumers if the downstream database is already overloaded.

---

# 58. Scheduled Job Runs Twice

Suppose application has 3 instances:

```text
Instance A → scheduled job
Instance B → scheduled job
Instance C → scheduled job
```

All may execute it.

Solutions:

```text
Distributed lock
Dedicated scheduler
Platform CronJob
Idempotent processing
```

Choose based on requirements.

---

# 59. Email Sent Twice

A request is retried:

```text
Create order
↓
Send email
↓
Response lost
↓
Retry
↓
Send email again
```

Possible solutions:

```text
Idempotency
Email event with unique ID
Outbox
Consumer deduplication
```

---

# 60. Order Workflow Design

A realistic workflow:

```text
Create Order
     ↓
Reserve Inventory
     ↓
Process Payment
     ↓
Confirm Order
     ↓
Publish OrderConfirmed
     ↓
Send Notification
```

Notification doesn't necessarily need to block checkout.

Use asynchronous events for non-critical side effects where appropriate.

---

# 61. Partial Failure in Order Workflow

Payment succeeds but notification fails:

```text
Payment = SUCCESS
Notification = FAILED
```

Do not roll back payment simply because email failed.

Separate critical business transactions from secondary side effects.

Retry notification asynchronously.

---

# 62. Saga Example

```text
Order Created
     ↓
Inventory Reserved
     ↓
Payment Successful
     ↓
Order Confirmed
```

If payment fails:

```text
Release Inventory
↓
Cancel Order
```

These compensating actions form part of the Saga design.

---

# 63. Distributed Transaction Interview Answer

> I would avoid trying to make one database transaction span multiple services. I would use local transactions combined with Saga, events, idempotency, and compensation depending on the workflow.

---

# 64. API Idempotency

Safe example:

```text
PUT /products/101
```

is generally designed to be idempotent when repeated requests produce the same intended resource state.

For POST operations where duplicate creation is dangerous:

```text
Idempotency-Key
```

can provide application-level protection.

---

# 65. API Rate Limiting

Protect endpoints such as:

```text
Login
OTP
Search
Checkout
Public APIs
```

Example:

```text
100 requests/minute/user
```

Return:

```text
429 Too Many Requests
```

when the policy is exceeded.

---

# 66. Cache vs Database

Use database for:

```text
Durable source of truth
Transactions
Relationships
Integrity
```

Use cache for:

```text
Fast repeated reads
Temporary derived data
Reduced database load
```

Never assume cached data is automatically authoritative.

---

# 67. When Not to Cache

Avoid caching when:

```text
Data changes constantly
Consistency requirement is strict
Cache hit rate is poor
Data is huge
Invalidation is more complex than the benefit
```

Measure before introducing a cache.

---

# 68. Observability Scenario

User says:

> Checkout is slow.

A good investigation:

```text
Trace checkout request
        ↓
Gateway
        ↓
Order Service
        ↓
Inventory
        ↓
Payment
        ↓
Database
```

Find:

```text
Payment = 2.5s
```

Then investigate Payment rather than optimizing unrelated code.

---

# 69. p95 and p99

Average latency can hide slow requests.

Example:

```text
Average = 100ms
p95 = 500ms
p99 = 3s
```

Some users are experiencing much worse latency.

Interview answer:

> I prefer looking at percentiles such as p95 and p99 for production latency because averages can hide tail latency.

---

# 70. Structured Logging

Instead of:

```text
Order failed
```

include useful context:

```text
orderId
customerId
correlationId
operation
duration
result
```

Avoid sensitive data.

---

# 71. Distributed Trace

Example:

```text
Trace ID: 123

Gateway        20ms
Order          80ms
Inventory      40ms
Payment      1500ms
```

The trace immediately shows Payment is the likely bottleneck.

---

# 72. Production Incident Process

A practical sequence:

```text
Detect
↓
Assess impact
↓
Mitigate
↓
Investigate
↓
Fix
↓
Verify
↓
Communicate
↓
Post-incident review
```

Don't focus only on the technical fix.

---

# 73. Root Cause Analysis

Use:

```text
What happened?
Why did it happen?
Why wasn't it detected earlier?
Why did existing controls fail?
How do we prevent recurrence?
```

Possible outputs:

```text
Code fix
Monitoring
Alert
Test
Runbook
Architecture change
```

---

# 74. Monitoring Alert Example

Bad:

```text
CPU > 70%
```

without context.

Better alerts can combine:

```text
Error rate
Latency
Traffic
Resource usage
Business impact
```

For example:

```text
5xx > 5% for 5 minutes
```

depending on the service's normal behavior.

---

# 75. Feature Flag

Feature flags allow controlled rollout:

```text
Feature OFF
↓
Enable for internal users
↓
10%
↓
50%
↓
100%
```

Use them carefully because stale flags create technical debt.

---

# 76. Backward Compatibility

For API evolution:

```text
Add field
```

is usually safer than:

```text
Rename/remove field immediately
```

Consumers may still depend on the old contract.

---

# 77. Event Schema Evolution

If Kafka consumers expect:

```json
{
  "orderId": 101,
  "amount": 999
}
```

don't casually remove:

```text
amount
```

Use compatibility strategies and schema/versioning practices appropriate to the event platform.

---

# 78. Microservice Should Be Independently Deployable

A service should ideally own:

```text
Code
Data
Deployment
Configuration
Operational responsibility
```

It should not require another team's deployment for every small change.

---

# 79. Distributed System Golden Rules

```text
Networks fail
Requests can be delayed
Requests can be duplicated
Services can be unavailable
Data can become temporarily inconsistent
Timeouts are necessary
Retries can cause duplicates
Observability is essential
```

---

# 80. Real-World Scenario: Database Down

Expected behavior:

```text
Request
↓
Database unavailable
↓
Timeout/failure
↓
Log error
↓
Return safe 5xx response
↓
Alert operations
```

Do not:

```text
Retry forever
```

---

# 81. Real-World Scenario: Redis Down

For a non-critical cache:

```text
Redis unavailable
↓
Fallback to DB
```

But protect the database from a cache-miss storm.

Potential controls:

```text
Rate limiting
Request coalescing
Circuit breaker
Fallback
```

---

# 82. Real-World Scenario: Kafka Down

If Kafka is used for non-critical asynchronous work:

```text
Business transaction
↓
Outbox
↓
Kafka unavailable
↓
Event remains in outbox
↓
Retry publisher later
```

This is one benefit of the Outbox Pattern.

---

# 83. Real-World Scenario: One Service Is Slow

Suppose:

```text
Order → Recommendation
```

Recommendation takes:

```text
5 seconds
```

If recommendations are not required to place an order:

```text
Do not block checkout
```

Use asynchronous processing or a timeout/fallback.

---

# 84. Real-World Scenario: Third-Party API Changes

If a payment provider changes its response:

```text
External API
↓
Adapter/client layer
↓
Internal domain model
```

Keep external API details isolated.

This prevents third-party schemas from leaking through the entire application.

---

# 85. Adapter Pattern for External APIs

Example:

```text
PaymentGateway
      ↑
StripePaymentGateway
RazorpayPaymentGateway
```

Business service depends on:

```java
PaymentGateway
```

rather than directly embedding vendor-specific logic everywhere.

---

# 86. Configuration Failure

Application starts with:

```text
DB_URL missing
```

Prefer failing fast for mandatory configuration.

A production application should not silently start with:

```text
null
```

for critical infrastructure settings.

---

# 87. Feature Configuration

For configurable behavior:

```yaml
app:
  checkout:
    enabled: true
```

Use typed configuration where possible:

```java
@ConfigurationProperties
```

rather than scattering strings throughout the codebase.

---

# 88. Production Debugging Example

Problem:

```text
Order API latency increased from 200ms to 2s.
```

Investigation:

```text
Metrics
↓
p95 increased
↓
Trace
↓
Inventory call = 1.5s
↓
Inventory DB query slow
↓
Execution plan
↓
Missing index
```

Fix:

```text
Add appropriate index
↓
Measure again
↓
p95 returns to normal
```

This is a strong interview story because it shows:

```text
Observation
Investigation
Root cause
Fix
Verification
```

---

# 89. Production Bug: Duplicate Orders

Problem:

```text
Users occasionally receive duplicate orders.
```

Investigation:

```text
Retries
↓
No idempotency
↓
POST request processed twice
```

Fix:

```text
Idempotency key
+
Unique business identifier
+
Database constraint
```

Verify:

```text
Repeated request produces one order
```

---

# 90. Production Bug: Overselling

Problem:

```text
Stock = 1
Two users purchase simultaneously
Stock becomes invalid
```

Root cause:

```text
Read-check-write race condition
```

Possible fix:

```text
Atomic update
Optimistic locking
Pessimistic locking
Inventory reservation
```

Choose based on workload and business requirements.

---

# 91. Production Bug: Memory Leak

Problem:

```text
Heap usage continuously increases
```

Investigation:

```text
Heap dump
↓
Large retained collection
↓
Cache has no eviction
```

Fix:

```text
Bound cache
↓
Configure TTL
↓
Evict entries
↓
Monitor memory
```

---

# 92. Production Bug: Slow Startup

Check:

```text
Bean initialization
Database connection
Migrations
External calls during startup
Classpath
Configuration
```

Avoid making application startup depend on unnecessary external services.

---

# 93. Production Bug: Failed Deployment

Check:

```text
Application logs
Health probes
Configuration
Database migration
Environment variables
Secrets
Compatibility
```

If rollout is unhealthy:

```text
Stop rollout
↓
Rollback if needed
↓
Investigate
```

---

# 94. Testing Real-World Failures

Tests should cover:

```text
Database failure
Timeout
Validation failure
Unauthorized access
Forbidden access
Duplicate request
Concurrent update
External API failure
Kafka failure
Cache miss
Cache unavailable
```

Test failure paths, not just happy paths.

---

# 95. Integration Testing

Use integration tests for:

```text
Repository queries
Database constraints
Transactions
Security configuration
HTTP behavior
Serialization
```

Tools:

```text
@SpringBootTest
MockMvc
Testcontainers
```

---

# 96. Testcontainers Scenario

For MySQL:

```text
JUnit test
    ↓
Testcontainers
    ↓
MySQL container
    ↓
Spring Boot
    ↓
Repository
```

This gives more realistic database behavior than mocking the repository.

---

# 97. Concurrency Testing

For inventory:

```text
100 concurrent requests
↓
Stock = 10
```

Verify:

```text
Stock never becomes negative
Successful orders <= available stock
```

Concurrency bugs often don't appear in sequential unit tests.

---

# 98. Load Testing

Tools can simulate:

```text
100 users
1,000 users
10,000 requests
```

Measure:

```text
Throughput
p95
p99
Error rate
CPU
Memory
Database load
```

---

# 99. Capacity Planning

Before scaling:

```text
Current traffic
Peak traffic
Expected growth
DB capacity
CPU
Memory
Network
External dependencies
```

Then estimate required capacity.

---

# 100. Final Real-World Architecture

```text
                         Client
                           |
                           v
                    Load Balancer
                           |
                           v
                    API Gateway
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          User          Product        Order
         Service        Service        Service
             |             |             |
            DB             DB            DB
                                         |
                              +----------+----------+
                              |                     |
                              v                     v
                         Inventory              Payment
                          Service                Service
                              |                     |
                             DB                    DB

                    Asynchronous Events
                           |
                           v
                         Kafka
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
        Notification   Analytics     Other Consumers

Observability:
Logs + Metrics + Traces + Alerts

Reliability:
Timeouts + Retry + Circuit Breaker
Idempotency + Outbox + Saga
```

---

# 101. Scenario Answer Framework

For almost any production question:

```text
1. Understand impact
2. Gather evidence
3. Identify bottleneck/root cause
4. Mitigate immediate impact
5. Implement permanent fix
6. Test the fix
7. Monitor after deployment
8. Prevent recurrence
```

This structure makes answers sound practical rather than theoretical.

---

# 102. Final Interview Rule

> **When answering real-world Spring Boot questions, don't jump directly to a technology. First identify the failure or requirement, collect evidence, explain the root cause, choose the simplest suitable solution, and mention how you would verify and monitor it.**

Next:

```text
18-spring-boot-testing-and-testcontainers.md
```
