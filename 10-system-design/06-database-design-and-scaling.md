# System Design — File 06: Database Design & Scaling

Databases are often the most important shared dependency in a backend system.

A common architecture starts as:

```text
Client
  ↓
Spring Boot
  ↓
MySQL
```

As traffic and data grow:

```text
Clients
   ↓
Load Balancer
   ↓
App Instances
   ↓
Cache
   ↓
Database Layer
   ├── Primary
   ├── Read Replicas
   └── Partitions / Shards
```

The goal is not to make the database architecture complicated.

The goal is to make the database capable of handling the required:

```text
Reads
Writes
Data volume
Concurrency
Availability
Consistency
```

---

# 1. Database Design Starts With Access Patterns

Don't start with:

```text
MySQL
PostgreSQL
MongoDB
```

Start with:

```text
What data do we have?
How is it queried?
How often is it updated?
What consistency is required?
How much data will exist?
```

The access pattern should influence the database design.

---

# 2. Relational Database

Examples:

```text
MySQL
PostgreSQL
Oracle
SQL Server
```

A relational database stores structured data using:

```text
Tables
Rows
Columns
Relationships
Constraints
Indexes
Transactions
```

---

# 3. Why Use a Relational Database?

Good candidates include:

```text
Orders
Payments
Users
Products
Inventory
Financial records
```

when you need:

```text
Structured relationships
Transactions
Constraints
Complex queries
Strong consistency requirements
```

---

# 4. Database as Source of Truth

For an e-commerce system:

```text
MySQL
  ↓
Authoritative order state
```

Redis might contain:

```text
Cached order data
```

but MySQL is the durable source of truth.

This separation is important.

---

# 5. Schema Design

Suppose we have:

```text
User
Product
Order
OrderItem
```

A simple relational model:

```text
User
 |
 +--- Order
       |
       +--- OrderItem
               |
               +--- Product
```

---

# 6. Normalization

Normalization reduces unnecessary duplication and update anomalies.

Example:

Instead of storing:

```text
Order
-------------------------
order_id
customer_name
customer_email
product_name
product_price
```

you can separate:

```text
Customer
Product
Order
OrderItem
```

This improves consistency.

---

# 7. Denormalization

Sometimes duplicated data is intentional for performance.

Example:

```text
Order
----------------
order_id
customer_id
customer_name
```

Even though customer name exists in:

```text
Customer
```

This can make reads simpler or preserve historical information.

Denormalization is a trade-off.

---

# 8. Normalization vs Denormalization

### Normalization

Benefits:

```text
Less duplication
Better consistency
Easier updates
```

Trade-offs:

```text
More joins
Potentially more complex reads
```

### Denormalization

Benefits:

```text
Faster/simple reads
Fewer joins
```

Trade-offs:

```text
Duplicate data
More complicated updates
Potential inconsistency
```

---

# 9. Primary Key

A primary key uniquely identifies a row.

Example:

```sql
CREATE TABLE product (
    id BIGINT PRIMARY KEY,
    name VARCHAR(255),
    price DECIMAL(10,2)
);
```

The key should be:

```text
Unique
Stable
Efficient
```

---

# 10. Surrogate Key

A surrogate key is an artificial identifier.

Example:

```text
BIGINT AUTO_INCREMENT
```

or:

```text
UUID
```

It doesn't represent business meaning.

---

# 11. Natural Key

A natural key comes from business data.

Example:

```text
email
ISBN
country_code
```

Natural keys can be useful, but business attributes may change.

A common approach is:

```text
Surrogate primary key
+
Unique constraint on natural identifier
```

---

# 12. UUID vs Numeric ID

### Numeric ID

Advantages:

```text
Compact
Efficient indexes
Simple
```

Potential concern:

```text
Sequential IDs can reveal volume/order
```

### UUID

Advantages:

```text
Globally unique
Can be generated independently
Useful across distributed systems
```

Trade-offs:

```text
Larger
Index/storage overhead
Random UUIDs can affect locality
```

The choice depends on the system.

---

# 13. Indexes

An index helps the database find rows efficiently.

