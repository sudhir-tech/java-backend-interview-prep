# SQL — Aggregate Functions, GROUP BY & HAVING

This file covers SQL aggregation, grouping and the interview patterns that commonly appear before moving into JOINs and more advanced SQL.

---

# 1. What Is Aggregation?

Aggregation combines multiple rows and produces a summarized result.

Example:

```sql
SELECT COUNT(*)
FROM employees;
```

Instead of returning every employee, the query returns one value:

```text
total employees
```

Common aggregate functions:

```text
COUNT
SUM
AVG
MIN
MAX
```

---

# 2. COUNT

Count all rows:

```sql
SELECT COUNT(*)
FROM users;
```

This counts rows returned by the query.

---

# 3. COUNT(column)

Example:

```sql
SELECT COUNT(phone)
FROM users;
```

`COUNT(column)` counts non-NULL values in that column.

So:

```text
COUNT(*)       → counts rows
COUNT(phone)   → counts non-NULL phone values
```

This distinction is a common interview question.

---

# 4. COUNT(DISTINCT)

Example:

```sql
SELECT COUNT(DISTINCT city)
FROM users;
```

This counts the number of distinct non-NULL cities.

---

# 5. SUM

Example:

```sql
SELECT SUM(price)
FROM products;
```

This returns the total of the selected numeric values.

---

# 6. SUM with a Filter

```sql
SELECT SUM(price)
FROM products
WHERE category = 'Laptop';
```

The filter is applied before the aggregation.

Conceptually:

```text
products
   ↓
WHERE category = Laptop
   ↓
SUM(price)
```

---

# 7. AVG

Example:

```sql
SELECT AVG(price)
FROM products;
```

This calculates the average of non-NULL values.

---

# 8. MIN

```sql
SELECT MIN(price)
FROM products;
```

Returns the minimum value.

---

# 9. MAX

```sql
SELECT MAX(price)
FROM products;
```

Returns the maximum value.

---

# 10. Aggregate Functions and NULL

Most aggregate functions ignore NULL values.

For example:

```sql
SELECT AVG(salary)
FROM employees;
```

NULL salaries are not included in the average.

This is different from treating NULL as zero.

---

# 11. COUNT(*) vs COUNT(column)

Suppose:

```text
phone
------
9999999999
NULL
8888888888
NULL
```

Then:

```sql
COUNT(*)
```

returns:

```text
4
```

while:

```sql
COUNT(phone)
```

returns:

```text
2
```

---

# 12. GROUP BY

`GROUP BY` creates groups of rows based on one or more columns.

Example:

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

Result might be:

```text
department   employee_count
-----------  --------------
Engineering  20
HR           5
Finance      8
```

---

# 13. Why GROUP BY Is Useful

Without grouping:

```sql
SELECT COUNT(*)
FROM employees;
```

returns one total.

With grouping:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

returns one count for each department.

---

# 14. GROUP BY Multiple Columns

Example:

```sql
SELECT department, city, COUNT(*) AS employee_count
FROM employees
GROUP BY department, city;
```

The group is identified by:

```text
department + city
```

---

# 15. GROUP BY and SELECT

A useful rule:

When a query contains `GROUP BY`, selected expressions generally need to be either:

```text
Grouped expressions
```

or:

```text
Aggregate expressions
```

Example:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

Valid.

---

# 16. Invalid Grouping Concept

This is problematic in standard SQL:

```sql
SELECT department, name, COUNT(*)
FROM employees
GROUP BY department;
```

Why?

Because multiple employees can exist within one department, so which `name` should the database return?

Some database configurations may allow non-grouped columns with special behavior, but relying on that is unsafe. In MySQL, `ONLY_FULL_GROUP_BY` helps prevent ambiguous queries.

---

# 17. GROUP BY with SUM

Example:

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

This calculates total salary per department.

---

# 18. GROUP BY with AVG

```sql
SELECT department, AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

This calculates average salary per department.

---

# 19. GROUP BY with MIN and MAX

```sql
SELECT
    department,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees
GROUP BY department;
```

---

# 20. GROUP BY with Multiple Aggregates

```sql
SELECT
    department,
    COUNT(*) AS employee_count,
    SUM(salary) AS total_salary,
    AVG(salary) AS average_salary,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees
