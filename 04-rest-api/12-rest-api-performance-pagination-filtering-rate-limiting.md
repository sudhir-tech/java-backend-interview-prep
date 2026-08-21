# REST API — Performance, Pagination, Filtering & Rate Limiting

This file covers practical techniques for designing REST APIs that remain fast and stable as traffic and data volume grow.

Topics:

```text
Pagination
Filtering
Sorting
Search
API performance
Database interaction
Caching
Rate limiting
Timeouts
Compression
Large responses
N+1 queries
Observability
Interview scenarios
```

---

# 1. Why REST API Performance Matters

A working API is not automatically a good API.

For example:

```text
GET /products
```

may work perfectly with:

```text
1,000 products
```

but become problematic with:

```text
10,000,000 products
```

Performance design must consider:

```text
Response time
Database load
Memory usage
Network bandwidth
Concurrent requests
Payload size
Scalability
```

---

# 2. Measure Before Optimizing

Do not optimize based only on assumptions.

Measure:

```text
Average latency
p95 latency
p99 latency
Throughput
Error rate
Database query time
Cache hit ratio
CPU
Memory
Connection pool usage
```

A useful goal is to identify the actual bottleneck first.

---

# 3. Latency Percentiles

Average latency can hide slow requests.

Example:

```text
Average = 100 ms
p95 = 300 ms
p99 = 2 seconds
```

This means a small percentage of requests are much slower.

For production APIs, p95 and p99 are often more useful than average latency.

---

# 4. Throughput

Throughput describes how much work the system handles over time.

Example:

```text
500 requests/second
```

Increasing throughput is useful only if:

```text
Latency remains acceptable
Error rate remains controlled
Dependencies can handle the load
```

---

# 5. Pagination

Never return millions of records in one response.

Bad:

```http
GET /api/products
```

returning:

```text
5,000,000 products
```

This can cause:

```text
High database load
Large memory usage
Large network response
Slow serialization
Client memory pressure
Timeouts
```

Use pagination.

---

# 6. Page-Based Pagination

Example:

```http
GET /api/products?page=0&size=20
```

Response:

```json
{
  "content": [
    {
      "id": 100,
      "name": "Laptop"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 500,
  "totalPages": 25
}
```

This is simple and common.

---

# 7. Spring Data Pageable

Spring Data supports:

```java
@GetMapping
public Page<ProductResponse> getProducts(
        Pageable pageable) {

    return productService.getProducts(pageable);
}
```

Repository:

```java
Page<Product> findAll(Pageable pageable);
```

This allows the database query to retrieve only the requested page.

---

# 8. Page vs Slice

Spring Data provides concepts such as:

```text
Page
Slice
```

`Page` generally includes total-count information.

`Slice` focuses more on whether another slice exists and can avoid an expensive total-count query in some situations.

Use `Page` when the UI needs:

```text
Total pages
Total records
```

Use `Slice` when you mainly need:

```text
Next page?
```

---

# 9. Pagination Parameters

Typical parameters:

```text
page
size
```

Example:

```http
GET /products?page=2&size=20
```

The server should enforce reasonable limits.

For example:

```text
size <= 100
```

Do not allow:

```http
size=1000000
```

without a strong reason.

---

# 10. Default Pagination

If the client does not provide pagination:

```http
GET /products
```

the server can apply defaults:

```text
page = 0
size = 20
```

This prevents accidentally returning the entire dataset.

---

# 11. Maximum Page Size

Example:

```text
Default size = 20
Maximum size = 100
```

If the client requests:

```http
size=1000
```

the API can:

```text
Reject it
```

or:

```text
Cap it at 100
```

The behavior should be documented.

---

# 12. Offset Pagination

Traditional pagination often uses:

```text
LIMIT
OFFSET
```

Example:

```sql
SELECT *
FROM products
ORDER BY id
LIMIT 20 OFFSET 100;
```

This is easy to understand but can become expensive for very large offsets.

---

# 13. Offset Pagination Problem

Suppose:

```text
OFFSET 1,000,000
```

The database may need to scan or skip a large number of rows before returning the requested records.

This can become inefficient for deep pagination.

---

# 14. Cursor Pagination

Cursor pagination uses a position in the dataset.

