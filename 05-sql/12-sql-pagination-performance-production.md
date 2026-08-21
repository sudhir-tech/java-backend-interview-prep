# SQL — Pagination, Performance & Production Patterns

This file focuses on practical SQL performance topics that commonly appear in Java backend interviews and real production systems.

You should understand:

```text
Pagination
LIMIT / OFFSET
Keyset pagination
Cursor pagination
Count queries
EXPLAIN
Indexes
Composite indexes
Covering indexes
Sargability
Query optimization
Large tables
Batch processing
Bulk updates
Connection pools
Read/write patterns
Caching
Database monitoring
Production troubleshooting
```

---

# 1. What Is Pagination?

Pagination means returning data in smaller chunks instead of loading the entire dataset.

Bad:

```sql
SELECT *
FROM orders;
```

If there are millions of rows, this can be expensive.

Instead:

```sql
SELECT *
FROM orders
ORDER BY id
LIMIT 20;
```

Return only the required page.

---

# 2. Why Pagination Matters

Without pagination:

```text
Millions of rows
↓
Database reads many rows
↓
Network transfer increases
↓
Application memory increases
↓
Response becomes slow
```

With pagination:

```text
Request
↓
Small result set
↓
Lower memory
↓
Lower network cost
↓
Faster response
```

---

# 3. LIMIT and OFFSET

Traditional pagination:

```sql
SELECT *
FROM products
ORDER BY id
LIMIT 20 OFFSET 40;
```

This means:

```text
Skip 40 rows
Return next 20
```

Conceptually:

```text
Page 1 → OFFSET 0
Page 2 → OFFSET 20
Page 3 → OFFSET 40
```

---

# 4. OFFSET Pagination

Typical API:

```text
GET /products?page=3&size=20
```

Backend calculates:

```text
offset = (page - 1) × size
```

For:

```text
page = 3
size = 20
```

we get:

```text
offset = 40
```

---

# 5. Problem with Large OFFSET

Suppose:

```sql
LIMIT 20 OFFSET 1000000;
```

The database may need to process/skip a large number of rows before returning the requested page.

As the offset grows:

```text
Performance can degrade
```

Exact behavior depends on the database and query plan.

---

# 6. Keyset Pagination

Keyset pagination uses a value from the previous page as the starting point.

Example:

```sql
SELECT *
FROM products
WHERE id > 1000
ORDER BY id
LIMIT 20;
```

Instead of:

```text
Skip 1000 rows
```

the database can use:

```text
id > 1000
```

with an appropriate index.

---

# 7. Keyset Pagination Example

First request:

```sql
SELECT *
FROM products
ORDER BY id
LIMIT 20;
```

Suppose the last ID is:

```text
120
```

Next request:

```sql
SELECT *
FROM products
WHERE id > 120
ORDER BY id
LIMIT 20;
```

Then the next cursor could be:

```text
140
```

and so on.

---

# 8. Keyset vs OFFSET

| Feature | OFFSET | Keyset |
|---|---|---|
| Simple to implement | Yes | Moderate |
| Jump to page 100 | Easy | Usually not |
| Large datasets | Can degrade | Often better |
| Stable under concurrent inserts | Can shift | Usually more stable |
| Requires cursor/key | No | Yes |
| Good for infinite scroll | Okay | Excellent |

---

# 9. Cursor Pagination

Cursor pagination exposes a continuation token rather than a page number.

Example:

```text
GET /products?limit=20&cursor=eyJpZCI6MTIwfQ==
```

The cursor might internally represent:

```text
last_id = 120
```

The client shouldn't need to understand the internal cursor structure.

---

# 10. Why Cursor Pagination?

Useful for:

```text
Feeds
Large datasets
Infinite scrolling
High-volume APIs
Frequently changing data
```

Instead of:

```text
page=5000
```

the client says:

```text
continue from this cursor
```

---

# 11. Stable Ordering

Pagination must have deterministic ordering.

Avoid:

```sql
ORDER BY created_at
```

if many rows can have the same timestamp.

Better:

```sql
ORDER BY created_at DESC, id DESC
```

