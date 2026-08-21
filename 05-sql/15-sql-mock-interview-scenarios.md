# SQL — Mock Interview & Scenario-Based Questions

This file is designed for practicing SQL answers in a Java backend interview.

The focus is not just writing queries.

You should be able to explain:

```text
What you would do
Why you would do it
What can go wrong
How you would optimize it
How it behaves under concurrency
```

---

# 1. Mock Interview Rules

For every question, answer in this order:

```text
1. Clarify the requirement
2. Explain the approach
3. Write the SQL
4. Mention edge cases
5. Mention performance if relevant
```

Keep your spoken answer concise.

---

# 2. Scenario: API Is Slow

### Interviewer

Your `/products` API suddenly became slow. What would you check?

### Strong answer

> I would first measure the endpoint latency and identify whether the database is actually the bottleneck. Then I'd inspect the generated SQL, query count, execution plan, indexes, connection-pool usage and whether there is an N+1 problem. After making a targeted change, I'd measure the endpoint again.

Key areas:

```text
API latency
SQL latency
Query count
N+1
EXPLAIN
Indexes
Connection pool
Locks
Rows returned
```

---

# 3. Scenario: Query Takes 5 Seconds

### Interviewer

A query that used to take 100 ms now takes 5 seconds. What do you do?

### Answer

> I'd compare the current execution plan with the previous behavior, check data growth, statistics, indexes, filtering and joins, and look for lock contention or infrastructure changes. I wouldn't immediately add an index without understanding the execution plan.

---

# 4. Scenario: Millions of Products

### Interviewer

There are 20 million products. The API currently uses:

```text
page=50000
size=20
```

What would you change?

### Answer

> I'd consider replacing deep OFFSET pagination with keyset or cursor pagination. I'd use stable ordering, such as `id` or `created_at, id`, and create an index aligned with the filtering and ordering pattern.

---

# 5. Scenario: High Traffic

### Interviewer

Your product API receives 50,000 requests per minute. How would you protect the database?

### Answer

> I'd first identify read and write patterns. For read-heavy traffic I'd consider caching frequently accessed products, proper indexes, pagination, connection-pool tuning and potentially read replicas. I'd also make sure the API has rate limiting and bounded result sizes.

Important:

```text
Don't simply increase DB connections.
```

---

# 6. Scenario: Database CPU Is 100%

### Interviewer

The database CPU is at 100%. What would you investigate?

### Answer

```text
1. Top queries by CPU/time
2. Query frequency
3. Execution plans
4. Missing/inefficient indexes
5. Full table scans
6. Expensive joins
7. Sorting/grouping
8. Lock contention
9. Connection concurrency
10. Recent deployments
```

Then optimize the highest-impact workload first.

---

# 7. Scenario: Too Many Database Connections

### Interviewer

Your application reports connection-pool exhaustion. What could cause it?

### Answer

Possible causes:

```text
Slow queries
Long transactions
Connection leaks
Too much concurrency
Database unavailable
Pool too small
Threads waiting
External calls inside transactions
```

Don't assume the solution is simply increasing the pool size.

---

# 8. Scenario: Database Is Healthy but API Is Slow

### Interviewer

Database CPU is normal and queries are fast, but API latency is high. What else would you check?

### Answer

```text
Application CPU
GC
Thread pools
Connection-pool wait time
External APIs
Network latency
Serialization
Large JSON responses
Locks
Application code
```

The database isn't automatically the bottleneck.

---

# 9. Scenario: N+1 Appears in Production

### Interviewer

A page loads 100 users and generates 101 SQL queries. What happened?

### Answer

> This is the N+1 query problem. One query loads the users and additional queries load a relationship for each user. I'd inspect the generated SQL and choose a fetch join, entity graph, DTO projection or batching strategy depending on the response requirements.

---

# 10. Scenario: JOIN FETCH Creates Huge Results

### Interviewer

You fixed N+1 using a fetch join, but now the query returns a huge result set. What do you do?

### Answer

> I'd check whether we're fetching a collection unnecessarily. For list endpoints, I'd consider a DTO projection or a dedicated query that returns only the required fields. I wouldn't assume that one large join is always better than multiple queries.

---

# 11. Scenario: Users Without Orders

### Interviewer

Find users who have never placed an order.

### Answer

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

Why `NOT EXISTS`?

> I only need to know whether an order exists, so existence semantics are a natural fit.

---

# 12. Scenario: Users With More Than 10 Orders

```sql
SELECT
    user_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 10;
```

---

# 13. Scenario: Users With Zero Orders Too

### Interviewer

Return every user and their order count, including users with zero orders.

### Answer

```sql
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
GROUP BY u.id, u.name;
```

Important:

```text
COUNT(o.id)
```

rather than:

```text
COUNT(*)
```

because the latter counts the preserved LEFT JOIN row.

---

# 14. Scenario: Highest Salary per Department

```sql
WITH ranked AS (
    SELECT
        e.*,
        ROW_NUMBER() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC, id
        ) AS rn
    FROM employees e
)
SELECT *
FROM ranked
WHERE rn = 1;
```

If ties must all be returned:

```text
Use RANK()
```

---

# 15. Scenario: Second Highest Salary

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

This returns the second distinct-highest salary.

---

# 16. Scenario: Duplicate Emails

```sql
SELECT
    email,
    COUNT(*) AS occurrences
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

Long-term prevention:

```sql
UNIQUE(email)
```

after existing duplicate data is cleaned up.

---

# 17. Scenario: Latest Order per User

```sql
WITH ranked AS (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY created_at DESC, id DESC
        ) AS rn
    FROM orders o
)
SELECT *
FROM ranked
WHERE rn = 1;
```

Why include `id`?

> It gives deterministic ordering when multiple orders have the same timestamp.

---

# 18. Scenario: Top 3 Orders per User

```sql
WITH ranked AS (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY amount DESC, id
        ) AS rn
    FROM orders o
)
SELECT *
FROM ranked
WHERE rn <= 3;
```

---

# 19. Scenario: Top Products by Revenue

Assume:

```text
products
orders
order_items
```

Query:

```sql
SELECT
    p.id,
    p.name,
    SUM(oi.quantity * oi.unit_price) AS revenue
FROM products p
JOIN order_items oi
    ON oi.product_id = p.id
JOIN orders o
    ON o.id = oi.order_id
WHERE o.status = 'PAID'
GROUP BY p.id, p.name
ORDER BY revenue DESC
LIMIT 10;
```

Always clarify whether:

```text
sales
=
units sold
```

or:

```text
sales
=
revenue
```

---

# 20. Scenario: Monthly Revenue

```sql
SELECT
    EXTRACT(YEAR FROM created_at) AS year,
    EXTRACT(MONTH FROM created_at) AS month,
    SUM(amount) AS revenue
FROM orders
WHERE status = 'PAID'
GROUP BY
    EXTRACT(YEAR FROM created_at),
    EXTRACT(MONTH FROM created_at)
ORDER BY year, month;
```

Mention:

> Date extraction syntax differs across databases.

---

# 21. Scenario: Users Spending Above Average

```sql
WITH user_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total_spent
    FROM orders
    WHERE status = 'PAID'
    GROUP BY user_id
)
SELECT *
FROM user_totals
WHERE total_spent > (
    SELECT AVG(total_spent)
    FROM user_totals
);
```

This uses a CTE twice.

Database support for referencing a CTE multiple times is common, but optimizer behavior varies.

---

# 22. Scenario: Products Never Ordered

```sql
SELECT p.*
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.id
);
```

---

# 23. Scenario: Categories Without Products

```sql
SELECT c.*
FROM categories c
LEFT JOIN products p
    ON p.category_id = c.id
WHERE p.id IS NULL;
```

---

# 24. Scenario: Product Search

### Interviewer

The product table contains 50 million rows. Users search:

```text
"wireless headphones"
```

Would you use:

```sql
LIKE '%wireless headphones%'
```

### Answer

> Not necessarily. A normal B-tree index may not efficiently support leading-wildcard searches. For serious search requirements I'd evaluate full-text search or a dedicated search engine such as Elasticsearch/OpenSearch depending on the use case.

---

# 25. Scenario: Search Prefix

For:

```sql
WHERE name LIKE 'wireless%'
```

a normal B-tree index may be able to help, depending on database/collation/query plan.

Always verify with:

```sql
EXPLAIN
```

---

# 26. Scenario: Pagination

### Interviewer

Implement:

```text
GET /products?page=3&size=20
```

### Answer

```sql
SELECT id, name, price
FROM products
ORDER BY id
LIMIT 20 OFFSET 40;
```

Calculation:

```text
offset = (page - 1) × size
```

---

# 27. Scenario: Deep Pagination

### Interviewer

What if the user requests page 500,000?

### Answer

> I'd avoid deep OFFSET pagination for a high-volume API. I'd consider cursor/keyset pagination using a stable indexed sort key.

Example:

```sql
SELECT id, name, price
FROM products
WHERE id > :lastId
ORDER BY id
LIMIT 20;
```

---

# 28. Scenario: Stable Cursor

If sorting by:

```text
created_at DESC
```

timestamps may not be unique.

Use:

```text
created_at DESC, id DESC
```

Next-page condition:

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

---

# 29. Scenario: Overselling Inventory

Initial stock:

```text
10
```

Two users simultaneously purchase:

```text
7
```

### Bad approach

```text
SELECT stock
↓
Java checks stock
↓
UPDATE stock
```

This can race.

### Better approach

```sql
UPDATE products
SET stock = stock - :quantity
WHERE id = :productId
  AND stock >= :quantity;