Example:

```http
GET /products?limit=20&cursor=eyJpZCI6MTAwfQ==
```

Response:

```json
{
  "items": [
    {
      "id": 101,
      "name": "Laptop"
    }
  ],
  "nextCursor": "..."
}
```

The cursor represents where the next query should continue.

---

# 15. Keyset Pagination

A common cursor-style implementation is keyset pagination.

Suppose the previous last ID is:

```text
100
```

Next query:

```sql
SELECT *
FROM products
WHERE id > 100
ORDER BY id
LIMIT 20;
```

This can perform better than large offsets when the ordering column is indexed.

---

# 16. Cursor Pagination Advantages

Useful for:

```text
Large datasets
Infinite scrolling
Feeds
High-volume APIs
Frequently changing data
```

Advantages:

```text
Stable traversal
Efficient deep pagination
Less dependence on large offsets
```

---

# 17. Cursor Pagination Limitations

Cursor pagination can be more complex.

Challenges include:

```text
Opaque cursor design
Sorting requirements
Jumping directly to page 50
Changing sort order
Debugging
```

It is not automatically better for every API.

---

# 18. Stable Ordering

Pagination needs deterministic ordering.

Bad:

```sql
ORDER BY created_at
```

if many records have the same timestamp.

Better:

```sql
ORDER BY created_at DESC, id DESC
```

The unique ID acts as a tie-breaker.

---

# 19. Filtering

Filtering allows clients to request only relevant records.

Example:

```http
GET /products?category=phones
```

Multiple filters:

```http
GET /products?category=phones&brand=Samsung&minPrice=20000
```

---

# 20. Sorting

Example:

```http
GET /products?sort=price,asc
```

or:

```http
GET /products?sort=createdAt,desc
```

Always validate allowed sort fields.

Do not blindly concatenate user-provided values into SQL.

---

# 21. Search

Example:

```http
GET /products?search=laptop
```

For simple search:

```text
Database indexes
LIKE queries
Full-text indexes
```

For large-scale search requirements:

```text
Elasticsearch
OpenSearch
```

may be considered.

---

# 22. Filtering + Pagination

A common API:

```http
GET /products
    ?category=electronics
    &minPrice=10000
    &maxPrice=100000
    &page=0
    &size=20
    &sort=price,asc
```

The server should:

```text
Validate parameters
 ↓
Build query
 ↓
Use indexes where appropriate
 ↓
Return only requested data
```

---

# 23. Dynamic Queries

Spring Data JPA can support dynamic filtering using:

```text
Specifications
Criteria API
Query methods
Custom JPQL
QueryDSL
```

For complex search requirements, use the approach that keeps query behavior maintainable and observable.

---

# 24. Database Indexes

Suppose the API frequently filters by:

```text
category
brand
price
```

Indexes may improve query performance.

Example:

```sql
CREATE INDEX idx_product_category
ON products(category);
```

But indexes also have costs:

```text
Storage
Write overhead
Maintenance
```

Do not create indexes without considering actual query patterns.

---

# 25. Composite Index

Suppose the query commonly uses:

```sql
WHERE category = ?
AND brand = ?
ORDER BY price;
```

A composite index may be useful.

Example:

```sql
CREATE INDEX idx_product_category_brand_price
ON products(category, brand, price);
```

Index design should be based on actual queries and database execution plans.

---

# 26. Query Optimization

Before optimizing a query:

```text
Inspect generated SQL
Run EXPLAIN
Check indexes
Check row counts
Check joins
Check filtering
Check sorting
```

Do not assume the ORM generated the optimal query.

---

# 27. N+1 Query Problem

Suppose:

```text
1 query → fetch 100 orders
```

Then application code performs:

```text
100 queries → fetch customer/order details
```

Total:

```text
101 queries
```

This is the N+1 problem.

---

# 28. Why N+1 Hurts APIs

N+1 can cause:

```text
High database traffic
Higher latency
Connection pool pressure
Poor scalability
```

The API may look simple while generating excessive database work.

---

# 29. Solving N+1

Possible approaches:

```text
Fetch join
EntityGraph
DTO projection
Batch fetching
Explicit optimized queries
```

Choose based on the use case.

---

