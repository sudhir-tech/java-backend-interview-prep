# SQL — Indexes & Query Performance

Indexes are one of the most important SQL topics for backend interviews.

For a Java/Spring Boot developer, you should understand:

```text
Why indexes exist
How indexes work
When to create them
When indexes hurt performance
Composite indexes
Index order
EXPLAIN
Slow queries
Full table scans
Pagination
```

---

# 1. What Is an Index?

An index is a data structure that helps the database find rows faster.

Think of a book.

Without an index:

```text
Read every page
↓
Find the topic
```

With an index:

```text
Look up the topic
↓
Jump to the relevant page
```

A database index works similarly.

---

# 2. Basic Example

Suppose:

```sql
SELECT *
FROM users
WHERE email = 'sudhir@example.com';
```

If `email` is not indexed, the database may need to inspect many rows.

If an index exists:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

the database can use the index to locate matching rows more efficiently.

---

# 3. Why Indexes Improve Reads

Imagine one million users.

Query:

```sql
SELECT *
FROM users
WHERE email = 'abc@example.com';
```

Without a suitable index, the database may scan a large portion of the table.

With:

```text
idx_users_email
```

the database can quickly narrow down the matching row.

---

# 4. Indexes Are Not Free

Indexes improve certain reads but introduce costs.

Every index consumes:

```text
Disk space
Memory/cache
Insert work
Update work
Delete work
Maintenance
```

When a row changes, relevant indexes may also need updating.

So:

```text
More indexes ≠ always better
```

---

# 5. Basic Index Creation

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Naming convention:

```text
idx_<table>_<column>
```

Example:

```text
idx_orders_user_id
idx_products_category_id
```

---

# 6. UNIQUE Index

A unique index enforces uniqueness.

Example:

```sql
CREATE UNIQUE INDEX idx_users_email_unique
ON users(email);
```

This means duplicate email values are not allowed.

A `UNIQUE` constraint may also cause the database to create a unique index internally, depending on the database.

---

# 7. Primary Key Index

When you define:

```sql
PRIMARY KEY (id)
```

the database generally creates or uses an index to enforce uniqueness and support primary-key lookups.

Example:

```sql
SELECT *
FROM users
WHERE id = 100;
```

Primary-key lookups are therefore typically very efficient.

---

# 8. Foreign Key Index

Suppose:

```sql
orders.user_id
```

references:

```text
users.id
```

A useful index is often:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

This can help with:

```sql
SELECT *
FROM orders
WHERE user_id = 100;
```

and joins:

```sql
SELECT *
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

The exact indexing strategy depends on the database and workload.

---

# 9. Index for WHERE

Common example:

```sql
SELECT *
FROM products
WHERE category_id = 10;
```

Potential index:

```sql
CREATE INDEX idx_products_category_id
ON products(category_id);
```

---

# 10. Index for ORDER BY

Suppose:

```sql
SELECT *
FROM products
ORDER BY created_at DESC;
```

An index on:

```sql
created_at
```

may help the database avoid expensive sorting in appropriate query plans.

```sql
CREATE INDEX idx_products_created_at
ON products(created_at);
```

The optimizer decides whether using the index is actually beneficial.

---

# 11. Index for WHERE + ORDER BY

Example:

```sql
SELECT *
FROM orders
WHERE user_id = 100
ORDER BY created_at DESC;
```

A composite index may be useful:

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at);
```

This is an important backend optimization pattern.

---

# 12. Composite Index

A composite index contains multiple columns.

Example:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

The index is based on:

```text
user_id
status
```

in that order.

---

# 13. Column Order Matters

Consider:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

This is generally useful for queries filtering by:

```text
user_id
```

or:

```text
user_id + status
```

But it may not be the best index for:

```text
status only
```

This is related to the:

```text
Leftmost Prefix
```

principle.

---

# 14. Leftmost Prefix

Index:

```sql
(user_id, status, created_at)
```

Conceptually supports prefixes such as:

```text
user_id
user_id + status
user_id + status + created_at
```

But a query only filtering on:

```text
status
```

does not generally get the same benefit from this index.

