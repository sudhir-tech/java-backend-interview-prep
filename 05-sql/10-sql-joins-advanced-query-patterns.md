# SQL — Joins & Advanced Query Patterns

Joins are one of the highest-priority SQL topics for Java backend interviews.

You should be comfortable with:

```text
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL OUTER JOIN
CROSS JOIN
SELF JOIN
JOIN conditions
WHERE vs ON
Multiple joins
Aggregations with joins
Subqueries
EXISTS
NOT EXISTS
CTEs
Window functions
CASE
NULL handling
```

---

# 1. What Is a JOIN?

A JOIN combines rows from two or more tables using a related condition.

Example:

```text
users
orders
```

Relationship:

```text
users.id
   ↓
orders.user_id
```

Query:

```sql
SELECT u.name, o.id
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

---

# 2. INNER JOIN

`INNER JOIN` returns rows where the join condition matches on both sides.

```sql
SELECT u.name, o.id
FROM users u
INNER JOIN orders o
    ON u.id = o.user_id;
```

If a user has no orders:

```text
That user is not returned.
```

---

# 3. INNER JOIN Example

Users:

```text
id   name
1    Sudhir
2    Rahul
3    Priya
```

Orders:

```text
id   user_id
101  1
102  1
103  2
```

Result:

```text
Sudhir  101
Sudhir  102
Rahul   103
```

Priya is excluded because she has no matching order.

---

# 4. LEFT JOIN

A `LEFT JOIN` returns:

```text
All rows from the left table
+
Matching rows from the right table
```

Example:

```sql
SELECT u.name, o.id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id;
```

Now Priya appears:

```text
Sudhir  101
Sudhir  102
Rahul   103
Priya   NULL
```

---

# 5. Why LEFT JOIN Is Important

A common backend/reporting requirement:

```text
Find all users,
including users who have never placed an order.
```

Use:

```sql
SELECT u.id, u.name
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
WHERE o.id IS NULL;
```

This returns users without orders.

---

# 6. RIGHT JOIN

A `RIGHT JOIN` returns:

```text
All rows from the right table
+
Matching rows from the left table
```

Example:

```sql
SELECT u.name, o.id
FROM users u
RIGHT JOIN orders o
    ON u.id = o.user_id;
```

Many developers prefer rewriting a `RIGHT JOIN` as a `LEFT JOIN` by swapping table order because it is often easier to read.

---

# 7. FULL OUTER JOIN

A `FULL OUTER JOIN` returns:

```text
Matching rows
+
unmatched rows from left
+
unmatched rows from right
```

Conceptually:

```text
LEFT JOIN
+
right-only rows
```

Not every database supports `FULL OUTER JOIN` directly.

For databases without native support, it can sometimes be emulated with combinations such as:

```text
LEFT JOIN
UNION
RIGHT JOIN
```

with careful duplicate handling.

---

# 8. CROSS JOIN

A `CROSS JOIN` creates a Cartesian product.

Example:

```sql
SELECT *
FROM colors
CROSS JOIN sizes;
```

If:

```text
colors = 3 rows
sizes  = 4 rows
```

result can contain:

```text
3 × 4 = 12 rows
```

Use it intentionally.

An accidental Cartesian product can be a serious performance problem.

---

# 9. SELF JOIN

A self join joins a table to itself.

Example:

```text
employees
```

contains:

```text
id
name
manager_id
```

Query:

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.id;
```

The same table appears twice with different aliases.

---

# 10. JOIN Aliases

Aliases improve readability.

Instead of:

```sql
SELECT users.name, orders.id
FROM users
JOIN orders
ON users.id = orders.user_id;
```

use:

```sql
SELECT u.name, o.id
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

For complex queries, aliases are essential.

---

# 11. Multiple JOINs

Real applications commonly need multiple joins.

Example:

```sql
SELECT
    u.name,
    o.id AS order_id,
    p.name AS product_name,
    oi.quantity
