# Spring Boot — Performance and Optimization

This file covers practical Spring Boot performance topics for backend interviews and real-world applications.

The main principle is:

> Measure first, identify the bottleneck, then optimize.

---

# 1. What Is Application Performance?

Important dimensions include:

```text
Latency
Throughput
CPU usage
Memory usage
Database performance
Network usage
Error rate
Resource utilization
```

Example:

```text
Latency = time for one request
Throughput = requests processed per second
```

---

# 2. How Do You Find a Performance Bottleneck?

Use:

```text
Metrics
Logs
Distributed tracing
Profiling
Database execution plans
Thread dumps
JVM metrics
```

Typical flow:

```text
User reports slow API
        ↓
Check p95/p99 latency
        ↓
Trace request
        ↓
Find slow component
        ↓
Measure root cause
        ↓
Optimize
        ↓
Measure again
```

---

# 3. Average Latency vs p95

Suppose:

```text
Average = 100 ms
p95 = 500 ms
p99 = 2 seconds
```

Average alone hides the slow requests.

Interview answer:

> I prefer looking at p95 and p99 for production APIs because they show tail latency and help identify the experience of slower users.

---

# 4. Throughput

Throughput represents how much work the system processes during a period.

Examples:

```text
500 requests/second
10,000 orders/minute
```

Increasing throughput isn't always useful if:

```text
Latency becomes unacceptable
Database becomes overloaded
Error rate increases
```

---

# 5. Horizontal vs Vertical Scaling

Horizontal:

```text
1 instance
   ↓
3 instances
```

Vertical:

```text
4 CPU / 8 GB
      ↓
8 CPU / 16 GB
```

Horizontal scaling is often preferred for stateless web applications because it provides more capacity and resilience.

---

# 6. Stateless Spring Boot Application

A stateless service does not keep request-specific session state only in local memory.

Example:

```text
Instance A
Instance B
Instance C
```

Any instance can process the request.

This makes load balancing and horizontal scaling easier.

---

# 7. Connection Pooling

Creating a database connection for every request is expensive.

Instead:

```text
Application
    ↓
Connection Pool
    ↓
Database Connections
```

Spring Boot commonly uses HikariCP.

Benefits:

```text
Reuse connections
Lower connection overhead
Control concurrency
```

---

# 8. HikariCP

Important settings include:

```text
maximumPoolSize
minimumIdle
connectionTimeout
idleTimeout
maxLifetime
```

Don't tune them blindly.

A larger pool is not always faster.

---

# 9. Connection Pool Sizing

If:

```text
Application = 20 instances
Pool = 50 connections each
```

potential database connections:

```text
20 × 50 = 1000
```

The database may not be able to handle that.

Think about:

```text
Database capacity
Number of application instances
Query duration
Concurrency
CPU
```

---

# 10. Slow Database Queries

Investigate:

```text
Execution plan
Indexes
Joins
Filters
Returned columns
Row count
Locking
Query frequency
```

Example:

```sql
EXPLAIN
SELECT *
FROM products
WHERE sku = 'ABC';
```

---

# 11. Database Indexes

Indexes can make reads faster:

```sql
CREATE INDEX idx_product_sku
ON products(sku);
```

But indexes also have costs:

```text
Storage
Insert/update overhead
Maintenance
```

Only add indexes based on actual access patterns.

---

# 12. Composite Index

Query:

```sql
SELECT *
FROM orders
WHERE customer_id = ?
  AND status = ?;
```

A composite index may help:

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

Index order matters.

---

# 13. Select Only Required Columns

Instead of:

```sql
SELECT *
FROM products;
```

consider:

```sql
SELECT id, name, price
FROM products;
```

Benefits:

```text
Less DB I/O
Less network transfer
Less memory
Less serialization
```

DTO projections can help with this.

---

# 14. N+1 Query Problem

Example:

```text
1 query → orders
N queries → customer/order details
```

Total:

```text
1 + N
```

For 100 orders:

```text
101 queries
```

Possible solutions:

```text
Fetch join
EntityGraph
DTO projection
Batch fetching
Purpose-built queries
```

---

# 15. Fetch Join

Example JPQL:

```java
@Query("""
    select o
    from Order o
    join fetch o.items
    where o.id = :id
""")
Optional<Order> findOrderWithItems(
    @Param("id") Long id
);
```

Use carefully because joining collection relationships can increase result size.

---

# 16. EntityGraph

Example:

```java
@EntityGraph(attributePaths = {"items"})
Optional<Order> findById(Long id);
```