The unique ID acts as a tie-breaker.

This is especially important for keyset pagination.

---

# 12. Keyset Pagination with Timestamp

Suppose:

```text
created_at
id
```

are used for ordering:

```sql
ORDER BY created_at DESC, id DESC
```

The next-page condition can be:

```sql
WHERE
    created_at < :lastCreatedAt
    OR (
        created_at = :lastCreatedAt
        AND id < :lastId
    )
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

This is a common production pattern.

---

# 13. Composite Index for Keyset Pagination

For:

```sql
ORDER BY created_at DESC, id DESC
```

a suitable composite index can help:

```sql
CREATE INDEX idx_orders_created_id
ON orders(created_at, id);
```

Exact index direction requirements and optimizer behavior depend on the database.

---

# 14. Pagination API Design

Offset API:

```text
GET /products?page=3&size=20
```

Response:

```json
{
  "content": [...],
  "page": 3,
  "size": 20,
  "totalPages": 50,
  "totalElements": 1000
}
```

Cursor API:

```text
GET /products?limit=20&cursor=abc123
```

Response:

```json
{
  "data": [...],
  "nextCursor": "xyz456",
  "hasMore": true
}
```

Cursor responses often avoid an expensive total-count query.

---

# 15. Count Query

Traditional pagination often needs:

```sql
SELECT COUNT(*)
FROM products;
```

to calculate:

```text
totalElements
totalPages
```

For very large tables, counting can be expensive depending on the database, filters and execution plan.

---

# 16. Should Every API Return Total Count?

No.

For large datasets, consider:

```text
hasMore
nextCursor
```

instead of:

```text
totalElements
totalPages
```

This can reduce database work.

---

# 17. Spring Data Pagination

Spring Data commonly provides:

```java
Page<Product> findAll(Pageable pageable);
```

Usage:

```java
Pageable pageable =
    PageRequest.of(0, 20);

Page<Product> page =
    productRepository.findAll(pageable);