# 30. DTO Projection

Instead of loading a large entity graph:

```java
SELECT new ProductSummaryDTO(...)
```

or an interface-based projection can retrieve only fields required by the API.

Benefits:

```text
Smaller query result
Less memory
Less serialization
Potentially faster response
```

---

# 31. Avoid Returning Entities Directly

Prefer:

```text
Entity
 ↓
Service
 ↓
DTO
 ↓
Controller
 ↓
JSON
```

rather than:

```text
Entity
 ↓
Controller
 ↓
JSON
```

DTOs provide better control over:

```text
API contract
Payload size
Sensitive fields
Lazy relationships
Versioning
```

---

# 32. Response Payload Size

Large JSON responses increase:

```text
Serialization cost
Network bandwidth
Client processing
Latency
```

Return only what the client needs.

---

# 33. Field Selection

Some APIs support:

```http
GET /products?fields=id,name,price
```

This can reduce payload size for large objects.

However, field selection adds API complexity and should be introduced only when it provides real value.

---

# 34. Compression

HTTP compression can reduce response size.

Common algorithm:

```text
gzip
```

Modern deployments may also use:

```text
Brotli
```

Compression is especially useful for larger text responses.

But compression also consumes CPU, so measure its impact.

---

# 35. HTTP Keep-Alive

Persistent connections reduce the overhead of repeatedly creating network connections.

Modern HTTP clients and servers commonly reuse connections.

This can improve:

```text
Latency
Connection efficiency
Throughput
```

---

# 36. HTTP/2

HTTP/2 provides features such as:

```text
Multiplexing
Header compression
Persistent connections
```

It can improve performance when many resources or requests are exchanged over the same connection.

The exact benefit depends on the architecture and deployment.

---

# 37. Caching

Caching can reduce database work.

Example:

```text
Request
 ↓
Redis
 ↓ hit
Return
```

For cache misses:

```text
Redis miss
 ↓
Database
 ↓
Redis
 ↓
Return
```

See the dedicated caching file for detailed Redis strategies.

---

# 38. Cache Appropriate Data

Good candidates:

```text
Frequently accessed
Expensive to calculate
Relatively stable
Safe to be slightly stale
```

Avoid caching everything.

---

# 39. Rate Limiting

Rate limiting controls how many requests a client can make.

Example:

```text
100 requests/minute/user
```

If the client exceeds the limit:

```http
429 Too Many Requests
```

---

# 40. Why Rate Limit?

Rate limiting protects against:

```text
Abuse
Accidental traffic spikes
Brute-force attacks
Resource exhaustion
Unfair resource usage
Database overload
```

It can also support fair usage between clients.

---

# 41. Rate Limiting Dimensions

You can limit by:

```text
IP address
User ID
API key
Client application
Endpoint
Tenant
Global system
```

The correct dimension depends on the application.

---

# 42. Token Bucket

Token bucket is a common rate-limiting algorithm.

Conceptually:

```text
Bucket
Capacity = 100 tokens

Tokens refill over time
```

Each request consumes a token.

If no token is available:

```text
429 Too Many Requests
```

---

# 43. Leaky Bucket

Leaky bucket controls the rate at which requests are processed.

Conceptually:

```text
Incoming requests
       ↓
     Queue
       ↓
Controlled output rate
```

It can smooth bursts.

---

# 44. Fixed Window

Example:

```text
100 requests
per minute
```

At:

```text
12:00:00
```

counter starts.

At:

```text
12:01:00
```

counter resets.

Simple, but traffic can spike around window boundaries.

---

# 45. Sliding Window

Sliding-window algorithms evaluate requests over a moving time period.

Example:

```text
Last 60 seconds
```

This provides more consistent control than a simple fixed window, although implementation can be more complex.

---

# 46. Redis Rate Limiting

Redis is useful because multiple application instances can share rate-limit state.

Architecture:

```text
Instance A ─┐
Instance B ─┼──> Redis counter/state
Instance C ─┘
```

Without shared state, each instance might independently allow the full request quota.

---

# 47. Rate Limit Headers

An API may communicate limits using headers such as:

```http
RateLimit-Limit: 100
RateLimit-Remaining: 20
RateLimit-Reset: 45
```

