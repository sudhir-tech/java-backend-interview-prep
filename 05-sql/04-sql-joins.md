# SQL — JOINs

JOINs are one of the most important SQL topics for Java backend interviews. They allow us to combine related data from multiple tables.

For an e-commerce system, we may have:

```text
users
products
orders
order_items
```

JOINs let us answer questions such as:

```text
Which user placed each order?
Which products are in an order?
Which users have never placed an order?
Which products have been ordered?
```

---

# 1. Why JOINs?

Suppose we have:

```text
users
+----+--------+
| id | name   |
+----+--------+
| 1  | Sudhir |
| 2  | Rahul  |
| 3  | Priya  |
+----+--------+
```

and:

```text
orders
+----+---------+--------+
| id | user_id | amount |
+----+---------+--------+
| 101| 1       | 50000  |
| 102| 1       | 20000  |
| 103| 2       | 10000  |
+----+---------+--------+
```

The relationship is:

```text
users.id
   ↓
orders.user_id
```

A JOIN combines these related rows.

---

# 2. INNER JOIN

`INNER JOIN` returns only rows where the join condition matches.

Example:

```sql
SELECT
    u.id,
    u.name,
    o.id AS order_id,
    o.amount
FROM users u
INNER JOIN orders o
    ON u.id = o.user_id;
```

Result:

```text
name    order_id   amount
------  ---------  ------
Sudhir  101        50000
Sudhir  102        20000
Rahul   103        10000
```

Priya does not appear because she has no matching order.

---

# 3. INNER JOIN Mental Model

Think:

```text
Table A
   ∩
Table B
```

Only matching rows are returned.

---

# 4. JOIN vs INNER JOIN

In most SQL dialects:

```sql
FROM users
JOIN orders
    ON users.id = orders.user_id
```

means:

```sql
FROM users
INNER JOIN orders
    ON users.id = orders.user_id
```

`INNER` can be omitted.

---

# 5. LEFT JOIN

A `LEFT JOIN` returns:

```text
All rows from the left table
+
Matching rows from the right table
```

Example:

```sql
SELECT
    u.id,
    u.name,
    o.id AS order_id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id;
```

Result:

```text
name    order_id
------  --------
Sudhir  101
Sudhir  102
Rahul   103
Priya   NULL
```

Priya is included even though she has no order.

---

# 6. LEFT JOIN Mental Model

Think:

```text
All of A
+
Matching part of B
```

This is extremely useful when you need to find records that may not have related data.

---

# 7. RIGHT JOIN

A `RIGHT JOIN` returns:

```text
All rows from the right table
+
Matching rows from the left table
```

Example:

```sql
SELECT
    u.name,
    o.id AS order_id
FROM users u
RIGHT JOIN orders o
    ON u.id = o.user_id;
```

It is less commonly used because the same query can usually be expressed more naturally by swapping table order and using `LEFT JOIN`.

---

# 8. FULL OUTER JOIN

A `FULL OUTER JOIN` returns:

```text
All matching rows
+
Unmatched rows from left
+
Unmatched rows from right
```

Conceptually:

```text
A ∪ B
```

However, MySQL does not provide native `FULL OUTER JOIN` syntax.

In MySQL, you may simulate it using combinations such as:

```text
LEFT JOIN
UNION
RIGHT JOIN
```

with appropriate handling of duplicates.

---

# 9. CROSS JOIN

A `CROSS JOIN` produces the Cartesian product.

Example:

```sql
SELECT
    u.name,
    p.name AS product_name
FROM users u
CROSS JOIN products p;
```

If there are:

```text
3 users
4 products
```

the result can contain:

```text
3 × 4 = 12 rows
```

Use it intentionally. Accidental Cartesian products can create huge result sets.

---

# 10. SELF JOIN

A self join joins a table to itself.

Example employee hierarchy:

```text
employees
+----+----------+------------+
| id | name     | manager_id |
+----+----------+------------+
| 1  | Manager  | NULL       |
| 2  | Sudhir   | 1          |
| 3  | Rahul    | 1          |
+----+----------+------------+
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

Result:

```text
employee  manager
--------  -------
Manager   NULL
Sudhir    Manager
Rahul     Manager
```

---

# 11. JOIN Condition

The `ON` clause defines how rows are related.

Example:

```sql
ON u.id = o.user_id
```

This means:

```text
users.id
=
orders.user_id
```

---

# 12. ON vs WHERE

This is a very important interview topic.

Consider:

```sql
SELECT
    u.name,
    o.id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