GROUP BY department;
```

This produces a summary for each department.

---

# 21. HAVING

`HAVING` filters groups after aggregation.

Example:

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

Only departments with more than five employees are returned.

---

# 22. WHERE vs HAVING

This is one of the most common SQL interview questions.

`WHERE`:

```text
Filters rows
Before grouping
```

`HAVING`:

```text
Filters groups
After grouping
```

Example:

```sql
SELECT department, COUNT(*)
FROM employees
WHERE salary > 50000
GROUP BY department
HAVING COUNT(*) >= 5;
```

Logical flow:

```text
employees
   ↓
WHERE salary > 50000
   ↓
GROUP BY department
   ↓
HAVING COUNT(*) >= 5
```

---

# 23. Can HAVING Be Used Without GROUP BY?

Yes, depending on the database and query.

Example:

```sql
SELECT COUNT(*) AS employee_count
FROM employees
HAVING COUNT(*) > 100;
```

The query produces a single aggregate group and then filters it.

---

# 24. Can WHERE Use Aggregate Functions?

Generally, no.

This is not valid:

```sql
SELECT department, COUNT(*)
FROM employees
WHERE COUNT(*) > 5
GROUP BY department;
```

Use:

```sql
HAVING COUNT(*) > 5;
```

because the aggregate is evaluated at the grouping stage.

---

# 25. Can HAVING Filter Non-Aggregated Group Columns?

Yes.

Example:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING department <> 'HR';
```

However, a condition that can be applied before grouping is often better expressed in `WHERE`, because it can reduce rows earlier.

---

# 26. WHERE + GROUP BY + HAVING

Example:

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
WHERE salary >= 50000
GROUP BY department
HAVING COUNT(*) >= 5;
```

Meaning:

```text
1. Keep employees earning >= 50000
2. Group them by department
3. Count each group
4. Keep groups with at least 5 employees
```

---

# 27. ORDER BY Aggregate

You can sort by an aggregate result.

Example:

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
ORDER BY employee_count DESC;
```

This returns departments with the most employees first.

---

# 28. Top Department by Employee Count

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
ORDER BY employee_count DESC
LIMIT 1;
```

---

# 29. Top 3 Departments by Employee Count

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
ORDER BY employee_count DESC
LIMIT 3;
```

If ties need special treatment, window functions may be more appropriate.

---

# 30. Total Number of Orders

```sql
SELECT COUNT(*) AS total_orders
FROM orders;
```

---

# 31. Total Revenue

```sql
SELECT SUM(amount) AS total_revenue
FROM orders
WHERE status = 'PAID';
```

---

# 32. Average Order Value

```sql
SELECT AVG(amount) AS average_order_value
FROM orders
WHERE status = 'PAID';
```

---

# 33. Highest Order Value

```sql
SELECT MAX(amount) AS highest_order
FROM orders;
```

---

# 34. Lowest Order Value

```sql
SELECT MIN(amount) AS lowest_order
FROM orders;
```

---

# 35. Number of Orders per User

```sql
SELECT
    user_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY user_id;
```

---

# 36. Users with More Than 5 Orders

```sql
SELECT
    user_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

This is a very common interview pattern.

---

# 37. Revenue per User

```sql
SELECT
    user_id,
    SUM(amount) AS total_spent
FROM orders
WHERE status = 'PAID'
GROUP BY user_id;
```

---

# 38. Users Spending More Than 100,000

```sql
SELECT
    user_id,
    SUM(amount) AS total_spent
FROM orders
WHERE status = 'PAID'
GROUP BY user_id
HAVING SUM(amount) > 100000;
```

---

# 39. Orders by Status

```sql
SELECT
    status,
    COUNT(*) AS order_count
FROM orders
GROUP BY status;
```

Example:

```text
PENDING      10
PAID         80
CANCELLED     7
```

---

# 40. Revenue by Status

```sql
SELECT
    status,
    SUM(amount) AS total_amount
FROM orders
GROUP BY status;
```

Be careful with business meaning: a cancelled order's amount is not necessarily revenue.

---

# 41. Products by Category

```sql
SELECT
    category,
    COUNT(*) AS product_count