Exact header conventions should follow the API standard adopted by the organization.

---

# 48. Retry-After

For a throttled request:

```http
429 Too Many Requests
Retry-After: 30
```

This tells the client when it should try again.

Clients should respect server throttling guidance rather than retrying aggressively.

---

# 49. Client Retry Strategy

Good retry behavior:

```text
Limited retries
Exponential backoff
Jitter
Idempotent operations
Respect Retry-After
```

Bad behavior:

```text
Immediate infinite retries
```

That can create a retry storm.

---

# 50. Exponential Backoff

Example delays:

```text
1 second
2 seconds
4 seconds
8 seconds
```

Add jitter:

```text
randomized delay
```

This prevents many clients from retrying at exactly the same time.

---

# 51. Timeout

Every remote call should have a sensible timeout.

Without a timeout:

```text
Slow dependency
 ↓
Request waits
 ↓
Thread/connection occupied
 ↓
More requests arrive
 ↓
Resource exhaustion
```

Timeouts are a core resilience mechanism.

---

# 52. Connection Pooling

Creating a database connection for every request is expensive.

A connection pool:

```text
Application
    ↓
Connection Pool
    ↓
Database
```

reuses established connections.

In Spring Boot applications using HikariCP, pool configuration should match:

```text
Traffic
Database capacity
Query latency
Application instances
```

---

# 53. HikariCP and API Performance

A pool that is too small can cause:

```text
Requests waiting for connections
```

A pool that is too large can cause:

```text
Too many database connections
Database contention
```

Bigger is not automatically better.

---

# 54. Thread Pool Pressure

If downstream calls are slow:

```text
Requests
 ↓
Threads waiting
 ↓
Thread pool exhausted
```

This can make an otherwise healthy application unavailable.

Monitor:

```text
Active threads
Queue size
Request latency
Rejected tasks
```

---

# 55. Async Processing

Some operations do not need to complete during the HTTP request.

Example:

```text
POST /orders
 ↓
Create order
 ↓
Publish event
 ↓
Return
```

Background work:

```text
Send email
Generate report
Update analytics
```

can happen asynchronously when business semantics allow it.

---

# 56. Don't Make Everything Async

Async processing introduces:

```text
Eventual consistency
Failure handling
Retry logic
Observability complexity
Ordering concerns
```

Use it when the business flow genuinely benefits from asynchronous execution.

---

# 57. Bulk APIs

Suppose a client needs:

```text
100 product updates
```

Calling:

```text
100 HTTP requests
```

may be inefficient.

A bulk endpoint can sometimes reduce network overhead:

```http
POST /api/products/bulk
```

But bulk APIs need:

```text
Validation
Partial failure handling
Idempotency
Payload limits
Transaction semantics
```

---

# 58. Batch Size Limits

Do not allow unlimited bulk requests.

Example:

```text
Maximum = 500 items
```

This protects:

```text
Memory
Database
Request processing
Network
```

The correct limit depends on the workload.

---

# 59. Large File Uploads

Avoid loading huge files completely into memory.

Prefer:

```text
Streaming
Multipart upload
Object storage
Pre-signed URLs
```

For example:

```text
Client
 ↓
Object Storage
 ↓
Spring Boot receives metadata
```

This keeps application servers from becoming file-transfer bottlenecks.

---

# 60. Streaming Responses

For large datasets or files, streaming can reduce memory pressure.

Instead of:

```text
Load everything
 ↓
Serialize everything
 ↓
Send
```

stream data incrementally when appropriate.

---

# 61. Database-Level Pagination

Bad:

```text
Fetch all records
 ↓
Java filters first 20
```

Better:

```text
Request first 20
 ↓
Database returns 20
```

Let the database perform filtering and pagination whenever possible.

---

# 62. Avoid In-Memory Filtering

Bad:

```java
repository.findAll()
    .stream()
    .filter(...)
    .limit(20);
```

for a large table.

Better:

```text
Database WHERE
Database LIMIT
Database ORDER BY
```

This reduces:

```text
Network transfer
Application memory
CPU
```

---

# 63. API Gateway Rate Limiting

In microservices:

```text
Client
 ↓
API Gateway
 ↓
Services
```