FROM users u
JOIN orders o
    ON o.user_id = u.id
JOIN order_items oi
    ON oi.order_id = o.id
JOIN products p
    ON p.id = oi.product_id;
```

Relationship:

```text
users
  ↓
orders
  ↓
order_items
  ↓
products
```

---

# 12. JOIN Order

SQL syntax may show:

```sql
FROM users u
JOIN orders o
    ON ...
JOIN order_items oi
    ON ...
```

but the database optimizer can choose a different physical execution order.

Therefore:

```text
SQL written order
≠
necessarily execution order
```

Use `EXPLAIN` to inspect the actual plan.

---

# 13. JOIN Condition

The `ON` clause defines how rows are related.

Example:

```sql
ON u.id = o.user_id
```

This means:

```text
user.id
=
order.user_id
```

A missing or incorrect join condition can produce incorrect results or a Cartesian product.

---

# 14. ON vs WHERE

This is a very important interview topic.

Consider:

```sql
SELECT u.name, o.id
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
WHERE o.status = 'PAID';
```

The `WHERE` condition removes rows where:

```text
o.status IS NULL
```

So the query behaves more like an inner join for that condition.

---

# 15. Filtering in ON

Compare:

```sql
SELECT u.name, o.id
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
   AND o.status = 'PAID';
```

Now users without paid orders can still appear:

```text
Sudhir  101
Rahul   NULL
Priya   NULL
```

This distinction is critical.

---

# 16. ON vs WHERE Mental Model

For an outer join:

```text
ON
→ decides which right-side rows match

