# SQL — Interview Problems & Query Practice

This file is a practical SQL problem set for Java backend interviews.

The goal is not just to memorize syntax.

For each problem:

```text
1. Understand the requirement
2. Identify tables and relationships
3. Write the simplest correct query
4. Consider NULL and duplicates
5. Consider performance
6. Explain your approach
```

---

# 1. Find All Users

```sql
SELECT *
FROM users;
```

Interview point:

Avoid `SELECT *` in production APIs when only specific fields are required.

---

# 2. Find Active Users

```sql
SELECT id, name, email
FROM users
WHERE status = 'ACTIVE';
```

---

# 3. Find Users Created in 2026

Prefer a range condition:

```sql
SELECT *
FROM users
WHERE created_at >= '2026-01-01'
  AND created_at < '2027-01-01';
```

This can be more index-friendly than applying a function to `created_at`.

---

# 4. Find Products Above a Price

```sql
SELECT id, name, price
FROM products
WHERE price > 1000;
```

---

# 5. Find Products Between Two Prices

```sql
SELECT id, name, price
FROM products
WHERE price BETWEEN 500 AND 5000;
```

Remember:

`BETWEEN` is inclusive at both boundaries in standard SQL behavior.

---

# 6. Find Users Whose Name Starts With S

```sql
SELECT *
FROM users
WHERE name LIKE 'S%';
```

A trailing wildcard can be more index-friendly than a leading wildcard.

---

# 7. Find Users Whose Name Contains "dev"

```sql
SELECT *
FROM users
WHERE name LIKE '%dev%';
```

A normal B-tree index may not efficiently support a leading-wildcard search.

---

# 8. Find Users with NULL Phone Numbers

Incorrect:

```sql
WHERE phone = NULL
```

Correct:

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

---

# 9. Find Users with Non-NULL Phone Numbers

```sql
SELECT *
FROM users
WHERE phone IS NOT NULL;
```

---

# 10. Find Distinct Cities

```sql
SELECT DISTINCT city
FROM users;
```

---

# 11. Sort Products by Price

```sql
SELECT id, name, price
FROM products
ORDER BY price DESC;
```

---

# 12. Find the Cheapest Product

```sql
SELECT id, name, price
FROM products
ORDER BY price ASC
LIMIT 1;
```

An aggregate alternative:

```sql
SELECT MIN(price)
FROM products;
```

The first query returns the product row; the second returns only the minimum price.

---

# 13. Find the Most Expensive Product

```sql
SELECT id, name, price
FROM products
ORDER BY price DESC
LIMIT 1;
```

---

# 14. Find the Average Product Price

```sql
SELECT AVG(price) AS average_price
FROM products;
```

---

# 15. Find Total Product Count

```sql
SELECT COUNT(*) AS product_count
FROM products;
```

---

# 16. Find Users with More Than 5 Orders

```sql
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS order_count
FROM users u
JOIN orders o
    ON o.user_id = u.id
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 5;
```

---

# 17. Find All Users Including Those Without Orders

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

# 18. Find Users Without Orders

Using `NOT EXISTS`:

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

This is a very common interview problem.

---

# 19. Find Users Who Have at Least One Order

```sql
SELECT u.*
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

---

# 20. Find Number of Orders per User

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

Why `LEFT JOIN`?

Because users with zero orders should still appear.

---

# 21. Find Total Amount Spent by Each User

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

---

# 22. Find Total Paid Amount per User

```sql
SELECT
    u.id,
    u.name,
    COALESCE(
        SUM(
            CASE
                WHEN o.status = 'PAID' THEN o.amount
                ELSE 0
            END
        ),
        0
    ) AS total_paid
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
GROUP BY u.id, u.name;
```

---

# 23. Find Users with Paid Orders

```sql
SELECT DISTINCT u.id, u.name
FROM users u
JOIN orders o
    ON o.user_id = u.id