This can control fetching for a specific query without globally changing the association's default fetch behavior.

---

# 17. Pagination

Never load massive datasets unnecessarily.

Instead:

```text
page = 0
size = 20
```

Example:

```java
Page<Product> result =
    repository.findAll(
        PageRequest.of(0, 20)
    );
```

---

# 18. Offset vs Cursor Pagination

Offset:

```text
?page=1000&size=20
```

can become expensive for large datasets.

Cursor/keyset:

```text
?after=product_1000&limit=20
```

can perform better for large, ordered datasets.

Choose based on the access pattern and API requirements.

---

# 19. Bulk Operations

Bad:

```text
Loop 10,000 records
↓
save()
save()
save()
...
```

Possible improvements:

```text
Batch writes
Bulk update
JDBC batching
Repository batch operations
```

Always verify transaction size and memory usage.

---

# 20. Hibernate Batch Processing

For large writes, configure batching appropriately.

Conceptually:

```properties
hibernate.jdbc.batch_size=50
```

Then group compatible SQL operations into batches.

Don't assume batching works equally for every ID-generation strategy and operation.

---

# 21. Transaction Performance

Avoid:

```text
BEGIN
↓
Database work
↓
External API
↓
Sleep/wait
↓
More DB work
↓
COMMIT
```

A transaction holding a database connection while waiting on an external service can reduce concurrency.

Prefer short local transactions.

---

# 22. Lazy Loading

Lazy loading can reduce unnecessary initial database work.

But it can also cause:

```text
LazyInitializationException
N+1 queries
Unexpected DB calls
```

Use explicit fetch strategies for important use cases.

---

# 23. Open Session in View

Open Session in View can allow lazy loading during web response processing.

However, it can hide database access in the presentation layer and make query behavior harder to reason about.

For API applications, many teams prefer explicit fetching in service/repository boundaries.

---

# 24. Caching

Caching is useful for:

```text
Frequently read data
Slow computations
Reference data
Read-heavy endpoints
```

Architecture:

```text
Request
  ↓
Cache
  ↓ miss
Database
  ↓
Cache
```

---

# 25. Cache Hit Ratio

If:

```text
1000 requests
900 cache hits
100 cache misses
```

Hit ratio:

```text
90%
```

A low hit ratio may mean:

```text
Poor cache key
Data changes frequently
TTL too short
Workload isn't cache-friendly
```

---

# 26. Cache-Aside Pattern

Typical flow:

```text
Read
 ↓
Cache?
 ├─ hit → return
 └─ miss
       ↓
     Database
       ↓
     Cache
       ↓
     return
```

This is a common application-level caching pattern.

---

# 27. Cache Invalidation

When data changes:

```text
Database updated
      ↓
Invalidate/update cache
```

The difficult part of caching is often not reading data but keeping cached data consistent enough for the business requirement.

---

# 28. TTL

TTL means:

```text
Time To Live
```

Example:

```text
Cache entry
↓
TTL = 10 minutes
↓
Expires
```

TTL prevents stale entries from remaining forever.

---

# 29. Cache Stampede

Suppose a popular key expires:

```text
10,000 requests
      ↓
Cache miss
      ↓
10,000 DB requests
```

Possible protections:

```text
Request coalescing
Jittered TTL
Distributed lock
Background refresh
Pre-warming
```

---

# 30. Cache Eviction

Common strategies:

```text
TTL
LRU
LFU
Explicit invalidation
Size-based eviction
```

The right strategy depends on access patterns.

---

# 31. Redis Performance

When using Redis:

```text
Keep values reasonably sized
Avoid huge keys/values
Use appropriate data structures
Monitor latency
Monitor memory
Use TTL where appropriate
```

Avoid turning Redis into a dumping ground for unlimited data.

---

# 32. Serialization Performance

Large JSON objects can increase:

```text
CPU
Memory
Network bandwidth
Response time
```

Use DTOs to return only required data.

---

# 33. Compression

HTTP compression can reduce network transfer size.

Useful for:

```text
Large JSON responses
Text payloads
```

Tradeoff:

```text
Less network bandwidth
More CPU for compression/decompression
```

Use it based on workload.

---

# 34. HTTP Keep-Alive

Reusing connections avoids repeatedly creating TCP/TLS connections.

For service-to-service communication, configure HTTP clients with appropriate connection pooling and timeouts.

---

# 35. HTTP Client Connection Pooling

For high-throughput service calls:

```text
Order Service
      ↓
HTTP connection pool
      ↓
Payment Service
```

This avoids creating a new connection for every request.

---

# 36. Always Configure Timeouts

External calls should have limits:

```text
Connect timeout
Read/response timeout
Connection acquisition timeout
```

Never let a dependency hang indefinitely.

---

# 37. Retry Can Hurt Performance

Suppose:

```text
1000 requests
↓
Dependency fails
↓
Each retries 3 times
```

Potential downstream traffic:

```text
3000+ requests
```

This can make an outage worse.

Use:

```text
Limited retries
Backoff
Jitter
Circuit breaker
Idempotency
```

---

# 38. Circuit Breaker

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

This protects resources.

---

# 39. Bulkhead Pattern

Bulkheads isolate resources.

Example:

```text
Payment calls → Pool A
Recommendation calls → Pool B
```

If recommendations become slow, they don't necessarily consume every thread available for payments.

---

# 40. Async Processing

Not every operation needs to happen synchronously.

Example:

```text
Place order
   ↓
Return response
   ↓
Publish OrderCreated
   ↓
Email service
Notification service
Analytics
```

This reduces request latency for non-critical side effects.

---

# 41. Async Doesn't Mean Faster Automatically

Asynchronous processing introduces:

```text
Eventual consistency
Retries
Ordering
Duplicate events
Monitoring complexity
Failure handling
```

Use it when the business workflow supports asynchronous behavior.

---

# 42. Thread Pools

A thread pool prevents unlimited thread creation.

Important concepts:

```text
Core pool
Maximum pool
Queue capacity
Rejection policy
```

A queue that is too large can hide overload and increase latency.

---

# 43. Thread Pool Exhaustion

Symptoms:

```text
Requests waiting
Timeouts
High latency
Low CPU sometimes
```

Possible causes:

```text
Slow DB
Slow HTTP calls
Blocking I/O
Deadlock
Too much concurrency
```

Use thread dumps and metrics.

---

# 44. Java Virtual Machine Performance

Monitor:

```text
Heap
GC
Threads
CPU
Class loading
Native memory where relevant
```

Don't tune JVM flags without evidence.

---

# 45. Garbage Collection

GC removes objects that are no longer reachable.

Too much GC can cause:

```text
CPU overhead
Pause time
Reduced throughput
Latency spikes
```

Investigate allocation patterns before changing collectors or heap sizes.

---

# 46. Heap Size

Increasing heap can:

```text
Reduce some allocation pressure
Delay OOM
```

but may also:

```text
Increase GC work
Hide memory leaks
Increase memory requirements
```

Tune based on measurements.

---

# 47. Object Allocation

Excessive temporary objects can increase GC pressure.

Potential sources:

```text
Large DTO transformations
String creation
Large collections
Repeated serialization
Unnecessary copying
```

Don't optimize allocations prematurely.

---

# 48. String Concatenation

For repeated concatenation in loops, consider:

```java
StringBuilder
```

rather than creating many intermediate strings.

Modern Java optimizes many ordinary concatenation cases, so profile before making micro-optimizations.

---

# 49. Stream Performance

Streams can improve readability, but they are not automatically faster.

Example:

```java
products.stream()
    .filter(...)
    .map(...)
    .toList();
```

Choose based on:

```text
Readability
Correctness
Workload
Profiling
```

---

# 50. Parallel Streams

Do not use:

```java
parallelStream()
```

as a generic performance solution.

Potential problems:

```text
Common ForkJoinPool contention
Database calls in parallel
Thread overhead
Unpredictable performance
```

Use only when the workload is suitable and measured.

---

# 51. Synchronization

Excessive locking can reduce throughput.

Instead of:

```text
One global lock
```

consider:

```text
Smaller critical sections
Concurrent collections
Atomic operations
Database atomic updates
Partitioned state
```

depending on the problem.

---

# 52. Database Lock Contention

Symptoms:

```text
Queries waiting
Transaction latency
Deadlocks
Reduced throughput
```

Investigate:

```text
Lock duration
Transaction length
Access order
Indexes
Isolation level
Hot rows
```

---

# 53. Deadlocks

Example:

```text
Transaction A:
locks Row 1
waits for Row 2

Transaction B:
locks Row 2
waits for Row 1
```

Result:

```text
Deadlock
```

Solutions may include:

```text
Consistent lock ordering
Shorter transactions
Better indexing
Smaller transaction scope
Retrying deadlock victims when safe
```

---

# 54. Database Connection vs Thread Pool