Without an appropriate index:

```text
Query
 ↓
Scan many rows
```

With an index:

```text
Query
 ↓
Index
 ↓
Relevant rows
```

---

# 14. Example Index

Suppose:

```sql
SELECT *
FROM orders
WHERE user_id = 42;
```

An index can help:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

---

# 15. Why Not Index Every Column?

Indexes have costs:

```text
Storage
Write overhead
Memory
Maintenance
```

Every insert/update may need to update affected indexes.

So:

```text
More indexes
≠
Always faster
```

---

# 16. Composite Index

Suppose:

```sql
SELECT *
FROM orders
WHERE user_id = 42
  AND status = 'PAID';
```

A composite index may help:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

---

# 17. Column Order Matters

For:

```text
(user_id, status)
```

the index is naturally useful for queries starting with:

```text
user_id
```

The best column order depends on:

```text
Query patterns
Selectivity
Sorting
Filtering
```

Don't memorize one universal ordering rule.

---

# 18. Covering Index

A covering index contains all columns required for a particular query.

Example:

```sql
SELECT user_id, status
FROM orders
WHERE user_id = 42;
```

An index containing the needed columns may allow the database to answer the query with less table access.

This can improve performance for suitable workloads.

---

# 19. Query Plan

When a query is slow, inspect the execution plan.

In MySQL:

```sql
EXPLAIN
SELECT *
FROM orders
WHERE user_id = 42;
```

Look for things such as:

```text
Index usage
Rows examined
Join strategy
Sort operations
Full scans
```

---

# 20. Full Table Scan

Example:

```text
1 million rows
```

Query:

```sql
SELECT *
FROM users
WHERE email = 'abc@example.com';
```

Without an appropriate index:

```text
Potentially scan many rows
```

An index on:

```text
email
```

can dramatically reduce work.

---

# 21. Selectivity

An index is often more useful when the condition narrows the result set significantly.

Example:

```text
user_id = 42
```

may be selective.

A boolean:

```text
is_active = true
```

might be less selective if most rows are active.

But the actual usefulness depends on:

```text
Data distribution
Query
Indexes
Optimizer
```

---

# 22. Transactions

A transaction groups operations into a logical unit.

Example:

```text
Create order
Reduce inventory
Create order items
```

These may need coordinated transactional behavior.

---

# 23. ACID

### Atomicity

All operations succeed or the transaction rolls back.

### Consistency

The transaction preserves defined data constraints/invariants.

### Isolation

Concurrent transactions should behave according to the selected isolation level.

### Durability

Committed data survives appropriate failures.

---

# 24. Transaction Example

Suppose:

```text
Account A:
₹10,000

Account B:
₹5,000
```

Transfer:

```text
A - ₹1,000
B + ₹1,000
```

If only A is updated:

```text
A = ₹9,000
B = ₹5,000
```

The transaction should prevent an invalid partial state.

---

# 25. Isolation Levels

Common SQL isolation levels:

```text
Read Uncommitted
Read Committed
Repeatable Read
Serializable
```

MySQL/InnoDB commonly uses:

```text
REPEATABLE READ
```

as its default isolation level, though applications should verify the exact database/version/configuration.

---

# 26. Read Uncommitted

Allows the possibility of:

```text
Dirty reads
```

A transaction can potentially observe changes that another transaction hasn't committed.

Rarely appropriate for critical business operations.

---

# 27. Read Committed

A query generally sees only committed data.

Can prevent:

```text
Dirty reads
```

But repeated reads may return different committed values if another transaction commits changes between them.

---

# 28. Repeatable Read

Within a transaction, repeated reads can provide a consistent snapshot according to the database's implementation.

MySQL/InnoDB uses MVCC to support this isolation behavior.

---

# 29. Serializable

Provides the strongest standard isolation level.

Conceptually:

```text
Concurrent transactions
 ↓
Behave closer to serial execution
```

But stronger isolation can reduce concurrency and increase contention.

---

# 30. Isolation Trade-off

Generally:

```text
Higher isolation
→ stronger guarantees
→ potentially lower concurrency
```