WHERE o.status = 'PAID';
```

An `EXISTS` version can avoid generating duplicate parent rows:

```sql
SELECT u.id, u.name
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
      AND o.status = 'PAID'
);
```

---

# 24. Find the Highest Order Amount

```sql
SELECT MAX(amount) AS highest_order
FROM orders;
```

---

# 25. Find the Second Highest Salary

One solution:

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

# 26. Second Highest Salary Using DENSE_RANK

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

---

# 27. Third Highest Salary

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
WHERE rnk = 3;
```

---

# 28. Highest Salary per Department

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

If you need all employees tied for the highest salary, use `RANK()` instead of `ROW_NUMBER()`.

---

# 29. Find Duplicate Emails

```sql
SELECT
    email,
    COUNT(*) AS occurrences
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

# 30. Find Duplicate Rows

If a combination should be unique:

```sql
SELECT
    name,
    email,
    COUNT(*) AS occurrences
FROM users
GROUP BY name, email
HAVING COUNT(*) > 1;
```

The columns in the `GROUP BY` should represent the definition of a duplicate for the business requirement.

---

# 31. Delete Duplicate Rows

Do not blindly run a delete.

First identify duplicates:

```sql
SELECT
    email,
    COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

Then decide which row should survive.

A common pattern using `ROW_NUMBER()` is:

```sql
WITH ranked AS (
    SELECT
        id,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY id
        ) AS rn
    FROM users
)
DELETE FROM users
WHERE id IN (
    SELECT id
    FROM ranked
    WHERE rn > 1
);
```

Exact syntax for modifying a CTE result differs by database. Test carefully and use a transaction/back-up strategy where appropriate.

---

# 32. Find the Latest Order for Each User

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

---

# 33. Find the First Order for Each User

```sql
WITH ranked AS (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY created_at ASC, id ASC
        ) AS rn
    FROM orders o
)
SELECT *
FROM ranked
WHERE rn = 1;
```

---

# 34. Find the Top 3 Orders per User

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

# 35. Find the Top 3 Products by Sales

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
LIMIT 3;
```

---

# 36. Find Products Never Ordered

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

# 37. Find Products Ordered at Least Once

```sql
SELECT p.*
FROM products p
WHERE EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.id
);
```

---

# 38. Find Customers Who Bought a Specific Product

```sql
SELECT DISTINCT u.id, u.name
FROM users u
JOIN orders o
    ON o.user_id = u.id
JOIN order_items oi
    ON oi.order_id = o.id
WHERE oi.product_id = 10;
```

---

# 39. Find Customers Who Bought More Than 5 Products

Interpretation:

```text
More than 5 product units
```

```sql
SELECT
    u.id,
    u.name,
    SUM(oi.quantity) AS units
FROM users u
JOIN orders o
    ON o.user_id = u.id
JOIN order_items oi
    ON oi.order_id = o.id
GROUP BY u.id, u.name
HAVING SUM(oi.quantity) > 5;
```

If the requirement means five distinct products, use:

```sql
HAVING COUNT(DISTINCT oi.product_id) > 5;
```

Always clarify what "products" means.

---

# 40. Find Monthly Sales

A database-specific date grouping expression may be used.

Portable conceptual form:

```sql
SELECT
    EXTRACT(YEAR FROM created_at) AS year,
    EXTRACT(MONTH FROM created_at) AS month,
    SUM(amount) AS total_sales
FROM orders
WHERE status = 'PAID'
GROUP BY
    EXTRACT(YEAR FROM created_at),
    EXTRACT(MONTH FROM created_at)
ORDER BY year, month;
```

Date functions differ across MySQL, PostgreSQL, SQL Server and Oracle.

---

# 41. Find Orders from the Last 30 Days

Use a database-appropriate date expression.

Conceptually:

```sql
SELECT *
FROM orders
WHERE created_at >= CURRENT_TIMESTAMP - INTERVAL '30' DAY;
```

Exact interval syntax varies by database.

For an interview, mention the database you are targeting.

---

# 42. Find Orders Between Two Dates

Prefer a half-open range for timestamps:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-08-01'
  AND created_at < '2026-09-01';
```