These are related but different:

```text
HTTP threads
     ↓
Database connection pool
     ↓
Database
```

If you have:

```text
200 request threads
20 DB connections
```

many requests may wait for database connections.

Increasing DB connections without considering database capacity can make things worse.

---

# 55. Database Read Replicas

For read-heavy workloads:

```text
Application
   |
   +---- Write → Primary
   |
   +---- Read  → Replica
```

Benefits:

```text
Read scaling
Reduced primary load
```

Tradeoffs:

```text
Replication lag
Routing complexity
Consistency concerns
```

---

# 56. CQRS

CQRS means:

```text
Command Query Responsibility Segregation
```

Conceptually:

```text
Writes → Command model
Reads  → Query model
```

Useful when read and write workloads have significantly different requirements.

Don't introduce it without a real need.

---

# 57. Database Denormalization

Sometimes data can be duplicated intentionally to optimize reads.

Tradeoff:

```text
Faster reads
     vs
More complex writes
Consistency maintenance
```

Use measurements and clear business requirements.

---

# 58. API Response Optimization

Reduce:

```text
Payload size
Unnecessary fields
Nested objects
Repeated data
```

Use:

```text
DTOs
Pagination
Filtering
Compression
Caching
```

---

# 59. Lazy vs Eager Performance

Eager loading can create:

```text
Large joins
Large object graphs
Unnecessary data
```

Lazy loading can create:

```text
N+1
Unexpected queries
```

The correct approach is usually:

> Fetch exactly what the use case needs.

---

# 60. Entity vs DTO Performance

DTO projections can reduce:

```text
Entity creation
Unnecessary columns
Object graph loading
Serialization overhead
```

Example:

```java
@Query("""
    select new com.example.ProductSummary(
        p.id,
        p.name,
        p.price
    )
    from Product p
""")
List<ProductSummary> findSummaries();
```

---

# 61. Batch vs Bulk Update

Batch:

```text
Multiple parameterized operations
```

Bulk:

```text
One database statement affecting many rows
```

Bulk updates can be faster but may bypass some normal entity lifecycle behavior and leave managed entities stale.

Understand JPA persistence-context implications.

---

# 62. Database Pagination With Sorting

Always define deterministic ordering when pagination requires stable results.

Example:

```sql
ORDER BY created_at DESC, id DESC
```

The second field can provide a deterministic tie-breaker.

---

# 63. Cursor Pagination

A common keyset approach:

```sql
WHERE id < ?
ORDER BY id DESC
LIMIT 20;
```

This can be more efficient than large offsets for suitable indexed access patterns.

---

# 64. API Caching Headers

HTTP caching can use:

```text
Cache-Control
ETag
Last-Modified
```

For appropriate public or cacheable resources.

This can reduce repeated server processing.

---

# 65. ETag

Conceptually:

```text
Client
 ↓
GET resource
 ↓
ETag: ABC
```

Next request:

```text
If-None-Match: ABC
```

If unchanged:

```text
304 Not Modified
```

This can reduce response payload transfer.

---

# 66. Compression Tradeoff

Compression:

```text
Payload ↓
Network usage ↓
CPU ↑
```

For very small payloads, compression may provide little benefit.

---

# 67. Performance Testing

Useful tests:

```text
Load testing
Stress testing
Spike testing
Soak testing
Capacity testing
```

Measure:

```text
Throughput
p95
p99
Error rate
CPU
Memory
DB
```

---

# 68. Load Test Example

Suppose target:

```text
1,000 requests/sec
```

Test:

```text
100
200
500
1000
1500
```

Observe where:

```text
Latency rises
Errors begin
Database saturates
CPU saturates
```

---

# 69. Stress Testing

Increase load beyond expected capacity.

Goal:

```text
Find breaking point
```

Also verify:

```text
Recovery behavior
```

after traffic returns to normal.

---

# 70. Soak Testing

Run sustained traffic for a long time.

Useful for detecting:

```text
Memory leaks
Connection leaks
Resource exhaustion
Gradual performance degradation
```

---

# 71. Performance Regression

After an optimization, compare:

```text
Before
vs
After
```

Example:

```text
p95:
800ms → 250ms

DB calls/request:
12 → 3

CPU:
70% → 45%
```

Use measurements rather than subjective claims.

---

# 72. Observability for Performance

Monitor:

```text
Request rate
Error rate
Latency
Saturation
```

Also:

```text
DB query latency
Connection pool
Cache hit ratio
Kafka lag
Thread pools
GC
External API latency
```