Choose the weakest isolation level that correctly satisfies the business requirement.

---

# 31. Database Connection Pool

Spring Boot commonly uses:

```text
HikariCP
```

Instead of creating a new DB connection for every request.

Flow:

```text
Application
    ↓
Connection Pool
 /  |  |  \
DB connections
    ↓
MySQL
```

---

# 32. Why Connection Pooling?

Creating database connections is relatively expensive.

Pooling provides:

```text
Connection reuse
Lower connection overhead
Controlled concurrency
```

---

# 33. Connection Pool Sizing

Suppose:

```text
10 app instances
```

Each has:

```text
20 DB connections
```

Potential total:

```text
200 connections
```

If you scale to:

```text
100 instances
```

you could reach:

```text
2,000 connections
```

The database may become overloaded.

---

# 34. Important Scaling Lesson

Application horizontal scaling can create database connection pressure.

Therefore monitor:

```text
Active connections
Idle connections
Connection wait time
Query latency
Database CPU
```

---

# 35. Read Replication

A primary database handles writes:

```text
Primary
```

Replicas copy data:

```text
Primary
  |
  +--- Replica1
  +--- Replica2
  +--- Replica3
```

Suitable read traffic can be sent to replicas.

---

# 36. Why Use Read Replicas?

Benefits:

```text
Read scaling
Reduced primary read load
High availability options
Disaster recovery
```

But replication introduces:

```text
Replication lag
Routing complexity
Failover complexity
```

---

# 37. Replication Lag

Suppose:

```text
Write → Primary
```

then immediately:

```text
Read → Replica
```

The replica may not have received the change yet.

So:

```text
Write:
status = PAID

Replica:
status = PENDING
```

temporarily.

---

# 38. Read-After-Write Consistency

If a user performs:

```text
POST /orders
```

and immediately:

```text
GET /orders/123
```

they usually expect to see the newly created order.

If reads go to replicas:

```text
Replica lag
```

could violate that expectation.

Potential strategies:

```text
Read from primary temporarily
Session/route-aware reads
Wait for replication
Use stronger consistency mechanisms
```

---

# 39. Database Failover

If the primary fails:

```text
Primary
   X
```

another node may become the new primary.

This requires:

```text
Failure detection
Leader election / managed failover
Replication
Client reconfiguration
```

Exact mechanisms depend on the database/platform.

---

# 40. Database Partitioning

Partitioning divides a table into smaller logical pieces.

Example by date:

```text
orders_2026_01
orders_2026_02
orders_2026_03
```

The exact implementation depends on the database.

Potential benefits:

```text
Manageability
Partition pruning
Large-table maintenance
Archival
```

---

# 41. Horizontal Partitioning

Rows are distributed into partitions.

Example:

```text
Users 1–1M
→ Partition A

Users 1M–2M
→ Partition B
```

This is often associated with:

```text
Sharding
```

when partitions are placed across separate database nodes.

---

# 42. Vertical Partitioning

Split columns into separate tables or storage groups.

Example:

```text
User core data
+
User profile data
```

Potentially useful when:

```text
Some fields are rarely accessed
Rows are very wide
Different access patterns exist
```

---

# 43. Sharding

Sharding distributes data across multiple database nodes.

Example:

```text
Shard 1
Users A–F

Shard 2
Users G–M

Shard 3
Users N–Z
```

The application or database routing layer determines where data belongs.

---

# 44. Why Shard?

Potential benefits:

```text
More storage capacity
More write capacity
More read capacity
Smaller datasets per node
```

But sharding adds significant complexity.

---

# 45. Sharding Key

A shard key determines where data goes.

Examples:

```text
user_id
tenant_id
region
```

A good key should ideally provide:

```text
Even distribution
Stable routing
High cardinality
Good query locality
```

---

# 46. Bad Shard Key

Suppose:

```text
country
```

and:

```text
80% of users are in one country
```

Then:

```text
Shard A → 80%
Shard B → 10%
Shard C → 10%
```

This creates a hotspot.

---

# 47. Hash Sharding

Conceptually:

```text
hash(user_id) % N
```