WHERE o.status = 'PAID';
```

The `WHERE` condition removes rows where:

```text
o.status IS NULL
```

So the query behaves much more like an inner join for this condition.

---

# 13. LEFT JOIN Condition in ON

Compare:

```sql
SELECT
    u.name,
    o.id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
   AND o.status = 'PAID';
```

Now all users remain.

Users without paid orders receive:

```text
NULL
```

for the order columns.

This distinction is extremely important.

---

# 14. Finding Rows Without a Match

A classic interview pattern:

```text
Find users who have never placed an order.
```

Use:

```sql
SELECT
    u.id,
    u.name
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
WHERE o.id IS NULL;
```

Mental model:

```text
LEFT JOIN
   ↓
Keep all users
   ↓
Keep only rows where order is missing
```

---

# 15. NOT EXISTS Alternative

The same requirement can often be expressed with:

```sql
SELECT
    u.id,
    u.name
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

Both patterns can be useful.

For performance-sensitive queries, compare actual execution plans rather than assuming one is always faster.

---

# 16. Finding Rows With a Match

Example:

```text
Find users who have at least one order.
```

Using `EXISTS`:

```sql
SELECT
    u.id,
    u.name
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

This avoids returning duplicate user rows when a user has many orders.

---

# 17. JOIN + DISTINCT

Another approach:

```sql
SELECT DISTINCT
    u.id,
    u.name
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

This works, but `EXISTS` can communicate the intent more directly when you only need to know whether a match exists.

---

# 18. One-to-Many JOIN

Suppose:

```text
users
    1
    |
    N
orders
```

Query:

```sql
SELECT
    u.name,
    o.id
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

One user can produce multiple result rows.

Example:

```text
Sudhir  101
Sudhir  102
Sudhir  103
```

This is not duplication from SQL's perspective; these are distinct joined rows.

---

# 19. Many-to-Many JOIN

Suppose:

```text
students
      |
      |
student_courses
      |
      |
courses
```

Query:

```sql
SELECT
    s.name,
    c.name AS course
FROM students s
JOIN student_courses sc
    ON s.id = sc.student_id
JOIN courses c
    ON c.id = sc.course_id;
```

This combines three related tables.

---

# 20. Multiple JOINs

Example e-commerce query:

```sql
SELECT
    u.name,
    o.id AS order_id,
    p.name AS product_name,
    oi.quantity
FROM users u
JOIN orders o
    ON u.id = o.user_id
JOIN order_items oi
    ON o.id = oi.order_id
JOIN products p
    ON p.id = oi.product_id;
```

Relationship chain:

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

# 21. JOIN with Aggregation

Find order count per user:

```sql
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
GROUP BY
    u.id,
    u.name;
```

Why `LEFT JOIN`?

Because users with zero orders should still appear.

---

# 22. COUNT(*) vs COUNT(o.id) with LEFT JOIN

This is a common interview trap.

Consider:

```sql
SELECT
    u.id,
    COUNT(*) AS order_count
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
GROUP BY u.id;
```

A user with no order can still have one NULL-extended joined row, so:

```text
COUNT(*)
```

can return:

```text
1
```

instead of zero.

Prefer:

```sql
COUNT(o.id)
```

because the order ID is NULL when no order exists.

---

# 23. Correct Zero-Order Count

```sql
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
GROUP BY
    u.id,
    u.name;
```

Now users with no orders correctly get:

```text
0
```

---

# 24. JOIN + WHERE

Example:

```sql
SELECT
    u.name,
    o.amount
FROM users u
JOIN orders o
    ON u.id = o.user_id
WHERE o.amount > 50000;
```

Because this is an inner join, filtering matching orders in `WHERE` is usually straightforward.

---

# 25. JOIN + GROUP BY + HAVING

Find users whose total paid amount exceeds 100,000:

```sql
SELECT
    u.id,
    u.name,
    SUM(o.amount) AS total_spent
FROM users u
JOIN orders o
    ON u.id = o.user_id
WHERE o.status = 'PAID'
GROUP BY
    u.id,
    u.name
HAVING SUM(o.amount) > 100000;
```

---

# 26. JOIN + ORDER BY

Find users with the highest spending:

```sql
SELECT
    u.id,
    u.name,
    SUM(o.amount) AS total_spent