This avoids common end-of-day timestamp problems.

---

# 43. Find Customers with More Than ₹10,000 Spent

```sql
SELECT
    user_id,
    SUM(amount) AS total_spent
FROM orders
WHERE status = 'PAID'
GROUP BY user_id
HAVING SUM(amount) > 10000;
```

---

# 44. Find Average Order Value

```sql
SELECT AVG(amount) AS average_order_value
FROM orders
WHERE status = 'PAID';
```

---

# 45. Find Order Count by Status

```sql
SELECT
    status,
    COUNT(*) AS order_count
FROM orders
GROUP BY status;
```

---

# 46. Find the Most Common Order Status

```sql
SELECT
    status,
    COUNT(*) AS order_count
FROM orders
GROUP BY status
ORDER BY order_count DESC
LIMIT 1;
```

---

# 47. Find Employees Earning Above Average

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

---

# 48. Find Employees Earning Above Their Department Average

```sql
SELECT e.*
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

This is a correlated subquery.

---

# 49. Same Problem with a CTE

```sql
WITH department_avg AS (
    SELECT
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT e.*
FROM employees e
JOIN department_avg d
    ON d.department_id = e.department_id
WHERE e.salary > d.avg_salary;
```

---

# 50. Running Total of Orders

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

---

# 51. Running Total per User

```sql
SELECT
    id,
    user_id,
    created_at,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY created_at, id
    ) AS user_running_total
FROM orders;
```

---

# 52. Compare Current Order with Previous Order

```sql
SELECT
    id,
    user_id,
    amount,
    LAG(amount) OVER (
        PARTITION BY user_id
        ORDER BY created_at, id
    ) AS previous_amount
FROM orders;
```

---

# 53. Find Orders Larger Than the Previous Order

```sql
WITH order_comparison AS (
    SELECT
        id,
        user_id,
        amount,
        LAG(amount) OVER (
            PARTITION BY user_id
            ORDER BY created_at, id
        ) AS previous_amount
    FROM orders
)
SELECT *
FROM order_comparison
WHERE amount > previous_amount;
```

---

# 54. Find Consecutive Activity

For problems involving:

```text
consecutive login days
consecutive purchases
consecutive events
```

window functions such as:

```text
LAG()
LEAD()
ROW_NUMBER()
```

are often useful.

The exact query depends on the definition of "consecutive."

---

# 55. Find Users with Orders in Every Month of a Period

This is a more advanced aggregation problem.

Conceptually:

```sql
SELECT user_id
FROM orders
WHERE status = 'PAID'
  AND created_at >= :start_date
  AND created_at < :end_date
GROUP BY user_id
HAVING COUNT(DISTINCT month_expression) = :required_months;
```

The month expression is database-specific.

---

# 56. Find Customers Who Bought Every Product in a Category

This is a relational-division style problem.

Conceptually:

```text
For each customer:
    count distinct products bought
    compare against number of products in category
```

One possible structure:

```sql
SELECT u.id
FROM users u
JOIN orders o
    ON o.user_id = u.id
JOIN order_items oi
    ON oi.order_id = o.id
JOIN products p
    ON p.id = oi.product_id
WHERE p.category_id = :categoryId
GROUP BY u.id
HAVING COUNT(DISTINCT p.id) = (
    SELECT COUNT(*)
    FROM products
    WHERE category_id = :categoryId
);
```

Be careful if the category is empty; business rules should define the expected result.

---

# 57. Find Orders with Multiple Items

```sql
SELECT
    order_id,
    COUNT(*) AS item_count