```

Then:

```text
affected rows = 1
→ success

affected rows = 0
→ insufficient stock / condition failed
```

Use the appropriate transaction/concurrency strategy around the complete order operation.

---

# 30. Scenario: Concurrent Order Placement

### Interviewer

How would you make order placement safe?

### Answer

> I'd use a service-level transaction for the database changes, make the inventory update concurrency-safe, create the order and order items in the same transaction, and ensure failures roll back the required database changes.

---

# 31. Scenario: Two Users Update the Same Product

### Interviewer

How can you prevent lost updates?

### Answer

Possible strategies:

```text
Optimistic locking
Pessimistic locking
Atomic SQL updates
```

For JPA:

```java
@Version
private Long version;
```

---

# 32. Scenario: Optimistic Locking

Two requests read:

```text
version = 5
```

Request A updates first:

```text
5 → 6
```

Request B attempts an update based on:

```text
version = 5
```

The update should fail rather than silently overwriting A's change.

---

# 33. Scenario: Pessimistic Lock

When contention is high and serialized access is required:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Product> findById(Long id);
```

This can prevent concurrent modifications while the transaction holds the lock.

Trade-off:

```text
More correctness control
+
Less concurrency
+
Potential lock waits
```

---

# 34. Scenario: Long Transaction

### Interviewer

You find a transaction that takes 10 seconds because it calls an external API.

What's wrong?

### Answer

> The transaction may hold database resources while waiting for the network call. I'd try to keep the database transaction focused and consider an asynchronous workflow or outbox pattern if the business process requires reliable event handling.

---

# 35. Scenario: Database + Kafka

Suppose you need:

```text
Create order in DB
+
Publish OrderCreated event
```

How do you make this reliable?

### Answer

> I'd consider the transactional outbox pattern. The order and an outbox event are written in the same database transaction. A separate process publishes the event and marks it processed.

---

# 36. Scenario: Cache Product Details

### Interviewer

Product details are read millions of times but change rarely. What would you do?

### Answer

> I'd consider cache-aside using Redis. On a cache hit, return the cached data. On a miss, read from the database and populate the cache. On updates, invalidate or refresh the cache according to the consistency requirement.

---

# 37. Scenario: Cache Is Stale

### Interviewer

The database says price ₹1,000 but Redis says ₹900. What do you investigate?

```text
Cache invalidation
TTL
Write path
Race conditions
Multiple application instances
Cache key
Serialization
Deployment/version differences
```

---

# 38. Scenario: Read Replica

### Interviewer

Can you send every GET request to a read replica?

### Answer

> Not necessarily. Replication lag can cause stale reads. After a write, requests that require read-after-write consistency may need to read from the primary or use another consistency strategy.

---

# 39. Scenario: Database Index

### Interviewer

This query is slow:

```sql
SELECT *
FROM orders
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;
```

What would you investigate?

### Answer

> I'd inspect the execution plan and consider an index aligned with the filter and ordering, such as `(user_id, created_at)`. I'd verify the actual benefit with `EXPLAIN` rather than assuming the index is correct.

---

# 40. Scenario: Composite Index

Index:

```text
(user_id, status, created_at)
```

Which query is most naturally aligned?

```sql
WHERE user_id = ?
AND status = ?
ORDER BY created_at DESC
```

The index matches the filtering and ordering pattern.

But actual usefulness depends on:

```text
Cardinality
Statistics
Database
Data distribution
Query plan
```

---

# 41. Scenario: Index on Every Column

### Interviewer

Why not add indexes to every column?

### Answer

> Indexes improve reads but add storage and write-maintenance cost. Inserts, updates and deletes may become more expensive. I create indexes based on actual access patterns and measured workload.

---

# 42. Scenario: Full Table Scan

### Interviewer

`EXPLAIN` shows a full table scan on a 100-million-row table. What do you do?

### Answer

> First I'd determine whether the scan is actually inappropriate. If the query should be selective, I'd investigate predicates, indexes, data distribution and statistics. Then I'd test an appropriate index or query rewrite and compare execution plans.

A full scan is not automatically bad for every query.

---

# 43. Scenario: EXPLAIN Looks Good but Query Is Slow

Possible causes:

```text
Lock waits
I/O
Network
Cold cache
Database CPU
Connection waiting
Large result transfer
Concurrent workload
Bad cardinality estimates
```

Don't treat the query plan as the only source of latency.

---

# 44. Scenario: COUNT(*) Is Slow

### Interviewer

Your API needs:

```text
totalElements
totalPages
```