determines the shard.

Advantages:

```text
Usually better distribution
```

Problem:

```text
Changing N
```

can cause many keys to move.

Consistent hashing can reduce movement in suitable architectures.

---

# 48. Range Sharding

Example:

```text
1–1M → Shard A
1M–2M → Shard B
2M–3M → Shard C
```

Easy to understand.

Potential problem:

```text
Recent IDs may receive most traffic
```

creating a hot shard.

---

# 49. Hot Partition

Suppose:

```text
Shard 1 → 80% traffic
Shard 2 → 10%
Shard 3 → 10%
```

Even though total capacity looks sufficient:

```text
Shard 1
→ overloaded
```

This is a common sharding problem.

---

# 50. Cross-Shard Queries

Suppose:

```text
Orders are sharded by user_id.
```

Query:

```text
Find all orders created today.
```

may require:

```text
Query every shard
```

This is expensive.

Therefore:

> Shard key selection should consider the most important query patterns.

---

# 51. Cross-Shard Transactions

Transactions across multiple shards are much harder than transactions within one shard.

Example:

```text
Shard A
+
Shard B
```

A transaction spanning both may require distributed transaction mechanisms or application-level workflows.

This is one reason sharding should not be introduced casually.

---

# 52. Database Federation

Another approach is separating databases by domain.

Example:

```text
User DB
Order DB
Product DB
Payment DB
```

This is common in service-oriented architectures.

It can improve:

```text
Ownership
Independent scaling
Isolation
```

but introduces:

```text
Cross-database consistency
Distributed queries
Operational complexity
```

---

# 53. Database-per-Service

In microservices, a service may own its database.

Example:

```text
Order Service
    ↓
Order DB

Product Service
    ↓
Product DB
```

This improves service ownership.

But:

```text
Order Service
```

shouldn't directly query:

```text
Product DB
```

in a tightly coupled design.

Instead use:

```text
API
Events
Replicated read models
```

depending on the requirement.

---

# 54. N+1 Query Problem

Suppose:

```text
SELECT 100 orders
```

Then the application runs:

```text
100 product queries
```

Total:

```text
101 queries
```

This can destroy performance.

Potential solutions:

```text
JOIN
Batch query
Fetch strategy
Projection
Caching
```

depending on the use case.

---

# 55. Pagination

Never load millions of records at once.

Bad:

```sql
SELECT *
FROM orders;
```

for a huge dataset.

Better:

```sql
LIMIT 20
```

with an appropriate pagination strategy.

---

# 56. Offset Pagination

Example:

```sql
SELECT *
FROM orders
ORDER BY id
LIMIT 20 OFFSET 10000;
```

Simple.

But deep offsets can become expensive.

---

# 57. Cursor / Keyset Pagination

Instead of:

```text
OFFSET 10000
```

use a cursor such as:

```text
last_seen_id = 10000
```

Then:

```sql
SELECT *
FROM orders
WHERE id > 10000
ORDER BY id
LIMIT 20;
```

This can be more efficient for large datasets when the ordering/index supports it.

---

# 58. Database Constraints

Constraints protect data integrity.

Examples:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
```

Use the database to enforce important invariants where appropriate.

---

# 59. Foreign Keys

Example:

```text
OrderItem
   ↓
Product
```

A foreign key can enforce referential integrity.

But at very large scale, some distributed architectures may intentionally manage relationships at the application level.

This is a trade-off.

---

# 60. Unique Constraints

Example:

```sql
UNIQUE(email)
```

This prevents duplicate emails at the database level.

Do not rely only on:

```text
"Check first, then insert"
```

because concurrent requests can race.

---

# 61. Optimistic Locking

Useful when multiple transactions may update the same record.

Example:

```text
version = 5
```

Application reads:

```text
version = 5
```

Another transaction updates:

```text
version = 6
```

First transaction tries to update:

```text
WHERE id = 101
AND version = 5
```

No row matches.

The update fails instead of silently overwriting the newer change.

---

# 62. Pessimistic Locking

The database locks rows so other transactions cannot modify them in conflicting ways during the lock.

Useful for certain:

```text
Inventory
Financial operations
Highly contended records
```

But locks can cause:

```text
Contention
Deadlocks
Lower concurrency
```

---

# 63. Deadlocks

A deadlock occurs when transactions wait on each other's locks.

Example:

```text
Transaction A:
locks Row 1
waits for Row 2