Gateway-level rate limiting can protect the entire platform before traffic reaches individual services.

Service-level limits may still be needed for sensitive operations.

---

# 64. Endpoint-Specific Limits

Not every endpoint needs the same limit.

Example:

```text
GET /products
1000/minute

POST /login
10/minute

POST /payments
30/minute
```

Authentication and payment endpoints often need stricter controls.

---

# 65. Avoid Information Leakage

Rate-limit responses should not reveal unnecessary internal information.

Avoid exposing:

```text
Redis topology
Internal server details
Database information
Stack traces
```

Return a clean API error.

---

# 66. API Error for Rate Limit

Example:

```json
{
  "status": 429,
  "error": "Too Many Requests",
  "message": "Rate limit exceeded",
  "timestamp": "2026-08-21T10:00:00Z"
}
```

Keep error formats consistent across the API.

---

# 67. Performance Optimization Order

A practical order:

```text
1. Measure
2. Identify bottleneck
3. Optimize database queries
4. Add appropriate indexes
5. Reduce payload
6. Add pagination
7. Add caching
8. Optimize remote calls
9. Add rate limiting
10. Load test
11. Monitor production
```

Do not jump directly to caching when the actual problem is an inefficient query.

---

# 68. Load Testing

Tools can simulate traffic:

```text
JMeter
k6
Gatling
```

Measure:

```text
Requests/sec
Latency
p95
p99
Errors
CPU
Memory
Database load
```

Test realistic traffic patterns.

---

# 69. Stress Testing

Stress testing pushes the system beyond normal expected capacity.

Goal:

```text
Find breaking point
```

Observe:

```text
When latency increases
When errors appear
Which dependency fails first
How recovery behaves
```

---

# 70. Capacity Planning

Suppose:

```text
Current traffic = 500 RPS
Expected traffic = 2,000 RPS
```

You need to evaluate:

```text
Application instances
Database capacity
Redis capacity
Connection pools
CPU
Memory
Network
External dependencies
```

Scaling only the Spring Boot instances may not solve a database bottleneck.

---

# 71. Horizontal Scaling

Instead of:

```text
One huge application
```

use:

```text
Load Balancer
    |
+---+---+
|   |   |
A   B   C
```

Stateless APIs are easier to scale horizontally.

---

# 72. Stateless REST APIs

A stateless service does not rely on local server memory to remember client request state between requests.

This makes:

```text
Load balancing
Horizontal scaling
Failover
```

easier.

JWT-based authentication can support stateless request authentication, although other state such as refresh tokens or revocation lists may still exist.

---

# 73. Database Bottleneck

Suppose:

```text
API instances ↑
```

but:

```text
Database CPU = 100%
```

Adding more application instances may make the problem worse.

Investigate:

```text
Slow queries
Indexes
N+1
Connection pool
Caching
Read replicas
Database scaling
```

---

# 74. Read Replicas

For read-heavy workloads:

```text
Primary DB
   |
   +---- Replica
   |
   +---- Replica
```

Read traffic can potentially be distributed to replicas.

But replication lag means reads may not always immediately reflect the latest write.

Do not use replicas blindly for read-after-write requirements.

---

# 75. Cache + Read Replica

A scalable read path might be:

```text
Client
 ↓
Spring Boot
 ↓
Redis
 ↓ miss
Read Replica
```

Writes:

```text
Spring Boot
 ↓
Primary DB
```

This is useful only when the consistency model supports it.

---

# 76. API Performance Checklist

```text
□ Pagination
□ Maximum page size
□ Stable sorting
□ Filtering
□ Database indexes
□ Query optimization
□ N+1 prevention
□ DTO projections
□ Small payloads
□ Compression
□ Caching
□ Connection pooling
□ Timeouts
□ Rate limiting
□ Retry with backoff
□ Load testing
□ Metrics
□ Distributed tracing
```

---

# 77. Interview: How Would You Improve a Slow REST API?

> I would first measure where the latency comes from instead of immediately adding caching. I would inspect database queries, indexes, N+1 issues, downstream calls, connection pools and payload size. Then I would apply pagination, query optimization, caching or other changes based on the actual bottleneck and validate the improvement with load testing.

---

# 78. Interview: Why Is Pagination Important?