FROM order_items
GROUP BY order_id
HAVING COUNT(*) > 1;
```

If "items" means total quantity:

```sql
HAVING SUM(quantity) > 1;
```

Again, clarify the requirement.

---

# 58. Find the Most Expensive Product per Category

```sql
WITH ranked AS (
    SELECT
        p.*,
        ROW_NUMBER() OVER (
            PARTITION BY category_id
            ORDER BY price DESC, id
        ) AS rn
    FROM products p
)
SELECT *
FROM ranked
WHERE rn = 1;
```

---

# 59. Find All Tied Highest-Priced Products per Category

Use `RANK()`:

```sql
WITH ranked AS (
    SELECT
        p.*,
        RANK() OVER (
            PARTITION BY category_id
            ORDER BY price DESC
        ) AS rnk
    FROM products p
)
SELECT *
FROM ranked
WHERE rnk = 1;
```

---

# 60. Find the Latest Product Price

If prices are stored historically:

```text
product_prices
--------------
product_id
price
effective_at
```

Use:

```sql
WITH ranked AS (
    SELECT
        pp.*,
        ROW_NUMBER() OVER (
            PARTITION BY product_id
            ORDER BY effective_at DESC, id DESC
        ) AS rn
    FROM product_prices pp
)
SELECT *
FROM ranked
WHERE rn = 1;
```

---

# 61. Find Users with Both Paid and Cancelled Orders

```sql
SELECT user_id
FROM orders
GROUP BY user_id
HAVING SUM(CASE WHEN status = 'PAID' THEN 1 ELSE 0 END) > 0
   AND SUM(CASE WHEN status = 'CANCELLED' THEN 1 ELSE 0 END) > 0;
```

---

# 62. Conditional Aggregation

A powerful pattern:

```sql
SELECT
    user_id,
    SUM(CASE WHEN status = 'PAID' THEN amount ELSE 0 END) AS paid_amount,
    SUM(CASE WHEN status = 'CANCELLED' THEN amount ELSE 0 END) AS cancelled_amount
FROM orders
GROUP BY user_id;
```

One query can calculate multiple categories.

---

# 63. Find Users with No Paid Orders

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
      AND o.status = 'PAID'
);
```

This includes users who have:

```text
No orders
```

or:

```text
Only non-paid orders
```

---

# 64. Find Users with Only Paid Orders

Interpretation:

```text
At least one order
AND no non-paid order
```

```sql
SELECT u.id
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
)
AND NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
      AND o.status <> 'PAID'
);
```

NULL status handling should be considered explicitly.

---

# 65. Find Orders with Invalid Users

If foreign keys are not enforced:

```sql
SELECT o.*
FROM orders o
LEFT JOIN users u
    ON u.id = o.user_id
WHERE u.id IS NULL;
```

In a properly constrained database, such invalid references should normally be prevented.

---

# 66. Find Orphan Order Items

```sql
SELECT oi.*
FROM order_items oi
LEFT JOIN orders o
    ON o.id = oi.order_id
WHERE o.id IS NULL;
```

Again, a foreign key should normally prevent these rows.

---

# 67. Find Products with Low Stock

```sql
SELECT *
FROM products
WHERE stock < 10;
```

---

# 68. Find Out-of-Stock Products

```sql
SELECT *
FROM products
WHERE stock = 0;
```

---

# 69. Find Categories with No Products

```sql
SELECT c.*
FROM categories c
LEFT JOIN products p
    ON p.category_id = c.id
WHERE p.id IS NULL;
```

---

# 70. Find Category Product Counts

```sql
SELECT
    c.id,
    c.name,
    COUNT(p.id) AS product_count
FROM categories c
LEFT JOIN products p
    ON p.category_id = c.id
GROUP BY c.id, c.name;
```

Use `COUNT(p.id)`, not `COUNT(*)`, when you want zero for categories with no products.

---

# 71. Find Categories with More Than 10 Products

```sql
SELECT
    c.id,
    c.name,
    COUNT(p.id) AS product_count
FROM categories c
JOIN products p
    ON p.category_id = c.id
GROUP BY c.id, c.name
HAVING COUNT(p.id) > 10;
```