```

A `Page` can include count information.

---

# 18. Slice in Spring Data

If you don't need total-count information, Spring Data can use:

```java
Slice<Product>
```

A slice focuses on:

```text
Current content
Whether another slice exists
```

This can avoid a separate count query in appropriate repository methods.

---

# 19. LIMIT Page Size

Never blindly trust client-provided page sizes.

Bad:

```text
?page=1&size=1000000
```

Set a maximum:

```text
default = 20
max = 100
```

For example:

```java
int size = Math.min(requestedSize, 100);
```

Validate negative or invalid values too.

---

# 20. EXPLAIN

`EXPLAIN` shows how the database plans to execute a query.

Example:

```sql
EXPLAIN
SELECT *
FROM orders
WHERE user_id = 10;
```

It can provide information such as:

```text
Access method
Indexes
Estimated rows
Join strategy
Filtering
Cost
```

Exact output differs by database.

---

# 21. EXPLAIN ANALYZE

Some databases support:

```sql
EXPLAIN ANALYZE
```

This can execute the query and provide actual execution information.

It can help compare:

```text
Estimated rows
vs
Actual rows
```

Use carefully with:

```text
INSERT
UPDATE
DELETE
```

because execution can modify data depending on the database and command.

---

# 22. What to Look for in EXPLAIN

Look for:

```text
Full table scans
Large row counts
Unexpected join strategy
Missing index usage
Large sorts
Temporary structures
Poor cardinality estimates
Expensive filters
```

Don't judge a plan based on one field alone.

---

# 23. Full Table Scan

A full table scan means the database examines a large portion or all of a table.

Example:

```sql
SELECT *
FROM users
WHERE email = 'x@example.com';
```

If `email` has no suitable index and the table is large, the database may scan many rows.

---

# 24. Index Lookup

With:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

the database may efficiently locate matching rows.

But an index is not guaranteed to be used.

The optimizer chooses based on:

```text
Statistics
Selectivity
Table size
Query shape
Cost
```

---

# 25. Selectivity

Selectivity describes how effectively a condition narrows down rows.

Example:

```text
user_id
```

may be highly selective.

But:

```text
gender
```

may have only a few possible values and may be less selective.

A highly selective predicate can make an index more useful.

---

# 26. Composite Index

Example:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

Useful queries may include:

```sql
WHERE user_id = ?
```

and:

```sql
WHERE user_id = ?
AND status = ?
```

But the usefulness for:

```sql
WHERE status = ?
```

alone may be different.

---

# 27. Leftmost Prefix

For:

```text
(user_id, status, created_at)
```

the leading column is:

```text
user_id
```

Queries using the leading part of the index can often benefit more than queries using only later columns.

This is commonly called the:

```text
leftmost-prefix principle
```

Exact optimizer behavior varies by database.

---

# 28. Index Column Order

Suppose query:

```sql
SELECT *
FROM orders
WHERE user_id = ?
AND status = ?
ORDER BY created_at DESC;
```

A possible composite index is:

```text
(user_id, status, created_at)
```

But index design should be based on:

```text
Actual query patterns
Selectivity
Ordering
Cardinality
Write workload
```

There is no universal column-order rule.

---

# 29. Covering Index

A covering index contains all columns required by a query so the database may be able to answer it using the index without fetching the full table row.

Example query:

```sql
SELECT id, email
FROM users
WHERE email = ?;
```

An index containing:

```text
email
id
```

may cover the query depending on the database.

Covering behavior is database-specific.

---

# 30. Why Not Index Every Column?

Indexes cost:

```text
Storage
INSERT time
UPDATE time
DELETE time
Maintenance
Memory/cache
```

Too many indexes can make write-heavy applications slower.

Index based on actual access patterns.

---

# 31. Index on Foreign Keys

Foreign-key columns are often important candidates for indexes.

Example:

```text
orders.user_id
```

because queries commonly do:

```sql
WHERE user_id = ?
```

or:

```sql
JOIN users
ON users.id = orders.user_id
```

Some databases do not automatically create an index for every foreign key, so verify your schema.

---

# 32. Sargability

A query is generally more index-friendly when the indexed column can be compared directly rather than transformed by a function.

Less index-friendly pattern:

```sql
WHERE LOWER(email) = 'sudhir@example.com'
```

Potentially better:

```sql
WHERE email = 'sudhir@example.com'
```

if email storage/case handling already matches the requirement.

Database-specific functional indexes can also solve some cases.

---

# 33. Function on Indexed Column

Example:

```sql
WHERE YEAR(created_at) = 2026
```

This may make it harder for a normal index on `created_at` to be used efficiently.

Range condition:

```sql
WHERE created_at >= '2026-01-01'
AND created_at < '2027-01-01'
```

is often more index-friendly.

---

# 34. Leading Wildcard

Query:

```sql
WHERE name LIKE '%sudhir%'
```

can be difficult for a normal B-tree index to optimize because the search begins with a wildcard.

Whereas:

```sql
WHERE name LIKE 'sudhir%'
```

may be more index-friendly.

For full-text search requirements, use an appropriate search technology/index.

---

# 35. SELECT *

Avoid unnecessary:

```sql
SELECT *
```

in production queries when you only need a few fields.

Instead:

```sql
SELECT id, name, price
FROM products;
```

Benefits can include:

```text
Less data transfer
Less memory
Potential covering-index opportunities
Clearer contracts
```

---

# 36. Fetch Only Required Columns

For an API:

```text
GET /products
```

you may only need:

```text
id
name
price
thumbnail
```

Don't fetch:

```text
Large description
Internal fields
Audit columns
Unnecessary relationships
```

if they aren't needed.

DTO projections can help.

---

# 37. Batch Processing

Instead of:

```text
Update one row
Update one row
Update one row
...
```

use batching where appropriate.

Example:

```sql
UPDATE products
SET status = 'INACTIVE'
WHERE id IN (1, 2, 3, 4, 5);
```

For very large jobs, process in manageable batches.

---

# 38. Bulk Update

Example:

```sql
UPDATE orders
SET status = 'EXPIRED'
WHERE status = 'PENDING'
AND created_at < CURRENT_TIMESTAMP - INTERVAL 30 DAY;
```

This can be more efficient than loading every order into Java and updating them one by one.

But bulk updates can bypass ORM entity lifecycle behavior and need careful transaction handling.

---

# 39. JPA Bulk Update Warning

Suppose:

```java
@Modifying
@Query("""
    UPDATE OrderEntity o
    SET o.status = :status
    WHERE o.id IN :ids
""")
int updateStatus(...);
```

The database is updated directly.

Already-loaded entities in the JPA persistence context may now contain stale state.

You may need:

```java
@Modifying(clearAutomatically = true)
```

or explicit context management depending on the use case.

---

# 40. Batch Inserts

Instead of:

```text
INSERT
INSERT
INSERT
INSERT
```

the application can use JDBC/JPA batching where appropriate.

Benefits can include:

```text
Fewer round trips
Better throughput
```

But batch sizes should be tuned.

---

# 41. Connection Pool

Java applications often use a connection pool such as:

```text
HikariCP
```

Conceptually:

```text
Application
    ↓