Database optimizers and index types can have additional nuances, so always verify with `EXPLAIN`.

---

# 15. Example

Index:

```sql
CREATE INDEX idx_orders_user_status_created
ON orders(user_id, status, created_at);
```

Good candidate:

```sql
WHERE user_id = 10
```

Good candidate:

```sql
WHERE user_id = 10
AND status = 'PAID'
```

Good candidate:

```sql
WHERE user_id = 10
AND status = 'PAID'
ORDER BY created_at DESC
```

Potentially poor candidate:

```sql
WHERE status = 'PAID'
```

because the first indexed column is not being used as a leading filter.

---

# 16. Equality Before Range

A common composite-index design guideline is to place columns used for equality conditions before columns used for range conditions.

Example query:

```sql
SELECT *
FROM orders
WHERE user_id = 10
AND created_at >= '2026-01-01';
```

Potential index:

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at);
```

Here:

```text
user_id
→ equality

created_at
→ range
```

This can be a strong index design.

Do not treat this as an absolute rule; verify with the database's optimizer and workload.

---

# 17. Index Selectivity

Selectivity describes how well a column distinguishes rows.

Example:

```text
user_id
```

may have many different values.

A boolean:

```text
is_active
```

may have only:

```text
true
false
```

An index on a low-cardinality column alone may provide limited benefit for some workloads.

But low-cardinality columns can still be useful as part of a composite index.

---

# 18. Cardinality

Cardinality refers to the number of distinct values in a column or result.

Example:

```text
country
```

might have relatively few distinct values.

```text
email
```

may have almost one distinct value per row.

High-cardinality columns often provide stronger filtering power, but index usefulness depends on the actual query and data distribution.

---

# 19. Full Table Scan

A full table scan means the database examines rows across the table to find matching data.

Example:

```sql
SELECT *
FROM users
WHERE LOWER(name) = 'sudhir';
```

Depending on the database and indexes, this may prevent use of a normal index on:

```text
name
```

because the query applies a function.

---

# 20. Index Scan / Lookup

When an index can efficiently identify matching rows, the database can use the index rather than scanning the whole table.

Conceptually:

```text
Index
 ↓
matching row locations
 ↓
table rows
```

The exact execution strategy depends on the database engine.

---

# 21. EXPLAIN

`EXPLAIN` shows how the database plans to execute a query.

Example:

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'abc@example.com';
```

It can help answer:

```text
Is an index being used?
How many rows might be examined?
Which access method is chosen?
Are joins expensive?
Is sorting required?
```

---

# 22. EXPLAIN ANALYZE

Some databases support:

```sql
EXPLAIN ANALYZE
```

This can execute the query and provide actual execution statistics.

For example:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE user_id = 100;
```

This is extremely useful when diagnosing real performance problems.

Be careful with `EXPLAIN ANALYZE` for:

```text
INSERT
UPDATE
DELETE
```

because it may actually execute the statement depending on the database.

---

# 23. EXPLAIN in MySQL

For MySQL, a common command is:

```sql
EXPLAIN
SELECT *
FROM orders
WHERE user_id = 100;
```

You may see columns such as:

```text
id
select_type
table
type
possible_keys
key
rows
Extra
```

---

# 24. What Is the key Column?

In MySQL `EXPLAIN`, `key` can show the index the optimizer selected.

Example:

```text
key = idx_orders_user_id
```

This suggests that index is being used for that table access.

---

# 25. What Is possible_keys?

`possible_keys` indicates indexes that could potentially be used for the query.

Important:

```text
possible_keys
```

does not mean the database actually selected them.

Check:

```text
key
```

to see the selected index in a standard MySQL EXPLAIN output.

---

# 26. What Is rows in EXPLAIN?

`rows` is an estimate of how many rows the optimizer expects to examine for that table access.

Smaller is often better, but:

```text
estimated rows ≠ actual rows
```

unless using execution analysis that reports actual statistics.

---

# 27. Query That Cannot Use a Normal Index Efficiently

Example:

```sql
SELECT *
FROM users
WHERE YEAR(created_at) = 2026;
```

The function is applied to the column.

A better range query can be:

```sql
SELECT *
FROM users
WHERE created_at >= '2026-01-01'
  AND created_at < '2027-01-01';