---

# 72. Find the Most Popular Category

Define popularity first.

If popularity means number of products:

```sql
SELECT
    category_id,
    COUNT(*) AS product_count
FROM products
GROUP BY category_id
ORDER BY product_count DESC
LIMIT 1;
```

If popularity means sales, the query must instead use order data.

---

# 73. Find Products Purchased by a User

```sql
SELECT DISTINCT p.*
FROM products p
JOIN order_items oi
    ON oi.product_id = p.id
JOIN orders o
    ON o.id = oi.order_id
WHERE o.user_id = :userId;
```

---

# 74. Find Frequently Bought Together

A common advanced problem:

```text
Find products that are frequently present
in the same order.
```

Conceptually:

```sql
SELECT
    oi1.product_id AS product_a,
    oi2.product_id AS product_b,
    COUNT(*) AS frequency
FROM order_items oi1
JOIN order_items oi2
    ON oi1.order_id = oi2.order_id
   AND oi1.product_id < oi2.product_id
GROUP BY oi1.product_id, oi2.product_id
ORDER BY frequency DESC;
```

This can be expensive on large datasets and needs appropriate filtering/indexing.

---

# 75. Find Customers with the Highest Number of Orders

```sql
SELECT
    user_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY user_id
ORDER BY order_count DESC
LIMIT 1;
```

If ties must all be returned, use a ranking approach.

---

# 76. Find All Customers Tied for Most Orders

```sql
WITH counts AS (
    SELECT
        user_id,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
),
ranked AS (
    SELECT
        user_id,
        order_count,
        RANK() OVER (
            ORDER BY order_count DESC
        ) AS rnk
    FROM counts
)
SELECT *
FROM ranked
WHERE rnk = 1;
```

---

# 77. Find Orders Whose Amount Is Above User Average

```sql
WITH user_avg AS (
    SELECT
        user_id,
        AVG(amount) AS avg_amount
    FROM orders
    GROUP BY user_id
)
SELECT o.*
FROM orders o
JOIN user_avg a
    ON a.user_id = o.user_id
WHERE o.amount > a.avg_amount;
```

---

# 78. Find the Percentage of Paid Orders

Conceptually:

```sql
SELECT
    100.0 *
    SUM(CASE WHEN status = 'PAID' THEN 1 ELSE 0 END)
    / NULLIF(COUNT(*), 0) AS paid_percentage
FROM orders;
```

`NULLIF` avoids division by zero.

Exact numeric behavior depends on the database.

---

# 79. Find Conversion Rate

Suppose:

```text
users
orders
```

A simple order conversion metric could be:

```text
users with at least one order
/
total users
```

Example:

```sql
SELECT
    100.0 *
    COUNT(DISTINCT CASE
        WHEN o.id IS NOT NULL THEN u.id
    END)
    / NULLIF(COUNT(DISTINCT u.id), 0) AS conversion_rate
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id;
```

Define the business metric carefully before writing the SQL.

---

# 80. SQL Problem: Pagination

Basic:

```sql
SELECT id, name, price
FROM products
ORDER BY id
LIMIT 20 OFFSET 40;
```

For high-volume APIs, consider keyset:

```sql
SELECT id, name, price
FROM products
WHERE id > :lastId
ORDER BY id
LIMIT 20;
```

---

# 81. SQL Problem: Latest 20 Orders

```sql
SELECT *
FROM orders
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

A suitable index can help depending on the database and workload.

---

# 82. SQL Problem: Latest 20 Paid Orders

```sql
SELECT *
FROM orders
WHERE status = 'PAID'
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Think about an index aligned with:

```text
status
created_at
id
```

and verify with `EXPLAIN`.

---

# 83. SQL Problem: Users with More Than ₹50,000 Total Spend