Connection Pool
    ↓
Database connections
```

Instead of opening a brand-new database connection for every request.

---

# 42. Connection Pool Exhaustion

If:

```text
Requests ↑
Transactions become long
Connections remain occupied
```

then:

```text
Pool available connections ↓
```

Eventually:

```text
Requests wait for connection
```

This can cause:

```text
High latency
Timeouts
Database pressure
```

---

# 43. Connection Pool Is Not a Performance Button

Increasing:

```text
maximumPoolSize
```

doesn't automatically make the application faster.

If the database can handle:

```text
20 concurrent queries
```

sending:

```text
200 concurrent queries
```

may make things worse.

Tune based on:

```text
Database capacity
Query latency
CPU
I/O
Concurrency
Application workload
```

---

# 44. Slow Query Troubleshooting

If an endpoint becomes slow:

```text
API slow
 ↓
Check application logs
 ↓
Find SQL query
 ↓
Measure query time
 ↓
EXPLAIN
 ↓
Check indexes
 ↓
Check row counts
 ↓
Check locks
 ↓
Check connection pool
 ↓
Optimize
 ↓
Measure again
```

Don't assume the database is always the problem.

---

# 45. Database Lock Contention

Two transactions may compete for the same resources.

Symptoms:

```text
Queries waiting
High latency
Deadlocks
Low throughput
```

Investigate:

```text
Long transactions
Lock waits
Transaction order
Indexes
Hot rows
```

Exact monitoring commands depend on the database.

---

# 46. Hot Row

A hot row is a frequently modified row that becomes a concurrency bottleneck.

Example:

```text
global_inventory_counter
```

If thousands of requests update the same row:

```text
Many transactions
       ↓
Same row
       ↓
Contention
```

Potential solutions depend on the use case:

```text
Partitioning
Atomic counters
Queueing
Sharding
Caching
Redesign
```

---

# 47. Read Replica

For read-heavy systems, databases can sometimes use read replicas.

Conceptually:

```text
                    ┌── Primary
Application ────────┤
                    └── Read Replica
```

Writes:

```text
Primary
```

Reads:

```text
Replica
```

But replication can introduce:

```text
Replication lag
Read-after-write consistency issues
Routing complexity
```

---

# 48. Read-After-Write Problem

Suppose:

```text
POST /orders
```

writes to primary.

Immediately:

```text
GET /orders
```

reads from replica.

If replication hasn't caught up:

```text
GET
↓
Order not visible yet
```

The application may need to route certain reads to the primary or use a consistency strategy.

---

# 49. Caching

Caching can reduce repeated database reads.

Example:

```text
API
 ↓
Redis
 ↓ cache miss
Database
```

Then:

```text
Database result
 ↓
Redis
 ↓
Future request
```

can avoid the database.

---

# 50. Cache Invalidation

Caching introduces a hard problem:

```text
When should cached data be updated or removed?
```

Common strategies:

```text
TTL
Cache-aside
Write-through
Write-behind
Explicit invalidation
```

For interviews, understand cache-aside well.

---

# 51. Cache-Aside

Typical flow:

```text
Request
 ↓
Check cache
 ↓