```

This can allow a normal index on `created_at` to be used more effectively.

---

# 28. Function on Indexed Column

Potentially problematic:

```sql
WHERE LOWER(email) = 'abc@example.com'
```

Potentially better:

```sql
WHERE email = 'abc@example.com'
```

if emails are already normalized.

Another solution may be:

```text
functional/expression index
```

if supported by the database.

---

# 29. Leading Wildcard

Query:

```sql
SELECT *
FROM users
WHERE name LIKE '%sudhir%';
```

A normal B-tree index on `name` generally cannot efficiently seek to the beginning of a value because the pattern starts with `%`.

Compare:

```sql
WHERE name LIKE 'Sudhir%'
```

which can be much more index-friendly for a normal B-tree index.

Exact behavior depends on database and collation.

---

# 30. Leading Wildcard Problem

Think:

```text
'Sudhir%'
```

The database knows the beginning:

```text
Sudhir...
```

But:

```text
'%Sudhir%'
```

could match almost anywhere.

For large-scale text search, consider:

```text
Full-text search
Search engine
Database-specific text indexes
```

depending on requirements.

---

# 31. SELECT * and Performance

Avoid blindly using:

```sql
SELECT *
```

in performance-sensitive application queries.

Instead:

```sql
SELECT id, name, email
FROM users
WHERE email = ?;
```

Benefits can include:

```text
Less data transferred
Less application mapping work
Potentially better index-only/covering plans
Clearer API/data requirements
```

This does not mean `SELECT *` is always bad.

---

# 32. Covering Index

An index is sometimes called covering for a query when it contains all the columns needed to answer that query, allowing the database to avoid looking up the base table.

Example:

```sql
SELECT user_id, status
FROM orders
WHERE user_id = 10
AND status = 'PAID';
```

Index:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

The database may be able to answer the query directly from the index.

This depends on the database engine and execution plan.

---

# 33. Why Too Many Indexes Are Bad

Suppose a table has:

```text
10 indexes
```

Every insert may require maintaining multiple index structures.

This increases:

```text
Write cost
Storage
Memory pressure
Maintenance
```

So index only for actual query patterns.

---

# 34. Indexes and INSERT

When inserting:

```sql
INSERT INTO users (...)
VALUES (...);
```

the database may need to update:

```text
Primary-key index
Unique indexes
Other indexes
```

Therefore:

```text
More indexes
→ potentially slower writes
```

---

# 35. Indexes and UPDATE

If you update an indexed column:

```sql
UPDATE users
SET email = 'new@example.com'
WHERE id = 10;
```

the relevant index entry may need to be changed.

Updating non-indexed columns may avoid some index maintenance.

---

# 36. Indexes and DELETE

When deleting a row:

```sql
DELETE FROM users
WHERE id = 10;
```

the database must remove the corresponding entries from relevant indexes.

Foreign-key checks may also be involved.

---

# 37. Indexing and Joins

Suppose:

```sql
SELECT *
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

Useful indexes often include:

```text
users.id
orders.user_id
```

The primary key on `users.id` is normally already indexed.

An index on:

```sql
orders(user_id)
```

can help the database find orders belonging to users.

---

# 38. Indexing WHERE + JOIN

Example:

```sql
SELECT *
FROM users u
JOIN orders o
    ON o.user_id = u.id
WHERE o.status = 'PAID';
```

Potential index:

```sql
CREATE INDEX idx_orders_status_user
ON orders(status, user_id);
```

But whether this is optimal depends on:

```text
Data distribution
Query frequency
Join order
Other filters
Existing indexes
Execution plan
```

Do not create indexes by blindly following a formula.

---

# 39. Composite Index Example

Query:

```sql
SELECT *
FROM orders
WHERE user_id = 100
AND status = 'PAID'
ORDER BY created_at DESC;
```

Potential index:

```sql
CREATE INDEX idx_orders_user_status_created
ON orders(user_id, status, created_at);
```

This aligns the index with:

```text
equality filters
+
ordering
```

The optimizer still decides whether it is useful.

---

# 40. Pagination Problem

Offset pagination:

```sql
SELECT *
FROM products
ORDER BY id
LIMIT 20 OFFSET 100000;
```

For large offsets, the database may need to process/skip many rows before returning the requested page.

This can become expensive.

---

# 41. Keyset Pagination

Instead of:

```sql
LIMIT 20 OFFSET 100000
```

use the last seen key:

```sql
SELECT *
FROM products
WHERE id > 100000
ORDER BY id
LIMIT 20;
```

This is commonly called:

```text
Keyset pagination
Cursor pagination
```

A suitable index on `id` helps.

---

# 42. Keyset Pagination with Created Time

If ordering by:

```text
created_at
```

alone is not unique, use a tie-breaker.

Example:

```sql
SELECT *
FROM products
WHERE (created_at, id) < ('2026-08-20 10:00:00', 500)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

A matching composite index may be useful:

```sql
CREATE INDEX idx_products_created_id
ON products(created_at, id);
```

Exact syntax and optimization depend on the database.

---

# 43. Index and Sorting

Suppose:

```sql
SELECT *
FROM products
ORDER BY created_at DESC
LIMIT 20;
```

An index on:

```text
created_at
```

may allow the database to retrieve the first rows without sorting the entire table.

But the optimizer considers:

```text
table size
selectivity
cost of scanning index
cost of table lookups
```

---

# 44. Index and GROUP BY

Example:

```sql
SELECT user_id, COUNT(*)
FROM orders
GROUP BY user_id;
```

An index on:

```sql
orders(user_id)
```

may help depending on the database's execution strategy.

Always verify with `EXPLAIN`.

---

# 45. Index and DISTINCT

Example:

```sql
SELECT DISTINCT user_id
FROM orders;
```

An index on:

```text
user_id
```

can sometimes help because the database can work with ordered index values.

Again:

```text
EXPLAIN
```

is the source of truth for the actual plan.

---

# 46. Index Condition vs Query Condition

An index doesn't automatically make every query fast.

Example:

```sql
WHERE user_id = 10
AND description LIKE '%phone%'
```

An index on:

```text
user_id
```

can help narrow the rows first, but the text search may still require additional work.

---

# 47. Database Optimizer

The database optimizer chooses an execution plan based on factors such as:

```text
Indexes
Statistics
Data distribution
Join conditions
Filters
Estimated costs
```

Therefore:

```text
"I created an index"
```

does not guarantee:

```text
"The database will use it."
```

---

# 48. Statistics

Databases maintain statistics about data distribution.

The optimizer uses them to estimate:

```text
How many rows match
Which index is useful
Join costs
Sorting costs
```

Outdated statistics can sometimes lead to poor plans.

The exact statistics-management behavior depends on the database.

---

# 49. Index Not Used

Possible reasons include:

```text
Small table
Low selectivity
Function on column
Type conversion
Leading wildcard
Poor index column order
Outdated statistics
Query returns large percentage of table
Optimizer estimates table scan is cheaper
```

This is a common interview question.

---

# 50. Indexing Small Tables

A tiny table may not benefit much from an index.

Example:

```text
countries
50 rows
```

A table scan of 50 rows may be cheaper than using an index plus fetching table rows.

The optimizer can choose a scan.

---

# 51. High Selectivity Example

Suppose:

```text
users = 10,000,000 rows
```

and:

```text
email = unique
```

A query:

```sql
WHERE email = ?
```

has high selectivity.

An index on email is usually very valuable.

---

# 52. Low Selectivity Example

Suppose:

```text
users = 10,000,000
```

and:

```text
gender = 'M'
```

matches 5,000,000 rows.

An index on gender alone may not provide much benefit for queries returning half the table.

The optimizer may prefer a scan.

---

# 53. Indexing Status

A common question:

```sql
WHERE status = 'PAID'
```

Should status be indexed?

Answer:

```text
It depends.
```

Consider:

```text
Number of rows
Distribution of statuses
Query frequency
Other filters
Composite indexes
Write volume
```

A status column can be useful in a composite index even if it isn't highly selective alone.

---

# 54. Indexing Soft Deletes

Suppose:

```sql
SELECT *
FROM users
WHERE deleted_at IS NULL
AND email = ?;
```

A query-specific composite or specialized index may be useful depending on the database and workload.

Don't blindly index every soft-delete column.

---

# 55. Indexing Search Queries

For:

```sql
WHERE name LIKE '%phone%'
```

a normal B-tree index may not solve the problem.

Depending on requirements, consider:

```text
Full-text indexes
Database-specific text search
Elasticsearch/OpenSearch
```

Choose based on:

```text
Search complexity
Scale
Ranking requirements
Latency
Infrastructure
```

---

# 56. N+1 Query Problem

Indexes can make individual queries faster but do not solve excessive query counts.

Example:

```text
1 query → fetch 100 users