but the table has hundreds of millions of rows.

What could you do?

### Answer

> I'd determine whether an exact count is actually required. For large cursor-based APIs, I may return `hasMore` and `nextCursor` instead. If an exact count is required, I'd investigate the count query and workload rather than assuming it should be run on every request.

---

# 45. Scenario: Large Batch Job

### Interviewer

You need to process 10 million orders.

Would you do:

```java
repository.findAll()
```

?

### Answer

> No. I'd use bounded pagination, streaming or database-side processing depending on the task. I'd process in manageable batches and avoid holding millions of entities in application memory.

---

# 46. Scenario: Bulk Update

### Interviewer

Deactivate 5 million inactive users.

Bad:

```text
Load all users
↓
Loop in Java
↓
save() one by one
```

Better:

```sql
UPDATE users
SET status = 'INACTIVE'
WHERE last_login < :cutoff
  AND status = 'ACTIVE';
```

For very large updates, consider batching/partitioning/locking impact.

---

# 47. Scenario: Bulk Update + JPA

### Interviewer

What problem can a JPQL bulk update create?

### Answer

> It updates database rows directly, so entities already managed in the persistence context can become stale. I may need to clear or refresh the persistence context after the bulk operation.

---

# 48. Scenario: Delete Old Data

### Interviewer

Delete 200 million old audit records.

Would you execute one huge DELETE?

### Answer

> Not blindly. I'd consider retention requirements, batching, transaction size, lock impact, indexes, archiving and partitioning. For very large time-based datasets, partitioning can make retention operations much easier.

---

# 49. Scenario: Deadlock

### Interviewer

Two transactions frequently deadlock. What do you investigate?

```text
Transaction order
Lock order
Long transactions
Indexes
Rows being locked
Concurrent operations
```

A common mitigation is to ensure transactions acquire resources in a consistent order.

---

# 50. Scenario: Connection Pool Exhaustion

### Interviewer

HikariCP shows all connections busy.

What do you check?

```text
Slow queries
Long transactions
Connection leaks
External calls inside transactions
Pool size
Database capacity
Thread contention
Timeout configuration
```

---

# 51. Scenario: Increase Hikari Pool?

### Interviewer

Would you increase:

```properties
maximumPoolSize=100
```

?

### Answer

> Only after understanding the bottleneck. If the database can efficiently process 20 concurrent queries, increasing the pool to 100 can increase contention and make latency worse.

---

# 52. Scenario: API Returns Huge JSON

### Interviewer

The `/orders` endpoint returns 50,000 records and becomes slow.

What would you change?

```text
Pagination
DTO projection
Maximum page size
Filtering
Field selection
Compression where appropriate
Caching where appropriate
```

Never let an endpoint return unbounded data by default.

---

# 53. Scenario: Security + SQL

### Interviewer

How do you prevent SQL injection?

### Answer

```text
Prepared statements
JPA parameter binding
Named parameters
JdbcTemplate parameters
```

Avoid:

```java
"SELECT * FROM users WHERE email = '" + email + "'"
```

---

# 54. Scenario: Transaction Rollback

### Interviewer

The order is inserted successfully, but inventory update fails. What should happen?

### Answer

> If both are part of the same database transaction and the business operation requires atomicity, the transaction should roll back so the order isn't left in a partially completed state.

---

# 55. Scenario: Checked Exception

### Interviewer

Will every exception automatically roll back a Spring transaction?

### Answer

> No. Spring's default rollback behavior primarily applies to unchecked exceptions. Checked exceptions may require explicit rollback configuration such as `rollbackFor`.

---

# 56. Scenario: Transactional Self Invocation

### Interviewer

Why might this not work as expected?

```java
public void methodA() {
    methodB();
}

@Transactional
public void methodB() {
}
```

### Answer

> In proxy-based Spring transaction management, an internal call on the same object can bypass the proxy, so the transactional interceptor may not be applied to `methodB()`.

---

# 57. Scenario: Read-Only Transaction

### Interviewer

Why use:

```java
@Transactional(readOnly = true)
```

?

### Answer

> It communicates that the operation is read-oriented and may allow optimizations depending on the persistence stack. It shouldn't be treated as a universal guarantee that writes are impossible.

---

# 58. Scenario: LazyInitializationException

### Interviewer

Why does this happen?

```text
Transaction ends
↓
Entity becomes detached
↓
Access LAZY relationship
↓
Exception
```

### Answer

> The relationship was not initialized while the persistence context was available. I'd explicitly fetch the required data using a fetch join, entity graph or DTO projection rather than making every relationship eager.

---

# 59. Scenario: EAGER Everything

### Interviewer

Why not configure every relationship as EAGER?

### Answer