Hit → return
```

On miss:

```text
Cache miss
 ↓
Database
 ↓
Store result in cache
 ↓
Return
```

For updates:

```text
Update database
 ↓
Invalidate/update cache
```

---

# 52. Database vs Cache

Database:

```text
Source of truth
Durable
Transactional
```

Cache:

```text
Fast
Temporary
Potentially stale
```

Don't treat the cache as the permanent source of truth unless the architecture explicitly supports that model.

---

# 53. Query Result Caching

Sometimes the same query is repeated:

```text
SELECT product details
```

Caching can help.

But ask:

```text
How often does data change?
How expensive is the query?
How stale can the result be?
How much memory does the cache use?
```

---

# 54. Production Metrics

Useful database/application metrics:

```text
Query latency
Requests per second
Error rate
Connection pool usage
Active connections
Waiting connections
CPU
Memory
Disk I/O
Lock waits
Deadlocks
Cache hit rate
Slow query count
```

---

# 55. Slow Query Logs

Many databases support slow-query logging.

Use it to identify:

```text
Queries taking too long
Queries executed frequently
Queries causing load
```

Don't optimize based only on one slow query.

Look at:

```text
Frequency × latency
```

A query taking 100 ms executed once may matter less than a 10 ms query executed 100,000 times.

---

# 56. Query Frequency Matters

Consider:

```text
Query A:
500 ms × 10 calls/minute
= 5 seconds/minute

Query B:
20 ms × 10,000 calls/minute
= 200 seconds/minute
```

Query B may have a much larger overall impact.

---

# 57. Database Performance Checklist

When optimizing:

```text
□ Measure first
□ EXPLAIN
□ Check indexes
□ Check joins
□ Check filtering
□ Avoid SELECT *
□ Fetch only required columns
□ Check pagination
□ Check N+1
□ Check connection pool
□ Check lock contention
□ Check query frequency
□ Check cache opportunities
□ Check database CPU/I/O
□ Re-measure after changes
```

---

# 58. Common Production Mistake

Bad:

```text
Endpoint slow
↓
Add index
↓
Problem remains
↓
Add more indexes
↓
Writes become slower
```

Better:

```text
Measure
↓
EXPLAIN
↓
Understand bottleneck
↓
Make targeted change
↓
Measure
```

---

# 59. Pagination + Index

Suppose:

```sql
SELECT id, name, price
FROM products
WHERE category_id = ?
ORDER BY id
LIMIT 20;
```

A useful index may be:

```text
(category_id, id)
```

depending on the database and workload.

This can help with:

```text
Filtering
+
Ordering
```

---

# 60. Keyset Pagination + Index

Query:

```sql
SELECT id, name, price
FROM products
WHERE category_id = ?
AND id > ?
ORDER BY id
LIMIT 20;
```

A composite index such as:

```text
(category_id, id)
```

can be a strong candidate.

Again:

```text
EXPLAIN
```

should verify the actual plan.

---

# 61. Pagination and Concurrent Data

OFFSET pagination can be affected when rows are inserted/deleted between requests.

Example:

```text
Page 1 loaded
↓
New product inserted
↓
Page 2 requested
```

The page boundary can shift.

Keyset pagination can provide more stable traversal when the ordering and cursor are designed appropriately.

---

# 62. Deep Pagination

Avoid:

```sql
LIMIT 50 OFFSET 500000;
```

for high-volume APIs when deep navigation isn't required.

Consider:

```text
Keyset pagination
Cursor pagination
Search filters
Time-based pagination
```

---

# 63. API Pagination Limits

A production API should generally define:

```text
Default page size
Maximum page size
Maximum allowed cursor size
Stable sorting
Validation
```

Example:

```text
default = 20
max = 100
```

This prevents clients from accidentally requesting huge datasets.

---

# 64. Search + Pagination

For a product search API:

```text
GET /products?
    category=10
    &minPrice=500
    &maxPrice=5000
    &limit=20
    &cursor=...