Transaction B:
locks Row 2
waits for Row 1
```

Neither can proceed.

The database detects the deadlock and typically aborts one transaction.

---

# 64. Preventing Deadlocks

Common practices:

```text
Consistent lock ordering
Short transactions
Appropriate indexes
Avoid unnecessary locks
Retry aborted transactions where appropriate
```

---

# 65. Database Transactions and External Services

A database transaction cannot normally include an external HTTP call atomically.

Bad pattern:

```text
BEGIN
 ↓
Update DB
 ↓
Call Payment API
 ↓
Payment hangs
 ↓
Transaction remains open
```

This can hold database resources unnecessarily.

Prefer:

```text
Short DB transaction
+
Asynchronous workflow / state machine
```

for complex distributed workflows.

---

# 66. Outbox Pattern Preview

Suppose:

```text
Create order in DB
Publish event to Kafka
```

If DB succeeds but Kafka fails:

```text
Order exists
Event missing
```

The outbox pattern stores the event in the same DB transaction:

```text
Transaction
 ├── Order
 └── Outbox Event
```

A separate process publishes the outbox event.

We'll study this more in the messaging/distributed-systems section.

---

# 67. Database Backup

Production databases need:

```text
Backups
Recovery testing
Retention
Monitoring
```

A backup that has never been restored is not enough.

Test:

```text
Can we actually recover?
```

---

# 68. RPO and RTO

### RPO

How much data loss is acceptable?

```text
RPO = 5 minutes
```

### RTO

How quickly must service recover?

```text
RTO = 30 minutes
```

Database architecture should support the required targets.

---

# 69. Database Monitoring

Monitor:

```text
CPU
Memory
Disk
IOPS
Connections
Query latency
Slow queries
Locks
Deadlocks
Replication lag
Cache/buffer hit ratio
Storage growth
```

---

# 70. Slow Query Log

For MySQL, slow-query logging can identify expensive queries.

Use it together with:

```text
EXPLAIN
```

and application metrics.

Don't optimize queries based only on intuition.

---

# 71. Connection Pool vs Database Capacity

Suppose:

```text
App:
20 connections

DB:
maximum 100 connections
```

If you run:

```text
10 instances
```

you may request:

```text
200 connections
```

The database cannot accommodate them all.

This is a classic scaling mistake.

---

# 72. Database Scaling Strategy

A practical progression:

```text
1. Correct schema
2. Good indexes
3. Optimize queries
4. Connection pooling
5. Cache
6. Vertical DB scaling
7. Read replicas
8. Partitioning
9. Sharding
```

This isn't a rigid sequence.

Measure before escalating complexity.

---

# 73. Database Bottleneck Scenario

### Situation:

```text
App CPU = 30%
DB CPU = 95%
```

Likely:

```text
Database bottleneck
```

Investigate:

```text
Slow queries
Indexes
Connections
Locks
Cache
Read replicas
```

---

# 74. Database Bottleneck Scenario

### Situation:

```text
DB CPU = 30%
App CPU = 95%
```

Likely:

```text
Application bottleneck
```

Investigate:

```text
Algorithms
Serialization
GC
CPU-heavy work
Thread pools
```

Don't automatically change the database.

---

# 75. Read-Heavy System

Suppose:

```text
90% reads
10% writes
```

Potential architecture:

```text
App
 ↓
Redis
 ↓ miss
Read Replicas
 ↓
Primary
```

But route reads according to consistency requirements.

---

# 76. Write-Heavy System

Suppose:

```text
90% writes
10% reads
```

Potential approaches:

```text
Batch writes
Queue processing
Partitioning
Sharding
Write optimization
```

The exact design depends on the workload.

---

# 77. Database Design for E-commerce

Core tables:

```text
users
products
categories
carts
cart_items
orders
order_items
payments
inventory
```

Potential relationships:

```text
User
 ↓