> It can cause unnecessary joins, huge result sets, excessive memory usage and unexpected database queries. Fetching should be designed according to the use case.

---

# 60. Scenario: DTO vs Entity

### Interviewer

Why not return JPA entities directly from REST APIs?

### Answer

> DTOs give the API a controlled contract, prevent accidental exposure of internal fields, reduce unnecessary data loading and help avoid problems with lazy relationships and serialization.

---

# 61. Scenario: Product List

### Interviewer

For:

```text
GET /products
```

would you return full `Product` entities?

### Answer

> Usually I'd return a product summary DTO containing only fields needed by the list, such as ID, name, price and image. This keeps the query and response smaller.

---

# 62. Scenario: Order Details

For:

```text
GET /orders/{id}
```

you might need:

```text
Order
Customer
Items
Product summary
Payment status
```

Instead of loading a massive entity graph blindly:

```text
Use a dedicated query
+
DTO projection
+
appropriate fetch strategy
```

---

# 63. Scenario: Database Constraint

### Interviewer

Should validation exist only in Java?

### Answer

> No. Application validation improves user experience, but important data integrity rules should also be enforced by database constraints such as NOT NULL, UNIQUE, FOREIGN KEY and CHECK where appropriate.

---

# 64. Scenario: Unique Email

If email must be unique:

```sql
ALTER TABLE users
ADD CONSTRAINT uk_users_email
UNIQUE (email);
```

Don't rely only on:

```text
SELECT before INSERT
```

because concurrent requests can race.

---

# 65. Scenario: Race Condition on Unique Email

Two requests:

```text
Request A → checks email → not found
Request B → checks email → not found
Request A → inserts
Request B → inserts
```

A unique database constraint prevents the final duplicate.

The application should handle the resulting constraint violation gracefully.

---

# 66. Scenario: Soft Delete

Instead of:

```sql
DELETE FROM products
WHERE id = 10;
```

use:

```text
deleted_at
```

or:

```text
active
```

depending on requirements.

Then queries include:

```sql
WHERE deleted_at IS NULL
```

Consider:

```text
Indexes
Uniqueness
Storage growth
Recovery
Business requirements
```

---

# 67. Scenario: Soft Delete and Unique Email

If users are soft-deleted, can another user reuse the same email?

This is a business rule.

Possible database-specific solutions include:

```text
Partial/filtered unique index
Generated key
Application workflow
Separate identity table
```

Do not solve it only in Java if database enforcement is possible and required.

---

# 68. Scenario: Audit Requirements

If the business requires:

```text
Who changed price?
When?
Old value?
New value?
```

consider:

```text
Audit table
Application audit events
Database auditing
Event/outbox pattern
```

depending on compliance and architecture.

---

# 69. Scenario: Production Schema Change

### Interviewer

Add a NOT NULL column to a huge production table.

Would you do it in one deployment?

### Answer

> I'd consider backward-compatible migration. For a large table, I'd typically introduce the column in a safe state, deploy application support, backfill in batches if needed, then enforce the final constraint once existing data is valid.

---

# 70. Scenario: Zero-Downtime Migration

A common pattern:

```text
1. Add new nullable column
2. Deploy code that supports old + new
3. Backfill data
4. Switch reads/writes
5. Verify
6. Add final constraints
7. Remove old column later
```

This reduces deployment coupling.

---

# 71. Scenario: Rename a Column

Avoid:

```text
Rename DB column
+
Deploy application
```

if old application instances may still be running.

Prefer an expand/contract approach:

```text
Add new column
↓
Write both
↓
Backfill
↓
Read new
↓
Stop old writes
↓
Remove old column
```

---

# 72. Scenario: High Read Traffic

Architecture:

```text
Clients
  ↓
Load Balancer
  ↓
Spring Boot instances
  ↓
Redis
  ↓
Database
```

Potential scaling strategy:

```text
Horizontal app scaling
+
Caching
+
DB optimization
+
Read replicas if required
```

The correct design depends on the bottleneck.

---

# 73. Scenario: High Write Traffic

Possible strategies:

```text
Efficient indexes
Batch writes
Queueing
Partitioning
Database scaling
Sharding
Async processing
```

But don't introduce distributed complexity before measuring the actual bottleneck.

---

# 74. Scenario: Database Becomes Unavailable

What should the application do?

```text
Timeout quickly
Avoid infinite retries
Use bounded retries where appropriate
Circuit breaking for dependencies
Return meaningful errors
Protect the database from retry storms
Monitor/alert
```

Retries must be designed carefully.

---

# 75. Retry Storm

Suppose:

```text
DB slows down
↓
Requests timeout
↓
Every request retries 5 times
↓
More DB traffic
↓
DB gets even worse
```

This is a retry storm.

Use:

```text
Backoff
Jitter
Retry limits
Circuit breakers
Bulkheads
```

where appropriate.

---

# 76. Scenario: Slow External API

Your transaction does:

```text
DB update
↓
External API call
↓
Wait 10 seconds
↓
DB commit
```

Problem:

```text
Connection occupied
Transaction open
Locks potentially held
Pool capacity reduced
```

Prefer separating external work from the critical database transaction when business requirements allow.

---

# 77. Scenario: Reporting Query

A dashboard runs a huge aggregation every 5 seconds.

What could you do?

```text
Caching
Materialized views
Pre-aggregation
Read replica
Reporting database
ETL
Dedicated analytics system
```

Choose based on:

```text
Freshness requirements
Query complexity
Data volume
Cost
```

---

# 78. Scenario: Database vs Elasticsearch

### Interviewer

Should Elasticsearch replace MySQL?

### Answer

> Not necessarily. MySQL can remain the source of truth for transactional data, while Elasticsearch/OpenSearch can provide specialized search capabilities. Data synchronization and consistency need to be designed.

---

# 79. Scenario: Cache vs Database

### Interviewer

Should Redis become the source of truth?

### Answer

> Usually the relational database remains the source of truth for transactional data. Redis is commonly used as a fast cache or supporting data store depending on the architecture.

---

# 80. Scenario: SQL vs NoSQL

### Interviewer

When would you consider NoSQL?

### Answer

> I'd consider it when the access patterns, scale or data model fit a NoSQL system better—for example, very high-volume key-value access or flexible document-oriented data. I wouldn't choose it simply because the application is large.

---

# 81. Scenario: Database Scaling

A useful progression:

```text
1. Measure
2. Optimize queries
3. Add correct indexes
4. Fix N+1
5. Add caching
6. Scale application horizontally
7. Read replicas
8. Partitioning
9. Sharding
```

Not every system needs all nine.

---

# 82. Scenario: API Rate Limiting

High traffic can overload the database even when SQL is optimized.

Protect the backend with:

```text
Rate limiting
Request quotas
Caching
Pagination limits
Circuit breakers
Backpressure
```

This is especially important for public APIs.

---

# 83. Scenario: Unbounded Query

Bad:

```text
GET /orders
```

returns:

```text
Every order ever created
```

Better:

```text
GET /orders?page=0&size=20
```

or:

```text
GET /orders?limit=20&cursor=...
```

---

# 84. Scenario: Production Incident

### Interviewer

Orders are failing after a deployment. What is your approach?

### Answer

```text
1. Check error rate
2. Check application logs
3. Check database health
4. Compare deployment changes
5. Identify failing query/API
6. Check connection pool
7. Check recent schema changes
8. Roll back or mitigate if necessary
9. Fix root cause
10. Verify recovery
```

Stay systematic.

---

# 85. Scenario: Index Deployment

Adding an index to a huge production table can itself be expensive.

Consider:

```text
Locking behavior
Online/concurrent index creation support
Disk space
CPU
Replication impact
Deployment timing
```

The exact strategy depends on the database.

---

# 86. Scenario: Data Consistency

### Interviewer

The application writes order data successfully but event publishing fails.

How do you avoid inconsistency?

### Answer

> I'd avoid assuming a distributed transaction across the database and messaging system. A transactional outbox is a common approach: commit the order and outbox record together, then publish asynchronously with retry handling.

---

# 87. Scenario: Duplicate Events

Suppose an event is published twice.

Your consumer should ideally be:

```text
Idempotent
```

For example:

```text
event_id
```

can be stored and checked before processing.

---

# 88. Scenario: Exactly Once

Be careful saying:

```text
"Kafka gives exactly-once so duplicates are impossible."
```

A safer interview answer:

> I'd design consumers to be idempotent even when the surrounding workflow can produce duplicate delivery or retries. Exactly-once semantics depend on the complete architecture, not just one component.

---

# 89. Scenario: SQL Injection

Bad:

```java
String sql =
    "SELECT * FROM users WHERE email = '" + email + "'";
```

Better:

```java
PreparedStatement
```

or:

```java
@Query("""
    SELECT u
    FROM User u
    WHERE u.email = :email
""")
```

with parameter binding.

---

# 90. Scenario: Connection Leak

Symptoms:

```text
Pool gradually reaches max
Requests start timing out
Restart temporarily fixes problem
```

Investigate:

```text
Unclosed JDBC resources
Transactions not completing
Streaming result sets
Custom connection handling
Exceptions bypassing cleanup
```

Modern Spring/JPA infrastructure normally manages many resources automatically, but custom JDBC code still needs proper resource handling.

---

# 91. Scenario: Query Timeout

Set reasonable timeouts according to the operation.

A timeout should prevent:

```text
One bad query
↓
Connection occupied indefinitely
↓
Pool exhaustion
```

But timeouts should not simply hide an underlying query-design problem.

---

# 92. Scenario: Lock Timeout

If a transaction waits too long for a lock:

```text
Investigate blocking transaction
↓
Find long-running transaction
↓
Check access order
↓
Optimize transaction
↓
Reduce lock duration
```

---

# 93. Scenario: Deadlock Prevention

A common technique:

```text
Always update resources in a consistent order.
```

For example:

```text
Always lock Product A before Product B
```

rather than:

```text
Transaction 1 → A then B
Transaction 2 → B then A
```

The database may still need deadlock detection/retry handling.

---

# 94. Scenario: Query Correctness

A query returns duplicate users after adding a join.

Why?

Because:

```text
One user
↓
Many orders
↓
Many result rows
```

Possible solutions:

```sql
SELECT DISTINCT ...
```

or:

```text
GROUP BY
```

or:

```text
EXISTS
```

depending on what the query actually needs.

---

# 95. Scenario: EXISTS vs JOIN

If the requirement is:

```text
"Find users who have at least one order."
```

`EXISTS` is often a natural expression:

```sql
SELECT u.*
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

If you actually need order columns, a join may be more appropriate.

---

# 96. Scenario: LEFT JOIN Accidentally Becomes INNER JOIN

Bad:

```sql
SELECT u.*
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
WHERE o.status = 'PAID';
```

The WHERE condition eliminates NULL order rows, so users without orders disappear.

If you want to preserve users:

```sql
SELECT u.*, o.id
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
   AND o.status = 'PAID';
```

This is a classic SQL interview trap.

---

# 97. Scenario: COUNT with LEFT JOIN

Correct for users with zero orders:

```sql
COUNT(o.id)
```

Potentially misleading:

```sql
COUNT(*)
```

because the LEFT JOIN produces one preserved row for users with no matching order.

---

# 98. Scenario: NULL Aggregation

Suppose:

```sql
SUM(amount)
```

has no matching rows.

It can return:

```text
NULL
```

Use:

```sql
COALESCE(SUM(amount), 0)
```

when zero is the required business result.

---

# 99. Scenario: SQL Injection Prevention in JdbcTemplate

Parameterized:

```java
jdbcTemplate.query(
    "SELECT * FROM users WHERE email = ?",
    rowMapper,
    email
);
```

Not:

```java
jdbcTemplate.query(
    "SELECT * FROM users WHERE email = '" + email + "'",
    rowMapper
);
```

---

# 100. Scenario: Interviewer Challenges Your Query

If the interviewer says:

> "What if there are duplicate timestamps?"

Answer:

> I'd add a unique tie-breaker such as `id` to the ordering so the result is deterministic.

If they say:

> "What about 100 million rows?"

Answer:

> I'd revisit pagination and indexing, inspect the execution plan, and consider keyset pagination or other scaling strategies.

If they say:

> "What if two users buy the last item?"

Answer:

> I'd use a concurrency-safe inventory update or appropriate locking rather than a non-atomic read-check-update.

---

# 101. Rapid-Fire Questions

Try answering each in 30 seconds:

```text
1. What is an index?

2. Why not index every column?

3. What is N+1?

4. What is a covering index?

5. What is keyset pagination?

6. Why is OFFSET bad for deep pagination?

7. What is a transaction?

8. What is dirty read?

9. What is optimistic locking?

10. What is pessimistic locking?

11. What is deadlock?

12. What is connection-pool exhaustion?

13. What is lazy loading?

14. What is dirty checking?

15. What is the persistence context?

16. What is DTO projection?

17. What is JPQL?

18. JPQL vs native SQL?

19. What is Flyway?

20. Why use Redis?

21. What is a read replica?

22. What is replication lag?

23. What is partitioning?

24. What is sharding?

25. What is the outbox pattern?
```

---

# 102. Behavioral + Technical Scenario

### Interviewer

Tell me about a production performance issue you handled.

Use this structure:

```text
Situation
→ What was slow?

Investigation
→ Logs
→ SQL
→ Metrics
→ EXPLAIN

Action
→ Query/index/fetch/pagination/cache change

Result
→ Latency/throughput/error improvement