```

Database query should use:

```text
Selective filters
Appropriate indexes
Stable ordering
Bounded result size
```

---

# 65. Search Indexes

For search such as:

```sql
WHERE name LIKE '%phone%'
```

a normal B-tree index may not be enough.

Depending on requirements, consider:

```text
Full-text indexes
Database-specific search features
Elasticsearch/OpenSearch
Dedicated search service
```

Don't force a relational index to solve every search problem.

---

# 66. Partitioning

Partitioning divides a large logical table into smaller physical partitions.

Example:

```text
orders
├── 2024
├── 2025
└── 2026
```

Possible partition key:

```text
created_at
```

Benefits can include:

```text
Partition pruning
Maintenance
Large-table management
```

But partitioning adds operational and query-planning complexity.

---

# 67. Partition Pruning

If a table is partitioned by:

```text
created_at
```

and query uses:

```sql
WHERE created_at >= '2026-01-01'
AND created_at < '2026-02-01'
```

the database may only need to inspect relevant partitions.

This is called:

```text
Partition pruning
```

Exact behavior depends on the database.

---

# 68. Sharding

Sharding distributes data across multiple database instances.

Conceptually:

```text
Application
    ↓
Shard router
    ├── DB 1
    ├── DB 2
    └── DB 3
```

Possible shard key:

```text
user_id
```

Sharding can scale very large workloads but adds major complexity.

---

# 69. Partitioning vs Sharding

Partitioning:

```text
One database system
↓
Data divided into partitions
```

Sharding:

```text
Multiple database instances
↓
Data distributed across them
```

Both solve different scaling problems.

---

# 70. Read/Write Separation

A system may use:

```text
Write service
 ↓
Primary DB

Read service
 ↓
Replica DB
```

This can increase read scalability.

But consistency requirements must be understood before using it.

---

# 71. Database Transactions and Performance

Transactions should be:

```text
Short
Focused
Predictable
```

Avoid:

```text
Long network calls
User interaction
Unnecessary processing
Large loops
```

inside a database transaction.

---

# 72. Large Data Processing

Don't load millions of rows into Java:

```java
List<Order> orders = repository.findAll();
```

for a batch operation.

Consider:

```text
Pagination
Streaming
Database-side aggregation
Batch processing
Bulk updates
```

depending on the task.

---

# 73. Streaming

For very large result sets, streaming can reduce application memory usage.

But streaming requires careful management of:

```text
Connection
Transaction
Cursor
Resource lifecycle
```

and should be used deliberately.

---

# 74. Batch Job Pattern

Example:

```text
Fetch 500 records
↓
Process
↓
Commit
↓
Fetch next 500
```

This is often safer than:

```text
One transaction
↓
10 million records
```

because the giant transaction can consume excessive resources.

---

# 75. Production SQL Mental Model

When an endpoint is slow:

```text
Application
   ↓
Connection pool?
   ↓
Database
   ↓
Query
   ↓
Execution plan
   ↓
Indexes
   ↓
Locks
   ↓
Rows processed
   ↓