```sql
SELECT
    user_id,
    SUM(amount) AS total_spent
FROM orders
WHERE status = 'PAID'
GROUP BY user_id
HAVING SUM(amount) > 50000;
```

---

# 84. SQL Problem: Top 5 Users by Spending

```sql
SELECT
    user_id,
    SUM(amount) AS total_spent
FROM orders
WHERE status = 'PAID'
GROUP BY user_id
ORDER BY total_spent DESC
LIMIT 5;
```

---

# 85. SQL Problem: Monthly Revenue

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

Mention that date functions are database-specific.

---

# 86. SQL Problem: Customers Who Never Purchased

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

---

# 87. SQL Problem: Second Most Expensive Product

```sql
SELECT MAX(price)
FROM products
WHERE price < (
    SELECT MAX(price)
    FROM products
);
```

For the entire row, use a ranking query or join back to the product table.

---

# 88. SQL Problem: Duplicate Emails

```sql
SELECT email
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

If duplicate emails should never be allowed, add:

```sql
UNIQUE(email)
```

after cleaning existing data.

---

# 89. SQL Problem: Delete Old Data Safely

Don't immediately run:

```sql
DELETE FROM orders
WHERE created_at < ...;
```

For large production tables, consider:

```text
Backup / retention requirements
Batch deletes
Transactions
Lock impact
Indexes
Archiving
Partitioning
```

---

# 90. SQL Problem: Find Missing IDs

Do not assume IDs must be gapless.

Sequences/auto-increment values can have gaps because of:

```text
Rollbacks
Deleted rows
Concurrent inserts
Allocation strategies
```

A missing ID is not automatically a problem.

---

# 91. SQL Problem: Detect Duplicate Orders

Define the business uniqueness first.

For example:

```sql
SELECT
    user_id,
    created_at,
    amount,
    COUNT(*) AS occurrences
FROM orders
GROUP BY user_id, created_at, amount
HAVING COUNT(*) > 1;
```

This is only a duplicate candidate query, not proof that the orders are duplicates.

---

# 92. SQL Problem: Update Product Prices

Example:

```sql
UPDATE products
SET price = price * 1.10
WHERE category_id = 5;
```

Before running a production update:

```sql
SELECT id, price
FROM products
WHERE category_id = 5;
```

Verify the target rows first.

---

# 93. SQL Problem: Transactional Update

Suppose placing an order requires:

```text
Insert order
Insert order items
Decrease inventory
```

These operations should generally be treated as one business transaction.

Conceptually:

```text
BEGIN
 ↓
Create order
 ↓
Create order items
 ↓
Update inventory
 ↓
COMMIT
```

If a critical operation fails:

```text
ROLLBACK
```

---

# 94. SQL Problem: Prevent Overselling

A naive flow:

```text
SELECT stock
↓
Java checks stock
↓
UPDATE stock
```

can race under concurrent requests.

A database-side conditional update can be safer:

```sql
UPDATE products
SET stock = stock - :quantity
WHERE id = :productId
  AND stock >= :quantity;
```

Then verify:

```text
affected rows = 1
```

If:

```text
affected rows = 0
```

the requested quantity was not available under that condition.

The exact concurrency strategy should match the application's transaction and locking model.

---

# 95. SQL Problem: Find Orders with Invalid Amount

```sql
SELECT *
FROM orders
WHERE amount < 0;
```

Better long-term protection:

```sql
CHECK (amount >= 0)
```

when supported and appropriate.

---

# 96. SQL Problem: Find NULL Critical Fields

```sql
SELECT *
FROM users
WHERE email IS NULL;
```

If email is mandatory:

```sql
ALTER TABLE users
MODIFY email VARCHAR(255) NOT NULL;
```

Exact syntax is database-specific.

Clean existing data before adding a restrictive constraint.

---

# 97. SQL Problem: Find Slow Queries

Use database monitoring and slow-query facilities.

Then:

```text
1. Identify query
2. Check frequency
3. Check latency
4. Run EXPLAIN
5. Check indexes
6. Check row counts
7. Check locks
8. Optimize
9. Measure again
```

---

# 98. SQL Problem: Avoid N+1

Instead of:

```text
SELECT users
SELECT orders for user 1
SELECT orders for user 2
SELECT orders for user 3
...
```

consider:

```sql
SELECT u.id, u.name, o.id
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id;
```

or an appropriate JPA fetch strategy/DTO query.

The correct solution depends on result size and API needs.

---

# 99. SQL Problem: Find Users with Latest Order

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
SELECT
    u.id,
    u.name,
    r.id AS latest_order_id
FROM users u
LEFT JOIN ranked r
    ON r.user_id = u.id
   AND r.rn = 1;
```