Learning
→ How you would prevent it
```

Don't invent exact numbers if you don't know them.

---

# 103. Strong Backend Interview Answer Pattern

For architecture questions:

```text
Requirement
↓
Simple solution
↓
Potential bottleneck
↓
Scaling strategy
↓
Consistency trade-off
↓
Monitoring
```

Example:

> "For a product API I'd start with indexed filtering and bounded pagination. If traffic becomes read-heavy, I'd introduce caching. If database reads remain the bottleneck, I'd evaluate read replicas. For very large datasets I'd consider keyset pagination and partitioning where appropriate."

---

# 104. SQL Interview Mistakes to Avoid

Don't say:

```text
"Just add an index."
```

Say:

```text
"I'd inspect EXPLAIN first."
```

Don't say:

```text
"Use EAGER to fix lazy loading."
```

Say:

```text
"I'd explicitly fetch what the use case requires."
```

Don't say:

```text
"Increase the connection pool."
```

Say:

```text
"I'd identify why connections are occupied."
```

Don't say:

```text
"Use Redis for everything."
```

Say:

```text
"I'd cache data that benefits from caching."
```

Don't say:

```text
"Use sharding for scale."
```

Say:

```text
"I'd introduce sharding only when simpler scaling options are insufficient."
```

---

# 105. Final Mock Interview

Answer these verbally before checking your notes:

### Q1
Your API receives 100,000 requests/minute. How would you protect the database?

### Q2
A query suddenly goes from 50 ms to 3 seconds. What do you check?

### Q3
You find 101 queries for 100 users. What happened?

### Q4
Your table contains 500 million rows. How would you paginate?

### Q5
Two users purchase the final product simultaneously. How do you prevent overselling?

### Q6
A database transaction takes 15 seconds because of an external API call. What would you change?

### Q7
A read replica returns stale data immediately after a write. Why?

### Q8
Redis contains stale product prices. How would you handle it?

### Q9
A bulk JPA update was executed and existing entities contain old values. Why?

### Q10
How would you deploy a schema change without breaking old application instances?

---

# 106. Model Answers

### Q1

> I'd start by identifying the traffic pattern and bottleneck. I'd use caching for hot reads, enforce pagination limits, optimize database queries and indexes, scale application instances horizontally, and consider read replicas if the workload is read-heavy. I'd also use rate limiting to protect the system.

### Q2

> I'd compare execution plans and inspect query latency, data growth, indexes, locks, statistics and recent deployments. I'd also check whether the database is under resource pressure.

### Q3

> It's likely N+1. One query loads the users and one query per user loads related data. I'd use an appropriate fetch strategy or DTO query depending on the endpoint.

### Q4

> I'd avoid deep OFFSET pagination and use keyset/cursor pagination with a stable indexed ordering.

### Q5

> I'd use an atomic conditional stock update, optimistic locking or pessimistic locking based on contention and business requirements, within the appropriate transaction.

### Q6

> I'd avoid holding the database transaction open during the external call. If reliable coordination is required, I'd consider an asynchronous workflow or transactional outbox.

### Q7

> The replica hasn't caught up with the primary yet. That's replication lag. For read-after-write consistency, the application may need to read from the primary or use another consistency strategy.

### Q8

> I'd review cache invalidation, TTL, update ordering and race conditions. The database should generally remain the source of truth for transactional product data.

### Q9

> Bulk updates operate directly on database rows and don't automatically synchronize already-managed entities in the persistence context. I'd clear or refresh the context as appropriate.

### Q10

> I'd use an expand/contract migration: introduce the new schema compatibly, deploy code supporting both versions, backfill, switch reads/writes, then remove the old structure after verification.

---

# 107. Final Interview Mindset

When an interviewer gives you a production scenario, don't jump straight to technology.

Think:

```text
What is the requirement?
        ↓
What is the bottleneck?
        ↓
What is the simplest solution?
        ↓
What happens under concurrency?
        ↓
What happens at 10× scale?
        ↓
What are the consistency trade-offs?
        ↓
How will I measure it?
```

This is the difference between:

```text
"I know SQL"
```

and:

```text
"I can build and troubleshoot a Java backend."
```

---

# 108. Final SQL + Backend Revision

Before your Java backend interview, make sure you can comfortably explain:

```text
SQL
├── SELECT / WHERE
├── JOIN
├── GROUP BY / HAVING
├── Subqueries
├── EXISTS
├── CTE
├── Window functions
├── NULL
├── Indexes
├── Composite indexes
├── EXPLAIN
├── Transactions
├── Isolation
├── Locks
├── Deadlocks
├── Pagination
└── Query optimization

Spring/JPA
├── Entity
├── Repository
├── Hibernate
├── Lazy loading
├── N+1
├── DTO projection
├── JPQL
├── Native SQL
├── @Transactional
├── Persistence context
├── Dirty checking
├── Optimistic locking
├── Pessimistic locking
└── Connection pool

Production
├── Redis
├── Read replicas
├── Rate limiting
├── Batch processing
├── Partitioning
├── Sharding
├── Outbox
├── Monitoring
└── Incident troubleshooting
```

> **Final rule:** Don't optimize what you haven't measured. In interviews, explaining how you would investigate the problem is often more impressive than immediately naming a technology.