then

100 queries → fetch each user's orders
```

Total:

```text
101 queries
```

This is the classic:

```text
N+1 query problem
```

In Spring/JPA, it can occur with lazy relationships.

---

# 57. Solving N+1

Possible solutions include:

```text
JOIN FETCH
Entity graphs
Batch fetching
Explicit queries
DTO projections
```

Indexes help the individual queries, but reducing unnecessary database round trips is a separate optimization.

---

# 58. Connection Pool vs Query Performance

In Spring Boot, HikariCP manages database connections.

A slow API can result from:

```text
Slow SQL
Connection pool exhaustion
Too many concurrent requests
Database locks
Network latency
```

Increasing the connection pool size does not automatically fix slow SQL.

Often the first step is:

```text
Find the slow query
↓
EXPLAIN
↓
Optimize query/index
```

---

# 59. Query Performance Workflow

A practical backend workflow:

```text
1. Identify slow endpoint
        ↓
2. Identify SQL query
        ↓
3. Measure query latency
        ↓
4. Run EXPLAIN
        ↓
5. Check indexes
        ↓
6. Check row estimates
        ↓
7. Check joins and filters
        ↓
8. Optimize query/index
        ↓
9. Test with realistic data
        ↓
10. Measure again
```

---

# 60. Do Not Optimize Blindly

Bad approach:

```text
Query is slow
↓
Create five indexes
↓
Hope it gets faster
```

Better:

```text
Measure
↓
EXPLAIN
↓
Understand bottleneck
↓
Make one targeted change
↓
Measure again
```

---

# 61. Index Design Questions

Before creating an index, ask:

```text
What query needs it?

What columns are filtered?

Which columns use equality?

Which use ranges?

Is there an ORDER BY?

Is there a JOIN?

How selective are the filters?

How frequently is the query executed?

How frequently is the table written?
```

---

# 62. Index Trade-Off

Think:

```text
Read-heavy system
→ more indexing may be acceptable

Write-heavy system
→ excessive indexes can become expensive
```

But real systems are usually mixed workloads.

Always optimize for actual access patterns.

---

# 63. B-Tree Index

B-tree-style indexes are common in relational databases.

They are useful for operations such as:

```text
=
>
<
>=
<=
BETWEEN
ORDER BY
prefix searches
```

depending on database implementation and query structure.

---

# 64. Hash Index

Hash indexes can be optimized for equality lookups in systems that support them.

Conceptually:

```text
key
 ↓
hash
 ↓