FROM users u
JOIN orders o
    ON u.id = o.user_id
WHERE o.status = 'PAID'
GROUP BY
    u.id,
    u.name
ORDER BY total_spent DESC;
```

---

# 27. LEFT JOIN + HAVING

Find users with at least 3 orders:

```sql
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
GROUP BY
    u.id,
    u.name
HAVING COUNT(o.id) >= 3;
```

Users with fewer than three orders are filtered out after grouping.

---

# 28. JOINing Three Tables

Example:

```sql
SELECT
    o.id AS order_id,
    u.name AS customer,
    p.name AS product
FROM orders o
JOIN users u
    ON u.id = o.user_id
JOIN order_items oi
    ON oi.order_id = o.id
JOIN products p
    ON p.id = oi.product_id;
```

This is common in real backend applications.

---

# 29. JOINing Four Tables

A larger e-commerce query:

```sql
SELECT
    u.name,
    o.id AS order_id,
    p.name AS product,
    oi.quantity,
    oi.price
FROM users u
JOIN orders o
    ON o.user_id = u.id
JOIN order_items oi
    ON oi.order_id = o.id
JOIN products p
    ON p.id = oi.product_id;
```

Always understand the cardinality of each relationship before writing the query.

---

# 30. Duplicate Rows After JOIN

Suppose:

```text
User
 ↓
5 orders
```

Joining users to orders produces:

```text
User 1 → Order 1
User 1 → Order 2
User 1 → Order 3
User 1 → Order 4
User 1 → Order 5
```

If you only wanted one row per user, the query needs:

```text
GROUP BY
DISTINCT
EXISTS
```

or another appropriate strategy.

Do not blindly add `DISTINCT` without understanding why duplicates exist.

---

# 31. JOIN Cardinality

Before joining, ask:

```text
One-to-one?
One-to-many?
Many-to-many?
```

For example:

```text
users → orders
```

is usually:

```text
1 : N
```

Therefore a join can multiply rows.

---

# 32. Cartesian Explosion

If joins are written incorrectly, result size can grow dramatically.

Example:

```text
100 users
1000 orders
5000 order_items
```

An incorrect join condition can create a huge Cartesian product.

Always verify:

```text
JOIN condition
Foreign-key relationship
Expected cardinality
```

---

# 33. Missing JOIN Condition

Dangerous:

```sql
SELECT *
FROM users u
JOIN orders o;
```

Depending on syntax/database, this may effectively produce a Cartesian product or be rejected.

Always specify the relationship intentionally.

---

# 34. JOIN on Non-Key Columns

You can join on other columns:

```sql
SELECT *
FROM users u
JOIN orders o
    ON u.email = o.customer_email;
```

But joining on non-unique or mutable business fields can create incorrect matches.

Prefer stable keys where the data model allows it.

---

# 35. JOIN on Foreign Keys

Typical:

```sql
ON orders.user_id = users.id
```

This is usually preferable because:

```text
user_id → foreign key
id      → primary key
```

The relationship is explicit and indexed appropriately in a well-designed schema.

---

# 36. Indexes and JOINs

JOIN performance can benefit from indexes on columns used for matching.

For:

```sql
ON orders.user_id = users.id
```

you commonly want:

```text
users.id
```

indexed by the primary key and often:

```text
orders.user_id
```

indexed as well.

The optimizer and database schema determine the exact best strategy.

---

# 37. LEFT JOIN with Multiple Conditions

Example:

```sql
SELECT
    u.name,
    o.id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
   AND o.status = 'PAID'
   AND o.amount > 50000;
```

This means:

```text
Keep every user
Match only paid orders above 50000
```

---

# 38. Same Conditions in WHERE

Compare:

```sql
SELECT
    u.name,
    o.id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
WHERE o.status = 'PAID'
  AND o.amount > 50000;