FROM products
GROUP BY category;
```

---

# 42. Average Price by Category

```sql
SELECT
    category,
    AVG(price) AS average_price
FROM products
GROUP BY category;
```

---

# 43. Maximum Product Price by Category

```sql
SELECT
    category,
    MAX(price) AS maximum_price
FROM products
GROUP BY category;
```

---

# 44. Minimum Product Price by Category

```sql
SELECT
    category,
    MIN(price) AS minimum_price
FROM products
GROUP BY category;
```

---

# 45. Categories with More Than 10 Products

```sql
SELECT
    category,
    COUNT(*) AS product_count
FROM products
GROUP BY category
HAVING COUNT(*) > 10;
```

---

# 46. GROUP BY with Date

Suppose:

```text
created_at
```

contains timestamps.

A common reporting query might group by date:

```sql
SELECT
    DATE(created_at) AS order_date,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATE(created_at);
```

This is useful for reporting but can have indexing implications when used over large datasets.

---

# 47. Grouping by Month

MySQL example:

```sql
SELECT
    YEAR(created_at) AS year,
    MONTH(created_at) AS month,
    COUNT(*) AS order_count
FROM orders
GROUP BY
    YEAR(created_at),
    MONTH(created_at)
ORDER BY
    year,
    month;
```

---

# 48. Grouping by Year

```sql
SELECT
    YEAR(created_at) AS year,
    COUNT(*) AS order_count
FROM orders
GROUP BY YEAR(created_at)
ORDER BY year;
```

---

# 49. Conditional Aggregation

You can combine `SUM` or `COUNT` with `CASE`.

Example:

```sql
SELECT
    COUNT(*) AS total_orders,
    SUM(CASE WHEN status = 'PAID' THEN 1 ELSE 0 END) AS paid_orders,
    SUM(CASE WHEN status = 'CANCELLED' THEN 1 ELSE 0 END) AS cancelled_orders
FROM orders;
```

This produces multiple metrics in one query.

---

# 50. Conditional SUM

Example:

```sql
SELECT
    SUM(CASE
        WHEN status = 'PAID' THEN amount
        ELSE 0
    END) AS paid_amount
FROM orders;
```

This is a powerful reporting pattern.

---

# 51. Conditional Aggregation by Group

Example:

```sql
SELECT
    user_id,
    SUM(CASE WHEN status = 'PAID' THEN amount ELSE 0 END) AS paid_amount,
    SUM(CASE WHEN status = 'CANCELLED' THEN amount ELSE 0 END) AS cancelled_amount
FROM orders
GROUP BY user_id;
```

---

# 52. COUNT with CASE

Another pattern:

```sql
SELECT
    COUNT(CASE WHEN status = 'PAID' THEN 1 END) AS paid_orders
FROM orders;
```

Because `COUNT(expression)` ignores NULL, only matching rows contribute.

---

# 53. SUM(CASE) vs COUNT(CASE)

Both can be used for conditional counting.

Example:

```sql
SUM(CASE WHEN status = 'PAID' THEN 1 ELSE 0 END)
```

and:

```sql
COUNT(CASE WHEN status = 'PAID' THEN 1 END)
```

can produce the same count.

The `SUM(CASE)` form makes the numeric counting behavior explicit.

---

# 54. GROUP BY NULL

If the grouped column contains NULL:

```sql
SELECT
    department,
    COUNT(*)
FROM employees
GROUP BY department;
```

Rows where `department` is NULL are generally grouped together as one group.

---

# 55. COUNT and DISTINCT

Find the number of unique users who placed orders:

```sql
SELECT COUNT(DISTINCT user_id)
FROM orders;
```

This is different from:

```sql
SELECT COUNT(*)
FROM orders;
```

The first counts unique users; the second counts orders.

---

# 56. Unique Customers by Month

A reporting query might be:

```sql
SELECT
    YEAR(created_at) AS year,
    MONTH(created_at) AS month,
    COUNT(DISTINCT user_id) AS unique_customers
FROM orders
GROUP BY
    YEAR(created_at),
    MONTH(created_at);