---

# 73. RED Method

For services:

```text
Rate
Errors
Duration
```

Example:

```text
Requests/sec
5xx rate
p95 latency
```

---

# 74. USE Method

For infrastructure resources:

```text
Utilization
Saturation
Errors
```

Example:

```text
CPU utilization
Thread saturation
Disk errors
```

---

# 75. Performance Optimization Priority

A useful order:

```text
1. Measure
2. Fix obvious algorithmic problems
3. Optimize database
4. Reduce unnecessary network calls
5. Add caching where justified
6. Tune resource pools
7. Optimize JVM/code
```

Avoid micro-optimizing before fixing architectural bottlenecks.

---

# 76. Common Performance Anti-Patterns

Avoid:

```text
N+1 queries
SELECT *
Huge API responses
Unbounded caches
No timeouts
Unlimited retries
Huge DB transactions
Too many DB connections
Unbounded queues
Synchronous non-critical work
Repeated external calls
```

---

# 77. Performance Scenario: Slow Product API

Problem:

```text
GET /products
p95 = 2 seconds
```

Investigation:

```text
Trace
↓
DB = 1.7 sec
↓
Query scans millions of rows
↓
Missing index
```

Fix:

```text
Add appropriate index
↓
Verify execution plan
↓
Load test
↓
Monitor
```

---

# 78. Performance Scenario: Slow Order API

Trace:

```text
Order API = 3 sec

Inventory = 100ms
Payment = 2.5 sec
Database = 200ms
```

If payment is required synchronously:

```text
Optimize payment client
Timeout
Connection pooling
Retry carefully
```

If payment can be asynchronous:

```text
Create order/pending state
↓
Publish event
↓
Process payment
```

depending on business requirements.

---

# 79. Performance Scenario: Database CPU High

Check:

```text
Top queries
Execution plans
Indexes
Connection count
Long transactions
Read/write mix
```

Don't simply increase database CPU before understanding the workload.

---

# 80. Performance Scenario: CPU High After Deployment

Compare:

```text
Before deployment
vs
After deployment
```

Check:

```text
New code path
Loop
Serialization
Logging
GC
Concurrency
```

Rollback may be the fastest mitigation if impact is severe.

---

# 81. Performance Scenario: Memory Slowly Increases

Pattern:

```text
10 AM → 40%
12 PM → 55%
2 PM  → 70%
4 PM  → 85%
```

Likely requires investigation into:

```text
Leak
Cache
Unbounded collection
Thread growth
Large retained objects
```

Use heap analysis.

---

# 82. Performance Scenario: Cache Helps but DB Still Overloaded

Possible reasons:

```text
Low cache hit ratio
Poor key design
TTL too short
Many unique requests
Cache invalidation too aggressive
Cold cache
```

Measure hit/miss patterns.

---

# 83. Performance Scenario: Retry Storm

Dependency fails:

```text
1000 requests
↓
Each retries 3 times
↓
3000+ calls
```

Dependency becomes even less healthy.

Use:

```text
Exponential backoff
Jitter
Circuit breaker
Retry budget
Idempotency
```

---

# 84. Performance Scenario: Thread Pool Saturation

Symptoms:

```text
Active threads ≈ maximum
Queue grows
Latency increases
```

Find what threads are waiting on:

```text
Database
HTTP
Locks
Disk
Other blocking operations
```

Don't simply increase the thread pool.

---

# 85. Performance Scenario: Connection Pool Saturation

Symptoms:

```text
Active connections = maximum
Pending threads > 0
```

Investigate:

```text
Slow SQL
Long transactions
Connection leak
Unexpected concurrency
Database capacity
```

---

# 86. Performance Scenario: Large Product Catalog

Requirements:

```text
Millions of products
Frequent searches
Pagination
Filtering
Sorting
```

Possible design:

```text
Database
+
Indexes
+
Pagination
+
Caching
```

If search requirements become complex:

```text
Search engine
```

may be appropriate.

Don't introduce Elasticsearch solely because the dataset is large; introduce it when search requirements justify it.

---

# 87. Performance Scenario: High Traffic Checkout

Potential architecture:

```text
Load Balancer
       ↓
Multiple Order Instances
       ↓
Database
       +
Cache
       +
Message Broker
```

Protect critical dependencies with:

```text
Timeouts
Rate limiting
Circuit breakers
Connection limits
```

---

# 88. Performance Scenario: Traffic Spike

Example:

```text
Normal = 100 req/sec
Spike = 5,000 req/sec
```

Possible protections:

```text
Autoscaling
Caching
Queueing
Rate limiting
Load shedding
CDN where applicable
```

Critical business traffic should be prioritized.

---

# 89. Load Shedding

When the system is overloaded:

```text
Accept everything
→
System collapses
```

Better:

```text
Protect critical operations
↓
Reject/defer lower-priority work
↓
Return 429/503 where appropriate
```

---

# 90. Backpressure

Backpressure prevents producers from overwhelming consumers.

Example:

```text
Producer
  ↓
Queue
  ↓
Consumer
```

If consumer capacity is lower:

```text
Queue grows
```

The system needs:

```text
Rate control
Bounded queues
Scaling
Load shedding
```

---

# 91. Performance and Reliability

Performance isn't just speed.

A fast system that frequently fails is not useful.

Balance:

```text
Latency
Throughput
Availability
Consistency
Cost
Reliability
```

---

# 92. Cost Optimization

Performance optimization should also consider cost.

Example:

```text
10 servers → 5 servers
```

if caching and query optimization reduce workload.

But don't reduce resources below safe capacity.

---

# 93. JVM Profiling

Profiling can identify:

```text
CPU hotspots
Allocation hotspots
Thread contention
Lock contention
```

Tools can include:

```text
Java Flight Recorder
Java Mission Control
Async-profiler
APM tools
```

Use profiling in an environment appropriate for the workload and operational constraints.

---

# 94. Thread Dump

A thread dump helps investigate:

```text
Deadlocks
Blocked threads
Thread pool saturation
Long-running operations
```

Look for states such as:

```text
BLOCKED
WAITING
TIMED_WAITING
RUNNABLE
```

---

# 95. Database Execution Plan

Use:

```sql
EXPLAIN
```

to understand:

```text
Index usage
Join strategy
Rows examined
Estimated cost
Scan behavior
```

For critical queries, compare execution plans before and after changes.

---

# 96. Slow Query Logging

Enable appropriate database/application monitoring to identify queries that exceed acceptable thresholds.

Don't leave extremely verbose SQL logging enabled in production without considering:

```text
Performance
Log volume
Sensitive data
Storage cost
```

---

# 97. Performance and Logging

Logging too much can itself become a bottleneck.

Avoid:

```text
Huge objects
Entire request/response bodies
Sensitive data
Repeated logs inside hot loops
```

Use appropriate log levels.

---

# 98. Performance and Serialization

If a response contains:

```text
1000 nested objects
```

serialization can become expensive.

Use:

```text
DTO
Pagination
Projection
Smaller payload
```

Measure serialization time if it becomes a bottleneck.

---

# 99. Performance and Security

Security checks also consume resources.

Don't disable security for performance.

Instead:

```text
Cache appropriate public data
Optimize authorization queries
Use efficient token validation
Avoid unnecessary database calls
```

---

# 100. Final Performance Interview Answer

Question:

> How do you optimize a Spring Boot application?

Strong answer:

> I don't start by changing configuration randomly. I first identify the bottleneck using metrics, traces, logs, profiling, or database execution plans. Then I optimize the relevant layer, whether that's SQL, connection pooling, caching, network calls, concurrency, or application code. Finally, I load-test or measure the change and monitor it after deployment.

---

# 101. Final Performance Checklist

```text
□ Measure p95/p99
□ Check throughput
□ Check error rate
□ Analyze slow queries
□ Review indexes
□ Avoid N+1
□ Use pagination
□ Tune connection pool carefully
□ Keep transactions short
□ Configure HTTP timeouts
□ Retry carefully
□ Use circuit breakers
□ Use caching where justified
□ Monitor cache hit ratio
□ Control thread pools
□ Avoid unbounded queues
□ Check JVM/GC
□ Profile CPU/memory issues
□ Test under load
□ Monitor after deployment
```

---

# 102. Final Mental Model

```text
              PERFORMANCE
                    |
       +------------+------------+
       |            |            |
    Database     Application    Network
       |            |            |
   Indexes       JVM/CPU      HTTP pool
   Queries       Threads      Timeouts
   Pooling       Memory       Payload
   Caching       GC           Retry
       |
       +------------+
                    |
               Observability
                    |
          Measure → Optimize
                    |
                 Verify
```

---

# 103. Final Rule

> **Never optimize based only on intuition. Measure the bottleneck, make one meaningful change, measure again, and verify that the improvement didn't create a new problem somewhere else.**