```

Now users without matching paid orders are removed.

The result is effectively an inner-join-like subset for those conditions.

---

# 39. Interview: ON vs WHERE

A strong answer:

> `ON` defines how rows are matched during the join, while `WHERE` filters the resulting rows. With an inner join, many conditions can produce the same result either way, but with an outer join the placement can change whether unmatched rows are preserved.

---

# 40. RIGHT JOIN vs LEFT JOIN

You can usually rewrite:

```sql
A RIGHT JOIN B
```

as:

```sql
B LEFT JOIN A
```

This is why many teams standardize on `LEFT JOIN` for readability.

---

# 41. FULL OUTER JOIN in MySQL

MySQL does not have:

```sql
FULL OUTER JOIN
```

as a native join type.

One conceptual workaround is:

```sql
SELECT ...
FROM A
LEFT JOIN B
    ON ...

UNION

SELECT ...
FROM A
RIGHT JOIN B
    ON ...;
```

The exact query depends on desired duplicate handling.

---

# 42. SELF JOIN Use Cases

Self joins can solve:

```text
Employee → Manager
Category → Parent category
Friend relationships
Hierarchical data
Comparing rows within one table
```

Example:

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.id;
```

---

# 43. Finding Duplicate Values

A common query:

```sql
SELECT
    email,
    COUNT(*) AS email_count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

This uses:

```text
GROUP BY
+
HAVING
```

rather than a JOIN.

---

# 44. Finding Duplicate Rows with a JOIN

Sometimes you may need to retrieve full duplicate records:

```sql
SELECT u.*
FROM users u
JOIN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
) d
    ON d.email = u.email;
```

This demonstrates combining aggregation and JOINs.

---

# 45. EXISTS vs JOIN

Suppose the requirement is:

```text
Find users who have at least one order.
```

You can use:

```sql
SELECT DISTINCT u.id, u.name
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

or:

```sql
SELECT u.id, u.name
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

`EXISTS` expresses existence directly and avoids generating duplicate user rows.

---

# 46. NOT EXISTS

Find users with no orders:

```sql
SELECT u.id, u.name
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

This is often a clean alternative to:

```text
LEFT JOIN + IS NULL
```

---

# 47. Anti-Join Pattern

A query like:

```sql
SELECT u.*
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
WHERE o.id IS NULL;
```

is commonly called an:

```text
anti-join pattern
```

It finds rows from one table with no matching row in another.

---

# 48. Semi-Join Pattern

A query like:

```sql
SELECT u.*
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

is commonly described as a:

```text
semi-join pattern
```

It finds rows with at least one match without returning the matching rows themselves.

---

# 49. JOIN with Aggregated Subquery

Example:

```sql
SELECT
    u.id,
    u.name,
    x.total_spent
FROM users u
JOIN (
    SELECT
        user_id,
        SUM(amount) AS total_spent
    FROM orders
    WHERE status = 'PAID'
    GROUP BY user_id
) x
    ON x.user_id = u.id;
```

This first calculates:

```text
spending per user
```

and then joins that result to users.

---

# 50. JOIN Order

SQL allows the database optimizer to choose an efficient execution strategy.

Do not assume the written order always equals the physical execution order.

Use:

```sql
EXPLAIN
```

to inspect the plan.

---

# 51. JOIN Performance Checklist

When a JOIN is slow, inspect:

```text
JOIN conditions
Indexes
Table sizes
Cardinality
Filters
Selected columns
Execution plan
Data distribution
```

Do not immediately assume:

```text
JOIN = slow
```

Well-indexed joins are fundamental to relational databases.

---

# 52. Avoid SELECT * in Large JOINs

Instead of:

```sql
SELECT *
FROM users u
JOIN orders o
    ON ...
```

prefer:

```sql
SELECT
    u.id,
    u.name,
    o.id AS order_id,
    o.amount
FROM users u
JOIN orders o
    ON ...;
```

This reduces unnecessary data transfer and makes the query contract clearer.

---

# 53. JOIN with Pagination

Example:

```sql
SELECT
    o.id,
    u.name,
    o.amount
FROM orders o
JOIN users u
    ON u.id = o.user_id
ORDER BY o.id DESC
LIMIT 20;
```

The JOIN does not prevent pagination.

But for large datasets, inspect the execution plan and consider appropriate indexes and keyset pagination.

---

# 54. Pagination with One-to-Many JOIN

Be careful:

```text
users
  ↓