```

---

# 57. Average Can Be Misleading

Suppose order values are:

```text
100
100
100
10000
```

The average is:

```text
2575
```

This may not represent a typical order.

For analytics, also consider:

```text
MEDIAN
PERCENTILES
DISTRIBUTION
```

SQL support varies by database.

---

# 58. SUM and NULL

Suppose:

```text
amount
------
100
200
NULL
```

Then:

```sql
SUM(amount)
```

returns:

```text
300
```

NULL does not contribute to the sum.

If all rows are NULL, aggregate results such as `SUM` can return NULL rather than zero.

Use:

```sql
COALESCE(SUM(amount), 0)
```

when business logic requires zero.

---

# 59. AVG and NULL

For:

```text
100
200
NULL
```

```sql
AVG(amount)
```

is:

```text
150
```

not:

```text
100
```

because the NULL value is ignored.

---

# 60. GROUP BY and Performance

Grouping can require:

```text
Sorting
Hashing
Temporary structures
Scanning many rows
```

The database optimizer decides the physical execution strategy.

For large datasets, inspect:

```sql
EXPLAIN
```

when performance matters.

---

# 61. Filtering Before GROUP BY

Compare:

```sql
SELECT department, COUNT(*)
FROM employees
WHERE status = 'ACTIVE'
GROUP BY department;
```

The `WHERE` condition can reduce the rows that need to be grouped.

This is usually preferable to grouping everything and then filtering when the condition is row-level.

---

# 62. HAVING After GROUP BY

Example:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 10;
```

The database needs group information to evaluate:

```text
COUNT(*) > 10
```

---

# 63. Logical Query Order

For an aggregate query:

```sql
SELECT department, COUNT(*)
FROM employees
WHERE salary > 50000
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY COUNT(*) DESC;
```

A useful logical model is:

```text
FROM
↓
WHERE
↓
GROUP BY
↓
HAVING
↓
SELECT
↓
ORDER BY
```

The optimizer may physically execute operations differently.

---

# 64. Aggregate Query with LIMIT

Example:

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 3;
```

This finds the three departments with the highest average salary.

---

# 65. Common Interview Query #1

**Find the number of employees in each department.**

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

---

# 66. Common Interview Query #2

**Find departments having more than 10 employees.**

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 10;
```

---

# 67. Common Interview Query #3

**Find the average salary by department.**

```sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

---

# 68. Common Interview Query #4

**Find the highest salary in each department.**

```sql
SELECT
    department,
    MAX(salary) AS highest_salary
FROM employees
GROUP BY department;
```

Important:

This gives the salary value, not necessarily the employee who earns it.

Finding the employee requires a different pattern, such as a join, subquery or window function.

---

# 69. Common Interview Query #5

**Find users who placed more than 3 orders.**

```sql
SELECT
    user_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 3;
```

---

# 70. Common Interview Query #6

**Find total spending per user.**

```sql
SELECT
    user_id,
    SUM(amount) AS total_spending
FROM orders
GROUP BY user_id;
```

---

# 71. Common Interview Query #7

**Find the top 5 users by spending.**

```sql
SELECT
    user_id,
    SUM(amount) AS total_spending
FROM orders
GROUP BY user_id
ORDER BY total_spending DESC
LIMIT 5;
```

---

# 72. Common Interview Query #8

**Find the number of unique customers.**

```sql
SELECT COUNT(DISTINCT user_id) AS unique_customers
FROM orders;
```

---

# 73. Common Interview Query #9

**Find categories containing more than 20 products.**

```sql
SELECT
    category,
    COUNT(*) AS product_count
FROM products
GROUP BY category
HAVING COUNT(*) > 20;
```

---

# 74. Common Interview Query #10

**Find monthly order counts.**

```sql
SELECT
    YEAR(created_at) AS year,
    MONTH(created_at) AS month,
    COUNT(*) AS order_count
FROM orders
GROUP BY
    YEAR(created_at),
    MONTH(created_at)
ORDER BY
    year,
    month;