WHERE
→ filters the final result
```

Moving a condition from `ON` to `WHERE` can change the meaning of the query.

---

# 17. JOIN + GROUP BY

Example:

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

This returns:

```text
User
+
number of orders
```

The `LEFT JOIN` ensures users with zero orders can still appear.

---

# 18. COUNT(*) vs COUNT(column)

With:

```sql
LEFT JOIN
```

this matters.

Example:

```sql
COUNT(*)
```

counts the resulting row.

But:

```sql
COUNT(o.id)
```

counts only non-NULL `o.id` values.

So for users with no orders:

```text
COUNT(*)   → 1
COUNT(o.id) → 0
```

This is a common interview question.

---

# 19. SUM with LEFT JOIN

Example:

```sql
SELECT
    u.id,
    u.name,
    COALESCE(SUM(o.amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
GROUP BY u.id, u.name;
```

`COALESCE` converts NULL into:

```text
0
```

for users with no matching orders.

---

# 20. HAVING with JOIN

Example:

```sql
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
GROUP BY u.id, u.name
HAVING COUNT(o.id) >= 5;
```

`HAVING` filters groups after aggregation.

---

# 21. WHERE vs HAVING

`WHERE`:

```text
Filters rows before grouping.
```

`HAVING`:

```text
Filters groups after aggregation.
```

Example:

```sql
WHERE status = 'PAID'
```

filters individual rows.

```sql
HAVING COUNT(*) > 5
```

filters groups.

---

# 22. JOIN + DISTINCT

Sometimes joins create duplicate-looking rows.

Example:

```sql
SELECT DISTINCT u.id, u.name
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

This returns each user once.

But don't use `DISTINCT` blindly to hide a bad join.

First understand why duplicates are being produced.

---

# 23. Duplicate Rows from One-to-Many

Suppose:

```text
User 1
 ↓
Order 101
Order 102
Order 103
```

Joining users to orders naturally produces:

```text
User 1  Order 101
User 1  Order 102
User 1  Order 103
```

That's not necessarily a bug.

The result is representing the one-to-many relationship.

---

# 24. EXISTS

`EXISTS` checks whether a subquery returns at least one row.

Example:

```sql
SELECT u.*
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

This means:

```text
Find users who have at least one order.
```

---

# 25. NOT EXISTS

Example:

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

This means:

```text
Find users with no orders.
```

This is often a clean alternative to an anti-join.

---

# 26. EXISTS vs IN

Example:

```sql
SELECT *
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
);
```

Alternative:

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

Both can express similar logic.

The optimizer may transform them internally.

Choose the query that clearly expresses the requirement and verify performance with `EXPLAIN`.

---

# 27. NOT IN and NULL

Be careful with:

```sql
NOT IN
```

if the subquery can return NULL.

Example:

```sql
WHERE id NOT IN (
    SELECT user_id
    FROM orders
);
```

If the subquery contains NULL, SQL's three-valued logic can produce surprising results.

`NOT EXISTS` is often safer for this pattern.

---

# 28. SQL NULL

SQL has:

```text
NULL
```

which means:

```text
unknown / missing value
```

It is not the same as:

```text
0
```

or:

```text
''
```

or:

```text
false
```

---

# 29. Comparing with NULL

Incorrect:

```sql
WHERE manager_id = NULL
```

Correct:

```sql
WHERE manager_id IS NULL
```

For non-null:

```sql
WHERE manager_id IS NOT NULL
```

---

# 30. COALESCE

`COALESCE` returns the first non-NULL value.

Example:

```sql
SELECT COALESCE(phone, 'Not provided')
FROM users;
```

If:

```text
phone = NULL
```

result:

```text
Not provided
```

---

# 31. CASE

`CASE` provides conditional logic.

Example:

```sql
SELECT
    id,
    amount,
    CASE
        WHEN amount >= 10000 THEN 'HIGH'
        WHEN amount >= 5000 THEN 'MEDIUM'
        ELSE 'LOW'
    END AS order_category
FROM orders;
```

---

# 32. CASE in Aggregation

Example:

```sql
SELECT
    user_id,
    SUM(
        CASE
            WHEN status = 'PAID' THEN amount
            ELSE 0
        END
    ) AS paid_amount
FROM orders
GROUP BY user_id;
```

This is useful for conditional aggregation.

---

# 33. Conditional COUNT

Example:

```sql
SELECT
    user_id,
    SUM(CASE WHEN status = 'PAID' THEN 1 ELSE 0 END) AS paid_orders
FROM orders
GROUP BY user_id;
```

Depending on the database, another form may be available.

---

# 34. Subquery

A subquery is a query inside another query.

Example:

```sql
SELECT *
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

This finds products priced above the average.

---

# 35. Correlated Subquery

A correlated subquery references the outer query.

Example:

```sql
SELECT p.*
FROM products p
WHERE price > (
    SELECT AVG(p2.price)
    FROM products p2
    WHERE p2.category_id = p.category_id
);
```

The inner query refers to:

```text
p.category_id
```

from the outer query.

---

# 36. Subquery vs JOIN

Many queries can be expressed using either:

```text
JOIN
```

or:

```text
subquery
```

Example requirement:

```text
Find users with orders.
```

JOIN:

```sql
SELECT DISTINCT u.*
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

EXISTS:

```sql
SELECT u.*
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

Neither is universally faster.

Use the clearer expression and verify with the actual execution plan.

---

# 37. Common Table Expression (CTE)

A CTE uses:

```sql
WITH
```

Example:

```sql
WITH paid_orders AS (
    SELECT *
    FROM orders
    WHERE status = 'PAID'
)
SELECT *
FROM paid_orders;
```

CTEs can improve readability and help structure complex queries.

---

# 38. CTE with Aggregation

Example:

```sql
WITH user_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY user_id
)
SELECT
    u.name,
    ut.total_spent
FROM users u
JOIN user_totals ut
    ON ut.user_id = u.id;
```

This separates:

```text
Calculate totals
↓
Join totals to users
```

---

# 39. Multiple CTEs

You can define multiple CTEs.

Example:

```sql
WITH
paid_orders AS (
    SELECT *
    FROM orders
    WHERE status = 'PAID'
),
user_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total
    FROM paid_orders
    GROUP BY user_id
)
SELECT *
FROM user_totals;
```

This can make complicated SQL much easier to read.

---

# 40. Recursive CTE

Some databases support recursive CTEs.

Useful for hierarchical data:

```text
Employee
  ↓
Manager
  ↓
Director
```

Conceptually:

```sql
WITH RECURSIVE employee_tree AS (
    ...
)
SELECT *
FROM employee_tree;
```

This is an advanced topic.

---

# 41. Window Functions

Window functions calculate values across related rows without collapsing them into one row per group.

Example:

```sql
SELECT
    id,
    user_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
    ) AS user_total
FROM orders;
```

Each order remains a separate row.

---

# 42. GROUP BY vs Window Function

`GROUP BY`:

```text
Combines rows into groups
```

Example:

```sql
SELECT user_id, SUM(amount)
FROM orders
GROUP BY user_id;
```

Result:

```text
one row per user
```

Window function:

```text
Keeps individual rows
+
adds calculated information
```

Example:

```sql
SELECT
    id,
    user_id,
    amount,
    SUM(amount) OVER (PARTITION BY user_id)
FROM orders;
```

---

# 43. ROW_NUMBER

Example:

```sql
SELECT
    id,
    user_id,
    amount,
    ROW_NUMBER() OVER (
        PARTITION BY user_id
        ORDER BY amount DESC
    ) AS rn
FROM orders;
```

This numbers each user's orders from highest amount downward.

---

# 44. Ranking

Common window functions:

```text
ROW_NUMBER()
RANK()
DENSE_RANK()
```

They differ when ties occur.

Example amounts:

```text
100
100
90
```

`RANK()`:

```text
1
1
3
```

`DENSE_RANK()`:

```text
1
1
2
```

`ROW_NUMBER()`:

```text
1
2
3
```

---

# 45. Top-N Per Group

A common interview question:

```text
Find the highest-paid employee in each department.
```

Using `ROW_NUMBER()`:

```sql
WITH ranked_employees AS (
    SELECT
        e.*,
        ROW_NUMBER() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS rn
    FROM employees e
)
SELECT *
FROM ranked_employees
WHERE rn = 1;
```

---

# 46. Second Highest Salary

One approach:

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

This finds the second distinct-highest salary.

Window functions can also solve it.

---

# 47. Second Highest Salary with DENSE_RANK

```sql
WITH ranked AS (
    SELECT
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
)
SELECT salary
FROM ranked
WHERE rnk = 2;
```

`DENSE_RANK` handles duplicate salaries correctly.

---

# 48. Latest Row Per User

Common backend question:

```text
Find each user's latest order.
```

Example:

```sql
WITH ranked_orders AS (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY created_at DESC, id DESC
        ) AS rn
    FROM orders o
)
SELECT *
FROM ranked_orders
WHERE rn = 1;
```

The `id` tie-breaker makes ordering deterministic when timestamps are equal.

---

# 49. Running Total

Example:

```sql
SELECT
    id,
    created_at,
    amount,
    SUM(amount) OVER (
        ORDER BY created_at, id
    ) AS running_total
FROM orders;
```

Result:

```text
Order  Amount  Running Total
101    100     100
102    200     300
103    150     450
```

---

# 50. LAG

`LAG()` accesses a previous row.

Example:

```sql
SELECT
    id,
    created_at,
    amount,
    LAG(amount) OVER (
        ORDER BY created_at, id
    ) AS previous_amount
FROM orders;
```

Useful for:

```text
Comparing current vs previous value
Change detection
Time-series analysis
```

---

# 51. LEAD

`LEAD()` accesses a following row.

```sql
SELECT
    id,
    created_at,
    amount,
    LEAD(amount) OVER (
        ORDER BY created_at, id
    ) AS next_amount
FROM orders;
```

Useful for:

```text
Next event
Next transaction
Time intervals
```

---

# 52. JOIN + Window Function

Example:

```sql
WITH ranked_orders AS (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY created_at DESC
        ) AS rn
    FROM orders o
)
SELECT
    u.name,
    r.id AS latest_order_id
FROM users u
JOIN ranked_orders r
    ON r.user_id = u.id
WHERE r.rn = 1;
```

This combines:

```text
CTE
+
window function
+
JOIN
```

---

# 53. UNION

`UNION` combines results from multiple queries and removes duplicates.

Example:

```sql
SELECT email FROM customers
UNION
SELECT email FROM suppliers;
```

---

# 54. UNION ALL

`UNION ALL` combines results without removing duplicates.

```sql
SELECT email FROM customers
UNION ALL
SELECT email FROM suppliers;
```

It is often faster than `UNION` because it doesn't need duplicate elimination.

---

# 55. UNION vs JOIN

JOIN:

```text
Combines columns from related rows.
```

UNION:

```text
Combines rows from compatible result sets.
```

Example:

```text
JOIN
users + orders
→ more columns

UNION
customers + suppliers
→ more rows
```

---

# 56. UNION Requirements

For `UNION` / `UNION ALL`, result queries generally need:

```text
Same number of columns
Compatible data types
```

Example:

```sql
SELECT id, name FROM users
UNION ALL
SELECT id, name FROM customers;
```

---

# 57. Query Execution Order

A simplified logical SQL processing order is:

```text
FROM
JOIN / ON
WHERE
GROUP BY
HAVING
SELECT
DISTINCT
ORDER BY
LIMIT / OFFSET
```

This helps explain many SQL questions.

---

# 58. Why Can't WHERE Use SELECT Alias?

Example:

```sql
SELECT
    price * quantity AS total
FROM order_items
WHERE total > 1000;
```

This usually fails because `WHERE` is logically evaluated before `SELECT`.

Use:

```sql
SELECT
    price * quantity AS total
FROM order_items
WHERE price * quantity > 1000;
```

or a subquery/CTE:

```sql
SELECT *
FROM (
    SELECT price * quantity AS total
    FROM order_items
) x
WHERE total > 1000;
```

Some database-specific extensions may differ, but the logical order is the key concept.

---

# 59. ORDER BY Alias

Unlike `WHERE`, `ORDER BY` can generally use a select-list alias.

Example:

```sql
SELECT
    price * quantity AS total
FROM order_items
ORDER BY total DESC;
```

This is one reason logical SQL processing order matters.

---

# 60. NULL and Aggregates

Common behavior:

```sql
SUM(column)
AVG(column)
```

generally ignore NULL values.

Example:

```text
100
NULL
200
```

`SUM` is:

```text
300
```

not:

```text
NULL
```

But:

```sql
SUM(...)
```

over no matching rows may return NULL, depending on the aggregate/context, so `COALESCE` is often useful.

---

# 61. COALESCE with Aggregates

Example:

```sql
SELECT
    COALESCE(SUM(amount), 0)
FROM orders
WHERE user_id = 999;
```

This gives:

```text
0
```

if no matching amount contributes to the aggregate.

---

# 62. JOIN and NULL

With:

```sql
LEFT JOIN
```

unmatched right-side columns become:

```text
NULL
```

Example:

```text
User has no order

u.name = Priya
o.id   = NULL
```

This is why:

```sql
WHERE o.id IS NULL
```

can find unmatched left-side rows.

---

# 63. Anti-Join

An anti-join means:

```text
Rows in A that have no matching row in B.
```

Example:

```sql
SELECT u.*
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
WHERE o.id IS NULL;
```

Equivalent concept:

```sql
WHERE NOT EXISTS (...)
```

---

# 64. Semi-Join

A semi-join means:

```text
Rows in A that have at least one match in B.
```

Example:

```sql
SELECT u.*
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

It returns each user once even if the user has many orders.

---

# 65. Avoid Accidental Cartesian Products

Bad:

```sql
SELECT *
FROM users u, orders o;
```

without a relationship condition.

This can create:

```text
users × orders
```

rows.

Use explicit joins:

```sql
FROM users u
JOIN orders o
    ON o.user_id = u.id
```

Explicit JOIN syntax is clearer and safer.

---

# 66. JOIN and Indexes

For:

```sql
SELECT *
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

useful indexes often include:

```text
users.id
orders.user_id
```

The primary key usually provides an index for `users.id`.

The actual execution plan should be checked with:

```sql
EXPLAIN
```

---

# 67. JOIN and Large Tables

For large tables, consider:

```text
Join columns
Filtering columns
Composite indexes
Statistics
Data distribution
Query selectivity
```

A JOIN itself isn't necessarily slow.

A poorly planned join can be slow.

---

# 68. Nested Loop Join

A database may execute a join using a nested-loop strategy.

Conceptually:

```text
For each row from A
    find matching rows in B
```

With a good index on B, this can be very efficient for selective joins.

---

# 69. Hash Join

A hash join can build a hash structure for one input and use it to find matches from the other input.

Conceptually:

```text
Build hash table
      ↓
Probe with other rows
      ↓
Matches
```

It is often useful for larger equality joins.

Support and optimizer behavior vary by database.

---

# 70. Sort-Merge Join

A sort-merge join conceptually:

```text
Sort input A
Sort input B
      ↓
Merge matching values
```

It can be effective when inputs are already sorted or when the database chooses this strategy for the workload.

---

# 71. Query Plan

The database optimizer chooses among strategies such as:

```text
Nested loop
Hash join
Merge join
Index lookup
Table scan
Sort
```

depending on:

```text
Statistics
Indexes
Data distribution
Query conditions
Estimated costs
```

Use:

```sql
EXPLAIN
```

to understand the selected plan.

---

# 72. SQL Injection

Never construct SQL using raw user input like:

```java
String sql =
    "SELECT * FROM users WHERE email = '" + email + "'";
```

This can create SQL injection vulnerabilities.

Use:

```text
Prepared statements
Parameterized queries
JPA parameters
JdbcTemplate parameters
```

Example:

```java
jdbcTemplate.query(
    "SELECT * FROM users WHERE email = ?",
    rowMapper,
    email
);
```

---

# 73. SQL Injection and JPA

With JPQL:

```java
@Query("""
    SELECT u
    FROM User u
    WHERE u.email = :email
""")
Optional<User> findByEmail(
    @Param("email") String email
);
```

The parameter is bound separately.

Avoid concatenating untrusted input into query strings.

---

# 74. N+1 Query Problem

A join can sometimes help avoid N+1 queries.

Bad pattern:

```text
1 query → fetch users

then for each user:
1 query → fetch orders
```

For 100 users:

```text
101 queries
```

Potential alternatives:

```text
JOIN FETCH
EntityGraph
DTO projection
Explicit JOIN
Batch fetching
```

Choose based on the required data and result size.

---

# 75. JOIN FETCH in JPA

Example:

```java
@Query("""
    SELECT DISTINCT u
    FROM User u
    JOIN FETCH u.orders
""")
List<User> findUsersWithOrders();
```

This can fetch related data in fewer queries.

But fetching multiple collections carelessly can create:

```text
Huge Cartesian-style result sets
Duplicate rows
Memory problems
```

So don't use `JOIN FETCH` blindly.

---

# 76. DTO Projection

Instead of fetching full entities:

```java
SELECT new com.example.UserOrderDTO(
    u.id,
    u.name,
    o.id
)
FROM User u
JOIN u.orders o
```

you can fetch only fields required by the API.

This can reduce:

```text
Data transfer
Entity creation
Memory usage
```

---

# 77. SQL Query Optimization Workflow

For a slow JOIN:

```text
1. Identify actual query
        ↓
2. Check row counts
        ↓
3. Run EXPLAIN
        ↓
4. Check join strategy
        ↓
5. Check indexes
        ↓
6. Check filtering
        ↓
7. Check result size
        ↓
8. Optimize
        ↓
9. Measure again
```

---

# 78. Real E-Commerce Query

Requirement:

```text
Find each user's total paid amount.
```

Query:

```sql
SELECT
    u.id,
    u.name,
    COALESCE(SUM(
        CASE
            WHEN o.status = 'PAID'
            THEN o.amount
            ELSE 0
        END
    ), 0) AS total_paid
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
GROUP BY u.id, u.name;
```

This combines:

```text
LEFT JOIN
CASE
SUM
COALESCE
GROUP BY
```

---

# 79. Real E-Commerce Query — Latest Order

```sql
WITH ranked_orders AS (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY o.user_id
            ORDER BY o.created_at DESC, o.id DESC
        ) AS rn
    FROM orders o
)
SELECT
    u.id,
    u.name,
    r.id AS latest_order_id,
    r.created_at
FROM users u
LEFT JOIN ranked_orders r
    ON r.user_id = u.id
   AND r.rn = 1;
```

This returns users even when they have no orders.

---

# 80. Real E-Commerce Query — Users Without Orders

```sql
SELECT u.id, u.name
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

This is clean and often preferable to using `NOT IN` when NULL behavior could be problematic.

---

# 81. Real E-Commerce Query — Top Product

```sql
SELECT
    p.id,
    p.name,
    SUM(oi.quantity) AS units_sold
FROM products p
JOIN order_items oi
    ON oi.product_id = p.id
JOIN orders o
    ON o.id = oi.order_id
WHERE o.status = 'PAID'
GROUP BY p.id, p.name
ORDER BY units_sold DESC
LIMIT 1;
```

This combines:

```text
JOIN
WHERE
GROUP BY
SUM
ORDER BY
LIMIT
```

---

# 82. Interview: INNER JOIN vs LEFT JOIN

> `INNER JOIN` returns only matching rows from both tables. `LEFT JOIN` returns all rows from the left table and matching rows from the right, using NULL for unmatched right-side values.

---

# 83. Interview: LEFT JOIN with WHERE Condition

> With an outer join, moving a right-table condition from `ON` to `WHERE` can eliminate rows where the right side is NULL. That can effectively change the result toward inner-join behavior, so I place the condition based on the intended semantics.

---

# 84. Interview: What Is a SELF JOIN?

> A self join joins a table to itself, usually with different aliases. A common example is an employee table where `manager_id` references another employee's `id`.

---

# 85. Interview: What Is a CROSS JOIN?

> A cross join creates a Cartesian product, combining every row from one table with every row from the other. I use it only when that combination is intentional because the result can grow very quickly.

---

# 86. Interview: JOIN vs EXISTS

> A JOIN combines columns from matching rows, while `EXISTS` checks whether at least one related row exists. If I only need to know whether a relationship exists, `EXISTS` can express the requirement clearly and avoids producing duplicate parent rows.

---

# 87. Interview: WHERE vs HAVING

> `WHERE` filters rows before grouping, while `HAVING` filters groups after aggregation. For example, I use `WHERE status = 'PAID'` to filter orders and `HAVING COUNT(*) > 5` to filter grouped users.

---

# 88. Interview: GROUP BY vs Window Function

> `GROUP BY` collapses rows into groups, while a window function calculates across related rows without removing the individual rows. I use window functions for things like ranking, running totals and latest-record-per-group queries.

---

# 89. Interview: What Is a CTE?

> A CTE, created with `WITH`, is a named query expression that can make complex SQL easier to structure and read. It is especially useful for multi-step queries and recursive queries where supported.

---

# 90. Interview: UNION vs UNION ALL

> `UNION` combines compatible result sets and removes duplicates. `UNION ALL` keeps duplicates and is generally cheaper because it doesn't perform duplicate elimination.

---

# 91. Interview: What Is a Window Function?

> A window function performs a calculation across related rows while keeping each original row in the result. Examples include `ROW_NUMBER`, `RANK`, `SUM OVER`, `LAG` and `LEAD`.

---

# 92. Interview: How Do You Find the Latest Record Per User?

> I commonly use `ROW_NUMBER()` partitioned by user and ordered by the timestamp descending. Then I filter for `row_number = 1`. I also add a unique tie-breaker such as ID to make the ordering deterministic.

---

# 93. Interview: How Do You Find Users Without Orders?

> I prefer `NOT EXISTS` for this requirement:
>
> ```sql
> SELECT u.*
> FROM users u
> WHERE NOT EXISTS (
>     SELECT 1
>     FROM orders o
>     WHERE o.user_id = u.id
> );
> ```

---

# 94. Interview: What Is the N+1 Problem?

> N+1 happens when the application executes one query to fetch a set of parent records and then one additional query for each parent. I address it using appropriate joins, fetch strategies, DTO projections or batching depending on the use case.

---

# 95. Interview: How Do You Optimize a Slow JOIN?

> I first inspect the actual query and run `EXPLAIN`. Then I check join columns, indexes, filtering, row estimates, data volume and the selected join strategy. I make a targeted change and measure the query again rather than adding indexes blindly.

---

# 96. Interview: What Is SQL Injection?

> SQL injection occurs when untrusted input is incorporated into SQL in a way that changes the query's meaning. I prevent it by using parameterized queries, prepared statements and safe ORM query parameters rather than string concatenation.

---

# 97. Advanced SQL Checklist

```text
□ INNER JOIN
□ LEFT JOIN
□ RIGHT JOIN
□ FULL OUTER JOIN
□ CROSS JOIN
□ SELF JOIN
□ Multiple JOINs
□ ON vs WHERE
□ JOIN + GROUP BY
□ COUNT(*) vs COUNT(column)
□ HAVING
□ DISTINCT
□ EXISTS
□ NOT EXISTS
□ IN / NOT IN
□ NULL
□ COALESCE
□ CASE
□ Subqueries
□ Correlated subqueries
□ CTE
□ Recursive CTE
□ Window functions
□ ROW_NUMBER
□ RANK
□ DENSE_RANK
□ LAG
□ LEAD
□ Running totals
□ Top-N per group
□ Latest row per group
□ UNION
□ UNION ALL
□ SQL execution order
□ Anti-join
□ Semi-join
□ Nested-loop join
□ Hash join
□ Merge join
□ EXPLAIN
□ N+1
□ JOIN FETCH
□ DTO projections
□ SQL injection
```

---

# 98. Final Mental Model

When writing a complex backend query, think:

```text
What data do I need?
        ↓
Which tables contain it?
        ↓
How are the tables related?
        ↓
INNER / LEFT / EXISTS?
        ↓
What rows should be filtered?
        ↓
WHERE
        ↓
Do I need grouping?
        ↓
GROUP BY / HAVING
        ↓
Do I need ranking or calculations?
        ↓
Window functions
        ↓
Do I need sorting/pagination?
        ↓
ORDER BY / LIMIT
        ↓
Is it fast enough?
        ↓
EXPLAIN
```

For Java backend development:

```text
API requirement
      ↓
Repository query
      ↓
JOIN / WHERE / GROUP BY
      ↓
Indexes
      ↓
EXPLAIN
      ↓
DTO/entity mapping
      ↓
API response
```

> **Interview shortcut:** Don't just memorize JOIN definitions. Be able to solve real problems: "users without orders", "latest order per user", "top product", "second-highest salary", "total spent per user", and "N+1 query problem". These demonstrate that you can actually use SQL rather than only define it.