---

# 100. Final Interview Challenge Set

Try solving these without looking at the answers:

```text
1. Find the second-highest salary.

2. Find duplicate emails.

3. Find users with no orders.

4. Find users with more than 5 orders.

5. Find the highest-paid employee in every department.

6. Find the latest order for every user.

7. Find the top 3 products by sales.

8. Find products that have never been ordered.

9. Find each user's total spending.

10. Find users whose spending exceeds ₹50,000.

11. Find employees earning above their department average.

12. Find the second-highest salary without using LIMIT.

13. Find all products tied for the highest price in each category.

14. Find the top 3 orders for every user.

15. Calculate a running total.

16. Compare every order with the user's previous order.

17. Find categories with zero products.

18. Find customers who purchased a particular product.

19. Find users with both PAID and CANCELLED orders.

20. Implement offset pagination.

21. Implement keyset pagination.

22. Find the latest 20 paid orders.

23. Find monthly revenue.

24. Find duplicate business records.

25. Explain how you would optimize a slow query.
```

---

# 101. How to Answer SQL Problems in Interviews

Don't immediately start typing.

Use this structure:

```text
1. Clarify the requirement
2. Identify the tables
3. Identify relationships
4. Decide JOIN / EXISTS / subquery
5. Decide filtering
6. Decide GROUP BY / window function
7. Handle NULLs
8. Consider duplicates
9. Explain complexity/performance
```

Example:

> "We need users without orders. Since I only need to know whether a matching order exists, I'll use `NOT EXISTS`. That avoids returning duplicate users when a user has multiple orders."

Then write:

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

---

# 102. SQL Interview Cheat Sheet

```text
Need matching rows
→ INNER JOIN

Need every left row
→ LEFT JOIN

Need only existence
→ EXISTS

Need no matching row
→ NOT EXISTS

Need grouping
→ GROUP BY

Filter groups
→ HAVING

Need conditional calculation
→ CASE

Need NULL replacement
→ COALESCE

Need ranking
→ ROW_NUMBER / RANK / DENSE_RANK

Need previous row
→ LAG

Need next row
→ LEAD

Need multi-step query
→ CTE

Need unique results
→ DISTINCT

Need combine result sets
→ UNION / UNION ALL

Need large-data pagination
→ Keyset / cursor

Need understand performance
→ EXPLAIN

Need prevent SQL injection
→ Parameters / prepared statements

Need avoid N+1
→ Join / fetch strategy / DTO projection
```

---

# 103. Final Mental Model

For SQL interview problems:

```text
Requirement
   ↓
Tables
   ↓
Relationships
   ↓
JOIN / EXISTS
   ↓
WHERE
   ↓
GROUP BY
   ↓
HAVING
   ↓
Window function / CTE
   ↓
ORDER BY
   ↓
Pagination
   ↓
EXPLAIN
```

The goal isn't to write the most complicated SQL.

The goal is:

```text
Correct
+
Readable
+
Safe
+
Efficient
```

> **Interview shortcut:** When given a SQL problem, explain your reasoning before writing the query. Interviewers often care as much about how you identify relationships, handle duplicates/NULLs, and think about performance as they do about the final SQL.