Order
 ↓
OrderItem
 ↓
Product
```

---

# 78. Product Read Path

A common design:

```text
Client
 ↓
Product API
 ↓
Redis
 ↓ miss
MySQL
```

Product data is often read much more frequently than it changes.

---

# 79. Order Write Path

A simplified transactional path:

```text
Client
 ↓
Order Service
 ↓
Validate
 ↓
Create Order
 ↓
Create Order Items
 ↓
Commit
```

Then asynchronous work can happen separately:

```text
Order Created
 ↓
Event
 ↓
Notification
Inventory
Analytics
```

---

# 80. Inventory

Inventory is more challenging because:

```text
Many users
+
Same product
+
Limited quantity
```

can cause concurrency issues.

Potential techniques:

```text
Atomic update
Optimistic locking
Pessimistic locking
Reservation
Queue
```

depending on requirements.

---

# 81. Example Inventory Update

Instead of:

```text
SELECT stock
UPDATE stock
```

with a race between the two, an atomic conditional update can be used:

```sql
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = 101
  AND quantity > 0;
```

Then check affected rows.

If:

```text
1 row updated
```

reservation succeeded.

If:

```text
0 rows updated
```

stock wasn't available.

This can avoid some race conditions.

---

# 82. Database Scaling Trade-offs

### Replicas

```text
+ Read scaling
- Replication lag
```

### Sharding

```text
+ Massive scale
- Operational complexity
```

### Caching

```text
+ Low latency
- Staleness/invalidation
```

### Denormalization

```text
+ Faster reads
- Duplicate data
```

---

# 83. Don't Shard Too Early

Sharding adds:

```text
Routing
Cross-shard queries
Rebalancing
Operational complexity
Backup complexity
Migration complexity
```

If:

```text
MySQL + indexes + cache + replicas
```

handles the workload:

```text
Don't shard.
```

---

# 84. Database Design Interview Framework

When asked to design the database:

```text
1. Identify entities
2. Identify relationships
3. Define access patterns
4. Choose schema
5. Add important constraints
6. Add indexes
7. Estimate data volume
8. Estimate read/write load
9. Consider caching
10. Consider replicas
11. Consider partitioning/sharding only if required
```

---

# 85. Interview Question

### How do you scale a MySQL database?

Answer:

> "I'd first optimize the schema, queries and indexes. Then I'd consider caching and read replicas for read-heavy workloads. If the data or write volume grows beyond a single node's capacity, I'd evaluate partitioning or sharding based on the access patterns."

---

# 86. Interview Question

### Why not create an index on every column?

Answer:

> "Indexes consume storage and increase write and maintenance cost. I'd create indexes based on actual query patterns and verify their effectiveness using execution plans."

---

# 87. Interview Question

### What is replication lag?

Answer:

> "It's the delay between a write being committed on the primary and that change becoming visible on a replica. It can cause stale reads."

---

# 88. Interview Question

### How do you handle read-after-write consistency?

Answer:

> "For operations that require immediate visibility, I can route the relevant read to the primary or use an architecture that guarantees the required consistency. I wouldn't blindly send every read to replicas."

---

# 89. Interview Question

### When would you shard a database?

Answer:

> "Only when a single database node cannot handle the required data volume or throughput after simpler optimizations, caching, scaling and replication have been considered."

---

# 90. Interview Question

### What makes a good shard key?

Answer:

> "It should distribute data and traffic evenly, remain stable, have sufficient cardinality and support important query patterns."

---

# 91. Interview Question

### What is a hot shard?

Answer:

> "It's a shard receiving disproportionately high traffic or data compared with the others, causing uneven resource utilization."

---

# 92. Interview Question

### What is optimistic locking?

Answer:

> "It detects conflicting updates using a version or timestamp rather than holding a database lock for the whole operation."

---

# 93. Interview Question

### What is pessimistic locking?

Answer:

> "It locks the relevant database rows so conflicting transactions can't modify them concurrently until the lock is released."

---

# 94. Interview Question

### How do you prevent duplicate records?

Answer:

> "I use database constraints such as unique indexes in addition to application-level validation. The database constraint is important because concurrent requests can race."

---

# 95. Interview Question

### How do you handle millions of records in an API?

Answer:

> "I use pagination, preferably cursor/keyset pagination for large datasets where appropriate, and make sure the ordering and filtering fields are properly indexed."

---

# 96. Interview Question

### What is the N+1 query problem?

Answer:

> "It's when one query loads a collection and then the application performs an additional query for each item. This can turn one logical operation into hundreds or thousands of database queries."

---

# 97. Interview Question

### How do you debug a slow SQL query?

Answer:

> "I'd check the query execution plan, indexes, rows examined, joins and filters, and look for full scans or expensive sorting. I'd also check database load and query latency in production."

---

# 98. Practical Scenario

### 100 million orders

Queries:

```text
Get orders by user
Get order by ID
Get orders by date
```

Possible indexes:

```text
(order_id)
(user_id, created_at)
(created_at)
```

The exact indexes should be validated against real query patterns.

---

# 99. Practical Scenario

### Database CPU is 90%

Don't immediately shard.

First:

```text
Check slow queries
Check indexes
Check query plans
Check connection count
Check locks
Check cache
```

Then:

```text
Scale vertically
Read replicas
Partition
Shard
```

as justified.

---

# 100. Practical Scenario

### Database storage keeps growing

Think:

```text
Retention
Archival
Partitioning
Compression
Object storage
Data lifecycle
```

Don't necessarily keep every historical record in the hot transactional database forever.

---

# 101. Practical Scenario

### Product search is slow

Don't necessarily add more MySQL servers.

If requirements include:

```text
Full-text search
Fuzzy matching
Ranking
Facets
```

consider a search engine such as:

```text
Elasticsearch
OpenSearch
```

with the database remaining the source of truth where appropriate.

---

# 102. Practical Scenario

### Application has 100 instances

Each:

```text
20 DB connections
```

Total:

```text
2,000 connections
```

The first question:

```text
Can the database handle 2,000 connections?
```

If not:

```text
Connection pooling architecture
Proxy/pooler
Reduce per-instance pool
Scale DB
```

may need consideration.

---

# 103. Database System Design Mental Model

Remember:

```text
Application
     ↓