location
```

They are generally not a universal replacement for B-tree indexes because ordered operations require different capabilities.

---

# 65. Clustered Index

Some databases organize table data around a clustered index concept.

For example, InnoDB stores table rows clustered around the primary key.

Important interview point:

```text
Clustered index behavior is database-specific.
```

Do not assume SQL Server, PostgreSQL and MySQL implement clustering identically.

---

# 66. MySQL InnoDB Primary Key

In InnoDB:

```text
Primary key
```

is the clustered index.

Secondary indexes contain the primary-key value used to locate the row.

This is why primary-key size can influence secondary-index storage.

---

# 67. Secondary Index

A secondary index is an index other than the clustered/primary organization.

Example:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

The exact physical structure depends on the database engine.

---

# 68. Index Prefix

For very large string columns, some databases support prefix indexes.

Example in MySQL:

```sql
CREATE INDEX idx_users_name_prefix
ON users(name(20));
```

This indexes the first 20 characters.

It can reduce index size but may reduce selectivity.

Use only when it fits the query and data characteristics.

---

# 69. Partial / Filtered Index

Some databases support indexes that only include rows matching a condition.

For example, PostgreSQL supports partial indexes.

Conceptually:

```sql
CREATE INDEX idx_active_users
ON users(email)
WHERE deleted_at IS NULL;
```

This can be powerful for workloads where only a subset of rows is frequently queried.

Database-specific feature.

---

# 70. Functional / Expression Index

Some databases support indexes on expressions.

Conceptually:

```sql
CREATE INDEX idx_lower_email
ON users (LOWER(email));
```

Then:

```sql
WHERE LOWER(email) = ?
```

can potentially use that index.

Support and syntax vary by database.

---

# 71. Indexing Timestamps

Common backend query:

```sql
SELECT *
FROM orders
WHERE created_at >= ?
AND created_at < ?;
```

An index on:

```text
created_at
```

can be useful.

For multi-tenant applications, a composite index may be better:

```sql
CREATE INDEX idx_orders_tenant_created
ON orders(tenant_id, created_at);
```

This can efficiently target one tenant's time range.

---

# 72. Multi-Tenant Index Design

Query:

```sql
SELECT *
FROM orders
WHERE tenant_id = ?
AND created_at >= ?
AND created_at < ?;
```

Potential index:

```sql
CREATE INDEX idx_orders_tenant_created
ON orders(tenant_id, created_at);
```

This is a common real-world backend pattern.

---

# 73. Index for API Endpoint

Suppose API:

```http
GET /users/{userId}/orders?status=PAID
```

Query:

```sql
SELECT *
FROM orders
WHERE user_id = ?
AND status = ?;
```

Potential index:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

This is how backend engineers should think about indexes:

```text
API access pattern
↓
SQL query
↓
index design
```

---

# 74. Index for Search + Pagination

Query:

```sql
SELECT id, name
FROM products
WHERE category_id = ?
ORDER BY id DESC
LIMIT 20;
```

Potential index:

```sql
CREATE INDEX idx_products_category_id
ON products(category_id, id);
```

This can support filtering and ordering efficiently depending on the database.

---

# 75. Covering Query Example

Suppose:

```sql
SELECT id, status
FROM orders
WHERE user_id = ?
AND status = 'PAID';
```

Index:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

The index contains the columns required by the query.

The database may be able to avoid extra table access.

---

# 76. Index vs Database Cache

These are different.

Index:

```text
Data structure for locating data
```

Cache:

```text
Frequently accessed data kept closer to computation
```

In a Spring Boot application, Redis may cache application-level data:

```text
API
 ↓
Redis
 ↓