orders
```

If you paginate the joined rows, you are paginating:

```text
user-order combinations
```

not necessarily:

```text
users
```

This can produce surprising application behavior.

If the API needs pages of users, first identify the user IDs for the page and then fetch related data separately or use an appropriate two-step strategy.

---

# 55. JOIN and NULL

With an outer join, unmatched columns from the other side become:

```text
NULL
```

Example:

```text
Priya | NULL
```

This is why conditions involving the right table need careful placement.

---

# 56. INNER JOIN and NULL

Suppose:

```text
orders.user_id = NULL
```

An equality condition:

```sql
u.id = o.user_id
```

does not evaluate to TRUE.

Therefore the row does not match an inner join.

---

# 57. JOIN on Multiple Columns

A relationship can use multiple columns.

Example:

```sql
ON a.customer_id = b.customer_id
AND a.order_date = b.order_date
```

This is useful when the relationship is defined by a composite key.

---

# 58. NATURAL JOIN

Some SQL dialects support:

```sql
NATURAL JOIN
```

It automatically joins columns with matching names.

Avoid relying on it in production code because schema changes can unexpectedly change which columns participate in the join.

Explicit `ON` conditions are clearer and safer.

---

# 59. USING

Some SQL dialects support:

```sql
SELECT *
FROM users
JOIN orders
USING (user_id);
```

This is useful when both tables have a column with exactly the same name.

But explicit `ON` is often clearer for complex relationships.

---

# 60. JOIN with Aliases

Good:

```sql
SELECT
    u.name,
    o.amount
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

Aliases make complex queries much easier to read.

---

# 61. Common Interview Query #1

**Find all users and their orders, including users with no orders.**

```sql
SELECT
    u.id,
    u.name,
    o.id AS order_id
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id;
```

---

# 62. Common Interview Query #2

**Find users who never placed an order.**

```sql
SELECT
    u.id,
    u.name
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
WHERE o.id IS NULL;
```

Alternative:

```sql
SELECT
    u.id,
    u.name
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

---

# 63. Common Interview Query #3

**Find the number of orders for each user.**

```sql
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
GROUP BY
    u.id,
    u.name;
```

---

# 64. Common Interview Query #4

**Find users with more than five orders.**

```sql
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS order_count
FROM users u
JOIN orders o
    ON o.user_id = u.id
GROUP BY
    u.id,
    u.name
HAVING COUNT(o.id) > 5;
```

---

# 65. Common Interview Query #5

**Find each user's total paid amount.**

```sql
SELECT
    u.id,
    u.name,
    SUM(o.amount) AS total_paid
FROM users u
JOIN orders o
    ON o.user_id = u.id
WHERE o.status = 'PAID'
GROUP BY
    u.id,
    u.name;
```

---

# 66. Common Interview Query #6

**Find products that have never been ordered.**

Assuming:

```text
products.id
order_items.product_id
```

Use:

```sql
SELECT
    p.id,
    p.name
FROM products p
LEFT JOIN order_items oi
    ON oi.product_id = p.id
WHERE oi.product_id IS NULL;
```

---

# 67. Common Interview Query #7

**Find products that have been ordered at least once.**

```sql
SELECT DISTINCT
    p.id,
    p.name
FROM products p
JOIN order_items oi
    ON oi.product_id = p.id;
```

Alternative:

```sql
SELECT
    p.id,
    p.name
FROM products p
WHERE EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.id
);
```

---

# 68. Common Interview Query #8

**Find each product's total ordered quantity.**

```sql
SELECT
    p.id,
    p.name,
    SUM(oi.quantity) AS total_quantity
FROM products p
JOIN order_items oi
    ON oi.product_id = p.id
GROUP BY
    p.id,
    p.name;
```

---

# 69. Common Interview Query #9

**Find the highest-spending user.**

```sql
SELECT
    u.id,
    u.name,
    SUM(o.amount) AS total_spent
FROM users u
JOIN orders o
    ON o.user_id = u.id
WHERE o.status = 'PAID'
GROUP BY
    u.id,
    u.name
ORDER BY total_spent DESC
LIMIT 1;
```

For ties, use a ranking/window-function approach.

---

# 70. Common Interview Query #10

**Find users who have both paid and cancelled orders.**

One approach:

```sql
SELECT
    u.id,
    u.name
FROM users u
JOIN orders o
    ON o.user_id = u.id
GROUP BY
    u.id,
    u.name
HAVING SUM(o.status = 'PAID') > 0
   AND SUM(o.status = 'CANCELLED') > 0;
```

This syntax works in MySQL because boolean expressions can be evaluated numerically.

A more portable approach is:

```sql
HAVING
    SUM(CASE WHEN o.status = 'PAID' THEN 1 ELSE 0 END) > 0