```

---

# 75. Interview: What Is GROUP BY?

> `GROUP BY` combines rows with the same grouping values so aggregate functions can calculate a result for each group. For example, I can group employees by department and use `COUNT(*)` to find the number of employees in each department.

---

# 76. Interview: WHERE vs HAVING

> `WHERE` filters individual rows before grouping, while `HAVING` filters groups after aggregation. If I need employees with salary above 50,000, I use `WHERE`. If I need departments containing more than five employees, I use `HAVING`.

---

# 77. Interview: COUNT(*) vs COUNT(column)

> `COUNT(*)` counts rows, while `COUNT(column)` counts only rows where that column is not NULL. So if a column has two NULL values in a four-row table, `COUNT(*)` returns four while `COUNT(column)` returns two.

---

# 78. Interview: What Does COUNT(DISTINCT) Do?

> It counts unique non-NULL values. For example, `COUNT(DISTINCT user_id)` tells me how many unique users placed orders rather than how many total orders were placed.

---

# 79. Interview: Can We Use Aggregate Functions in WHERE?

> Generally no, because `WHERE` filters rows before aggregation. If I need to filter based on an aggregate such as `COUNT(*) > 5`, I use `HAVING`.

---

# 80. Interview: Can We Use WHERE and HAVING Together?

> Yes. I use `WHERE` to reduce the individual rows before grouping and `HAVING` to filter the resulting groups. This can also improve efficiency because fewer rows may need to be grouped.

---

# 81. Interview: How Do You Find the Highest Salary Per Department?

> If I only need the salary value, I can use `MAX(salary)` with `GROUP BY department`. If I also need the employee's details, I would use a join, subquery or window function because the aggregate alone doesn't identify which employee owns that salary.

---

# 82. Interview: Why Does GROUP BY Sometimes Need All Selected Columns?

> When a query groups rows, every selected expression that isn't aggregated generally needs to be part of the grouping or be functionally dependent in a way the database accepts. Otherwise the database cannot unambiguously determine which value to return for a group.

---

# 83. Interview: How Do You Count Conditional Rows?

> A common approach is conditional aggregation, for example `SUM(CASE WHEN status = 'PAID' THEN 1 ELSE 0 END)`. This lets me calculate multiple conditional metrics in one query.

---

# 84. Interview: What Happens to NULL in COUNT?

> `COUNT(*)` includes the row even if columns are NULL. `COUNT(column)` ignores rows where that column is NULL. This distinction is important when measuring completeness or optional fields.

---

# 85. Practical Backend Example

Suppose an e-commerce application has:

```text
orders
----------------------------
id
user_id
amount
status
created_at
```

Business requirement:

```text
Find users who have successfully paid
more than 5 orders and spent over 100,000.
```

Query:

```sql
SELECT
    user_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_spent
FROM orders
WHERE status = 'PAID'
GROUP BY user_id
HAVING COUNT(*) > 5
   AND SUM(amount) > 100000
ORDER BY total_spent DESC;
```

Think through it:

```text
WHERE
↓
only PAID orders

GROUP BY
↓
one group per user

COUNT / SUM
↓
calculate user metrics

HAVING
↓
keep qualifying users

ORDER BY
↓
highest spending first
```

---

# 86. Aggregation Checklist

```text
□ COUNT(*)
□ COUNT(column)
□ COUNT(DISTINCT column)
□ SUM
□ AVG
□ MIN
□ MAX
□ GROUP BY
□ GROUP BY multiple columns
□ HAVING
□ WHERE + GROUP BY
□ GROUP BY + HAVING
□ ORDER BY aggregate
□ LIMIT after aggregation
□ Conditional aggregation
□ CASE with aggregates
□ NULL behavior
□ Aggregate performance
```

---

# 87. Final Mental Model

Think of aggregation like this:

```text
Raw rows
   ↓
Filter with WHERE
   ↓
Create groups with GROUP BY
   ↓
Calculate COUNT/SUM/AVG/MIN/MAX
   ↓
Filter groups with HAVING
   ↓
Sort with ORDER BY
   ↓
Limit results
```

For backend work:

```text
API request
    ↓
Business requirement
    ↓
SQL aggregation
    ↓
Database calculates summary
    ↓
DTO / response
```

> **Aggregation is where SQL starts becoming powerful for real business queries. Once you are comfortable with `GROUP BY`, `HAVING`, `COUNT`, `SUM` and conditional aggregation, you're ready for one of the most important SQL interview topics: JOINs.**