Connection Pool
     ↓
Cache
     ↓
Database
     ↓
Replica / Partition / Shard
```

At every level ask:

```text
What is the bottleneck?
```

---

# 104. Final Checklist

You should be able to explain:

```text
□ Relational database
□ Source of truth
□ Normalization
□ Denormalization
□ Primary/natural/surrogate keys
□ UUID vs numeric IDs
□ Indexes
□ Composite indexes
□ Covering indexes
□ EXPLAIN
□ ACID
□ Transactions
□ Isolation levels
□ HikariCP
□ Connection pool sizing
□ Replication
□ Read replicas
□ Replication lag
□ Read-after-write consistency
□ Partitioning
□ Sharding
□ Shard keys
□ Hot shards
□ Cross-shard queries
□ Optimistic locking
□ Pessimistic locking
□ Deadlocks
□ Pagination
□ N+1 queries
□ Constraints
□ Backups
□ RPO/RTO
□ Database monitoring
□ Database scaling strategy
```

---

# 105. One-Minute Interview Answer

### "How would you scale the database of an e-commerce backend?"

> "I'd start by optimizing the schema, indexes and slow queries because scaling a bad query doesn't solve the root problem. For read-heavy traffic I'd add Redis caching and potentially read replicas. I'd monitor connection pools and replication lag as the application scales horizontally. If the data or write workload eventually exceeds a single database's capacity, I'd evaluate partitioning or sharding based on the access patterns and choose a shard key that distributes both data and traffic evenly."

---

# 106. Key Takeaway

> **Database scaling should be driven by workload and access patterns. Optimize first, cache where appropriate, replicate reads when useful, and introduce partitioning or sharding only when simpler approaches can no longer meet the requirements.**

**File 06 complete.**