Database
```

An SQL index improves database query access.

---

# 77. Redis Does Not Replace SQL Indexes

Bad assumption:

```text
We use Redis
→ indexes aren't necessary
```

Not true.

Redis can reduce database traffic, but cache misses still hit the database.

The database still needs efficient queries.

---

# 78. Indexes and Transactions

Indexes can affect transaction behavior and lock costs.

For example:

```text
UPDATE
WHERE indexed condition
```

can allow the database to identify affected rows more efficiently.

But indexes do not eliminate:

```text
locking
contention
deadlocks
```

These are separate concerns.

---

# 79. Interview: What Is an Index?

> An index is a database data structure that helps the database locate rows efficiently for supported query patterns. It improves reads but consumes storage and adds overhead to inserts, updates and deletes.

---

# 80. Interview: Why Do We Need Indexes?

> We use indexes to improve query performance, especially for frequently filtered, joined or ordered columns. I choose indexes based on actual query patterns rather than indexing every column.

---

# 81. Interview: What Are the Disadvantages of Indexes?

> Indexes consume storage and need to be maintained during inserts, updates and deletes. Too many indexes can therefore hurt write performance and increase maintenance cost.

---

# 82. Interview: What Is a Composite Index?

> A composite index contains multiple columns. The column order matters because databases can generally use the leading portion of the index most effectively.

---

# 83. Interview: What Is the Leftmost Prefix Rule?

> For a composite index such as `(user_id, status, created_at)`, queries using `user_id` or `user_id` with subsequent indexed columns can generally benefit from the index. A query filtering only on `status` usually cannot use it in the same way.

---

# 84. Interview: How Do You Find a Slow Query?

> I first identify the slow endpoint and the SQL being executed, measure its latency, then use `EXPLAIN` or `EXPLAIN ANALYZE` where appropriate. I check scans, indexes, row estimates, joins, sorting and filtering before making a targeted optimization.

---

# 85. Interview: What Is EXPLAIN?

> `EXPLAIN` shows the database's planned execution strategy for a query. It helps me understand things like index usage, join strategy and estimated rows examined.

---

# 86. Interview: Why Might an Index Not Be Used?

> The optimizer may decide a table scan is cheaper, especially for a small table or low-selectivity condition. Other reasons include functions on indexed columns, leading wildcards, poor composite-index order, type conversions or outdated statistics.

---

# 87. Interview: Should Every Column Be Indexed?

> No. Indexes should be based on actual query patterns. Indexing every column increases storage and write overhead without necessarily improving performance.

---

# 88. Interview: What Is a Covering Index?

> A covering index contains all the columns needed by a query, allowing the database to potentially answer the query from the index without fetching the base table row. The actual benefit depends on the database and execution plan.

---

# 89. Interview: What Is Keyset Pagination?

> Keyset pagination uses the last seen sort key instead of a large OFFSET. For example, instead of `OFFSET 100000`, I can query for rows with an ID greater than the last returned ID. It generally scales better for large datasets.

---

# 90. Interview: Does Redis Make Database Indexes Unnecessary?

> No. Redis can reduce database traffic by caching frequently accessed data, but cache misses still reach the database. The underlying SQL queries should still be efficient.

---

# 91. Practical Optimization Example

Suppose:

```sql
SELECT *
FROM orders
WHERE user_id = ?
AND status = ?
ORDER BY created_at DESC
LIMIT 20;
```

First:

```text
Measure latency
```

Then:

```sql
EXPLAIN
SELECT ...
```

Then consider:

```sql
CREATE INDEX idx_orders_user_status_created
ON orders(user_id, status, created_at);
```

Then test again.

The key is:

```text
Measure
→ Explain
→ Change
→ Measure again
```

---

# 92. Production Debugging Example

Imagine an API:

```text
GET /users/100/orders
```

suddenly becomes slow.

Investigation:

```text
API latency
↓
SQL logs
↓
Slow query
↓
EXPLAIN
↓
Full table scan
↓
Missing index on user_id
↓
Add targeted index
↓
Measure again
```

This is a realistic backend troubleshooting workflow.

---

# 93. Indexing Checklist

```text
□ Understand query pattern
□ WHERE columns
□ JOIN columns
□ ORDER BY columns
□ GROUP BY columns
□ Composite indexes
□ Column order
□ Equality vs range
□ Selectivity
□ Cardinality
□ EXPLAIN
□ EXPLAIN ANALYZE
□ Full table scans
□ Index scans/lookups
□ Covering indexes
□ Too many indexes
□ Write overhead
□ Pagination
□ Keyset pagination
□ N+1 queries
□ Slow-query investigation
□ Database statistics
□ API → SQL → index thinking
```

---

# 94. Final Mental Model

Don't think:

```text
"I should index this column because indexes are fast."
```

Think:

```text
API request
   ↓
SQL query
   ↓
WHERE / JOIN / ORDER BY
   ↓
Expected access pattern
   ↓
Candidate index
   ↓
EXPLAIN
   ↓
Measure
   ↓
Optimize
```

The best index is not the one with the most columns.

It is the one that efficiently supports a real workload without creating unnecessary write and storage costs.

> **For backend interviews, remember this sentence: "I don't add indexes blindly. I look at the query pattern, check the execution plan, consider selectivity and write overhead, then validate the improvement with measurements."**