> Pagination prevents the API from loading and returning an unnecessarily large dataset. It reduces database work, memory usage, serialization cost and network bandwidth. For very large datasets, I would consider cursor or keyset pagination instead of large offsets.

---

# 79. Interview: Offset vs Cursor Pagination?

> Offset pagination is simple and useful for normal page-based interfaces, but deep offsets can become expensive on large datasets. Cursor or keyset pagination uses a stable position, such as an indexed ID, and is usually better for large datasets and infinite-scroll style APIs.

---

# 80. Interview: What Is the N+1 Problem?

> N+1 occurs when the application executes one query to load a collection and then executes additional queries for each item. For example, one query loads 100 orders and another 100 queries load their customers. I would investigate fetch joins, entity graphs, DTO projections or batch fetching.

---

# 81. Interview: How Do You Prevent Large API Responses?

> I use pagination, DTOs, field selection where justified, compression and streaming for appropriate workloads. I also enforce maximum page sizes so a client cannot accidentally request an extremely large response.

---

# 82. Interview: What Is Rate Limiting?

> Rate limiting controls how many requests a client can make during a period. It protects the application from abuse, traffic spikes and resource exhaustion. In a distributed Spring Boot system, Redis can maintain shared rate-limit state across application instances.

---

# 83. Interview: What Happens When a Client Exceeds the Rate Limit?

> I normally return `429 Too Many Requests` and, when appropriate, provide retry information such as `Retry-After`. The client should use bounded retries with exponential backoff and jitter rather than immediately sending more requests.

---

# 84. Interview: How Would You Design a High-Traffic Product API?

> I would keep the service stateless, use pagination and indexed database queries, cache frequently accessed product details in Redis, return DTOs with only required fields, configure sensible connection and request timeouts, add rate limiting, and monitor p95/p99 latency, database load, cache performance and error rates.

---

# 85. Interview: How Do You Handle a Million Products?

> I would never load all products into memory. I would expose paginated or cursor-based APIs, push filtering and sorting to the database, use appropriate indexes, return compact DTOs, and consider caching popular or relatively stable data.

---

# 86. Interview: How Do You Protect the Database During Traffic Spikes?

> I would use rate limiting, pagination, caching, connection-pool limits and timeouts. For expensive hot-key reads, I would use cache-stampede protection. I would also monitor database CPU, connection usage and query latency so the application does not simply push uncontrolled traffic into the database.

---

# 87. Interview: Why Is a Large Connection Pool Bad?

> A large pool can create too many concurrent database operations and overwhelm the database. The pool should be sized according to database capacity, query latency, application concurrency and the number of application instances. More connections do not automatically mean more throughput.

---

# 88. Interview: Average Latency vs p99?

> Average latency gives an overall mean, but it can hide slow requests. p99 tells me how slow the worst roughly one percent of requests are. For production APIs, I usually monitor p95 and p99 along with average latency because tail latency directly affects user experience.

---

# 89. Interview: When Would You Use Cursor Pagination?

> I would consider cursor or keyset pagination for large datasets, feeds and infinite scrolling where deep pagination is expected. It can avoid the performance problems associated with very large offsets, provided the API has a stable and indexed ordering strategy.

---

# 90. Final Mental Model

```text
                    REST API
                       |
       +---------------+---------------+
       |               |               |
   Request size     Database        Dependencies
       |               |               |
   Pagination      Indexes          Timeouts
   Rate limit      Query tuning     Retries
   Compression     N+1 prevention   Backoff
       |               |               |
       +---------------+---------------+
                       |
                    Caching
                       |
                    Redis
                       |
                  Observability
                       |
              p95 / p99 / errors
```

For a high-traffic ecommerce API:

```text
Client
  ↓
Load Balancer
  ↓
Spring Boot
  |
  +── Rate Limiting
  |
  +── Redis Cache
  |
  +── Pagination / Filtering
  |
  +── Optimized DTO Query
  |
  +── MySQL
  |
  +── Async Events
```

> **The fastest API is not created by one magic optimization. Good REST performance comes from controlling the amount of data processed, making database queries efficient, protecting dependencies, caching the right data, limiting abusive traffic, and measuring the system continuously.**