Rows returned
```

Measure each layer.

---

# 76. Interview: OFFSET vs Keyset Pagination

> OFFSET pagination is simple and supports page-number navigation, but deep offsets can become expensive and page boundaries can shift when data changes. Keyset pagination uses the last-seen ordering value and is usually better for large datasets and infinite-scroll style APIs.

---

# 77. Interview: How Do You Implement Cursor Pagination?

> I use a stable, deterministic sort such as `created_at DESC, id DESC`, return a cursor representing the last row, and use that cursor in the next query. I also create an index that supports the filter and ordering pattern.

---

# 78. Interview: Why Use a Tie-Breaker in Pagination?

> If multiple rows have the same timestamp or sort value, pagination can become unstable. I add a unique field such as ID as a secondary sort key so the ordering is deterministic.

---

# 79. Interview: How Do You Investigate a Slow Query?

> I reproduce the query, inspect its execution plan with `EXPLAIN`, check indexes, row counts, joins, filtering and lock contention, then make a targeted optimization and measure the result again.

---

# 80. Interview: Why Not Add Indexes Everywhere?

> Indexes improve some reads but consume storage and add work to inserts, updates and deletes. Too many indexes can hurt write performance, so I create them based on actual query patterns and measurements.

---

# 81. Interview: What Is a Covering Index?

> A covering index contains all the columns needed for a query, allowing the database to potentially answer the query directly from the index without fetching the full table row.

---

# 82. Interview: What Is Sargability?

> A sargable predicate is written in a way that allows the database to efficiently use an index. For example, a range condition on an indexed date column is generally more index-friendly than applying a function to that column in the WHERE clause.

---

# 83. Interview: How Can You Optimize a Large Table Query?

> I first identify the access pattern and execution plan. Then I consider selective indexes, composite indexes, pagination, partitioning, query rewrites, reduced columns, caching or archiving depending on the actual bottleneck.

---

# 84. Interview: What Is Connection Pool Exhaustion?

> It happens when application requests occupy all available database connections and new requests have to wait or time out. Long transactions, slow queries and excessive concurrency are common contributors.

---

# 85. Interview: Does Increasing the Connection Pool Always Improve Performance?

> No. The database has finite CPU, memory and I/O capacity. Increasing concurrency beyond what the database can handle can increase contention and latency instead of improving throughput.

---

# 86. Interview: What Is N+1?

> N+1 occurs when the application executes one query for the parent list and then one additional query per parent. I address it using appropriate fetch strategies, joins, DTO projections or batching.

---

# 87. Interview: When Would You Use Redis?

> I would use Redis for data that is frequently read, relatively expensive to retrieve, and can tolerate an appropriate caching strategy. I would keep the database as the source of truth and design cache invalidation carefully.

---

# 88. Interview: Read Replica Trade-offs

> Read replicas can increase read capacity, but replication lag can cause stale reads and read-after-write consistency problems. The application needs a routing and consistency strategy.

---

# 89. Interview: When Would You Use Partitioning?

> I'd consider partitioning for very large tables where queries and maintenance naturally align with a partition key such as date. I'd verify that partition pruning and operational benefits justify the additional complexity.

---

# 90. Interview: Partitioning vs Sharding

> Partitioning divides a table within a database system, while sharding distributes data across multiple database instances. Sharding provides broader horizontal scaling but introduces significantly more application and operational complexity.

---

# 91. Interview: How Do You Handle Millions of Rows in a Batch Job?

> I avoid loading everything into memory. Depending on the use case, I use pagination or streaming, process records in bounded batches, keep transactions reasonably small, and use bulk database operations where appropriate.

---

# 92. Production SQL Checklist

```text
□ Pagination
□ LIMIT / OFFSET
□ Keyset pagination
□ Cursor pagination
□ Stable ordering
□ Count queries
□ Page size limits
□ EXPLAIN
□ EXPLAIN ANALYZE
□ Full table scans
□ Selectivity
□ Composite indexes
□ Leftmost prefix
□ Covering indexes
□ Sargability
□ LIKE and search
□ SELECT *
□ Bulk updates
□ Batch processing
□ Connection pools
□ HikariCP
□ Lock contention
□ Hot rows
□ Read replicas
□ Replication lag
□ Cache-aside
□ Redis
□ Slow query logs
□ Partitioning
□ Partition pruning
□ Sharding
□ Streaming
□ Large batch jobs
□ N+1
```

---

# 93. Final Mental Model

For production SQL:

```text
Don't guess.
   ↓
Measure.
   ↓
EXPLAIN.
   ↓
Understand the bottleneck.
   ↓
Optimize the query/index/design.
   ↓
Measure again.
```

For pagination:

```text
Small/simple data
→ OFFSET pagination

Large/high-volume data
→ Keyset/cursor pagination
```

For database scaling:

```text
Optimize query
      ↓
Add appropriate indexes
      ↓
Fix N+1
      ↓
Cache
      ↓
Read replicas
      ↓
Partitioning
      ↓
Sharding
```

Move to the next level only when the workload actually requires it.

> **Interview shortcut:** The strongest production answer is not "I'll add an index." Say: "I'll first measure the query, inspect the execution plan, identify whether the bottleneck is scanning, joining, sorting, locking or connection availability, then apply the smallest change that addresses the actual bottleneck and measure again."