AND
    SUM(CASE WHEN o.status = 'CANCELLED' THEN 1 ELSE 0 END) > 0;
```

---

# 71. Interview: What Is INNER JOIN?

> `INNER JOIN` returns only rows where the join condition matches in both tables. If a user has no order, that user won't appear when users are inner-joined to orders.

---

# 72. Interview: What Is LEFT JOIN?

> `LEFT JOIN` returns every row from the left table and matching rows from the right table. If there is no match, the right-side columns become NULL. It's useful for finding missing relationships, such as users who never placed an order.

---

# 73. Interview: LEFT JOIN vs INNER JOIN?

> Inner join returns only matching records. Left join preserves all records from the left table, even when there is no match on the right. I choose based on whether unmatched left-side records are required.

---

# 74. Interview: What Is a SELF JOIN?

> A self join joins a table to itself using different aliases. A common example is an employee table where each employee has a manager ID referencing another employee.

---

# 75. Interview: What Is a CROSS JOIN?

> A cross join produces the Cartesian product of two tables. If one table has 100 rows and another has 50, the result can contain 5,000 combinations. It is useful for intentional combinations but dangerous when caused accidentally.

---

# 76. Interview: Why Does a JOIN Create Duplicate Rows?

> Usually because the relationship is one-to-many or many-to-many. For example, one user can have multiple orders, so joining users to orders naturally produces multiple rows for that user. I first understand the cardinality instead of blindly using DISTINCT.

---

# 77. Interview: JOIN vs EXISTS?

> If I need columns from both tables, a JOIN is appropriate. If I only need to know whether a related record exists, `EXISTS` can express that intent more directly and avoids generating duplicate parent rows.

---

# 78. Interview: How Do You Find Records With No Match?

> A common approach is a `LEFT JOIN` followed by `WHERE right_table.id IS NULL`. Another clean option is `NOT EXISTS`. For example, to find users without orders, I can use either pattern.

---

# 79. Interview: Why Does LEFT JOIN + WHERE Sometimes Behave Like INNER JOIN?

> Because a condition on the right table in the WHERE clause removes NULL-extended rows. If I need to preserve unmatched left-side rows, I often put the right-table filtering condition inside the ON clause instead.

---

# 80. Interview: How Do You Optimize a JOIN?

> I first inspect the execution plan with `EXPLAIN`, verify that join columns are appropriate and indexed, reduce unnecessary rows and columns, check cardinality and make sure the join condition is correct. I avoid assuming an index or a particular join order is optimal without checking the plan.

---

# 81. JOIN Checklist

```text
□ INNER JOIN
□ LEFT JOIN
□ RIGHT JOIN
□ FULL OUTER JOIN concept
□ CROSS JOIN
□ SELF JOIN
□ ON clause
□ ON vs WHERE
□ One-to-one
□ One-to-many
□ Many-to-many
□ JOIN + GROUP BY
□ JOIN + HAVING
□ JOIN + aggregation
□ LEFT JOIN + IS NULL
□ EXISTS
□ NOT EXISTS
□ Anti-join
□ Semi-join
□ Multiple JOINs
□ JOIN cardinality
□ Duplicate rows
□ Cartesian product
□ JOIN indexes
□ EXPLAIN
□ Pagination with JOINs
```

---

# 82. Final Mental Model

Think about JOINs as relationships:

```text
users
  |
  | 1
  |
  | N
orders
  |
  | 1
  |
  | N
order_items
  |
  | N
  |
  | 1
products
```

A query follows those relationships:

```text
users
  ↓ JOIN
orders
  ↓ JOIN
order_items
  ↓ JOIN
products
```

Then:

```text
WHERE
  ↓
GROUP BY
  ↓
HAVING
  ↓
ORDER BY
  ↓
LIMIT
```

The most important interview mental model is:

```text
INNER JOIN
→ matching rows only

LEFT JOIN
→ everything on the left + matches on the right

LEFT JOIN + right.id IS NULL
→ left rows with no match

EXISTS
→ at least one related row

NOT EXISTS
→ no related row
```

> **JOINs are about relationships, not just syntax. Before writing a JOIN, understand the relationship between the tables and the expected cardinality. That one habit prevents a huge number of SQL bugs and interview mistakes.**
