# SQL — Subqueries & Nested Queries

Subqueries are queries written inside another SQL query. They are extremely common in backend interviews because they test whether you can break a problem into smaller database operations.

---

# 1. What Is a Subquery?

A subquery is a query nested inside another query.

Example:

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The inner query calculates:

```text
average salary
```

The outer query finds employees earning more than that average.

Mental model:

```text
Inner query
    ↓
produces a result
    ↓
outer query uses that result
```

---

# 2. Why Use Subqueries?

Subqueries are useful when a requirement naturally has multiple steps.

For example:

```text
1. Find the average salary.
2. Find employees whose salary is above that average.
```

Instead of doing this manually, SQL can express both steps in one query.

---

# 3. Scalar Subquery

A scalar subquery returns exactly one value.

Example:

```sql
SELECT
    name,
    salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The subquery:

```sql
SELECT AVG(salary)
FROM employees;
```

returns one value.

That value can be used with:

```text
>
<
=
>=
<=
<>
```

---

# 4. Subquery Returning One Row

Example:

```sql
SELECT *
FROM employees
WHERE department_id = (
    SELECT id
    FROM departments
    WHERE name = 'Engineering'
);
```

The inner query must return at most one row for `=` to work.

If it returns multiple rows, the query can fail.

---

# 5. Multiple-Row Subquery

If a subquery can return multiple values, use operators such as:

```text
IN
ANY
ALL
```

Example:

```sql
SELECT *
FROM employees
WHERE department_id IN (
    SELECT id
    FROM departments
    WHERE location = 'Bangalore'
);
```

The inner query may return multiple department IDs.

---

# 6. IN with a Subquery

Example:

```sql
SELECT
    name
FROM employees
WHERE department_id IN (
    SELECT id
    FROM departments
    WHERE name IN ('Engineering', 'Finance')
);
```

This means:

```text
Find departments matching the inner query
        ↓
Find employees belonging to those departments
```

---

# 7. NOT IN

Example:

```sql
SELECT
    name
FROM employees
WHERE department_id NOT IN (
    SELECT id
    FROM departments
    WHERE location = 'Mumbai'
);
```

Be careful with `NULL`.

`NOT IN` can produce surprising results if the subquery returns NULL.

For existence checks, `NOT EXISTS` is often safer and clearer.

---

# 8. EXISTS

`EXISTS` checks whether the subquery returns at least one row.

Example:

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

Meaning:

```text
Does this user have at least one order?
```

If yes, the user is returned.

---

# 9. Why SELECT 1 in EXISTS?

You will commonly see:

```sql
SELECT 1
FROM orders
WHERE ...
```

inside `EXISTS`.

The actual selected value doesn't matter.

`EXISTS` only cares whether a row exists.

So these communicate the same intent:

```sql
SELECT 1
```

or:

```sql
SELECT *
```

But `SELECT 1` makes the intent obvious.

---

# 10. NOT EXISTS

Example:

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

This finds users with no orders.

---

# 11. Correlated Subquery

A correlated subquery refers to a column from the outer query.

Example:

```sql
SELECT
    e.name,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

The inner query depends on:

```text
e.department_id
```

from the outer query.

Meaning:

```text
For each employee:
    calculate the average salary of their department
    compare employee salary with that average
```

---

# 12. Correlated vs Non-Correlated

Non-correlated:

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The inner query is independent of the outer row.

Correlated:

```sql
SELECT *
FROM employees e
WHERE salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

The inner query references the outer query.

---

# 13. Subquery in WHERE

Most common form:

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The subquery supplies a value used for filtering.

---

# 14. Subquery in SELECT

A subquery can appear in the SELECT list.

Example:

```sql
SELECT
    e.name,
    e.salary,
    (
        SELECT AVG(salary)
        FROM employees
    ) AS company_average
FROM employees e;
```

Each result row can show:

```text
employee salary
company average
```

This can be useful, although joins or window functions may be better for more complex reporting.

---

# 15. Subquery in FROM

A subquery in `FROM` is often called a derived table.

Example:

```sql
SELECT *
FROM (
    SELECT
        department_id,
        AVG(salary) AS average_salary
    FROM employees
    GROUP BY department_id
) d;
```

The inner query produces a temporary result set that the outer query can use.

---

# 16. Derived Table with Filtering

Example:

```sql
SELECT *
FROM (
    SELECT
        department_id,
        AVG(salary) AS average_salary
    FROM employees
    GROUP BY department_id
) d
WHERE d.average_salary > 70000;
```

Conceptually:

```text
employees
   ↓
GROUP BY department
   ↓
calculate average
   ↓
derived table
   ↓
filter average > 70000
```

---

# 17. Derived Table Must Have an Alias

Many SQL databases require an alias for a subquery used in `FROM`.

Good:

```sql
FROM (
    SELECT ...
) d
```

Not:

```sql
FROM (
    SELECT ...
)
```

Use a meaningful alias when possible.

---

# 18. Subquery with MAX

Find employees earning the maximum salary:

```sql
SELECT
    name,
    salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
);
```

Notice this can return multiple employees if there is a tie.

---

# 19. Subquery with MIN

Find employees earning the minimum salary:

```sql
SELECT
    name,
    salary
FROM employees
WHERE salary = (
    SELECT MIN(salary)
    FROM employees
);
```

---

# 20. Second Highest Salary

A common interview question.

One approach:

```sql
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

This finds the highest salary below the overall maximum.

---

# 21. Second Highest Distinct Salary

The previous query naturally handles duplicate maximum salaries because it asks for:

```text
MAX(salary) below the maximum
```

Example:

```text
100000
100000
90000
80000
```

Result:

```text
90000
```

---

# 22. Second Highest Using DISTINCT + ORDER BY

Another approach:

```sql
SELECT salary
FROM employees
GROUP BY salary
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

This works well when you want the second distinct salary.

---

# 23. Third Highest Salary

Using ordering:

```sql
SELECT salary
FROM employees
GROUP BY salary
ORDER BY salary DESC
LIMIT 1 OFFSET 2;
```

`OFFSET 2` skips:

```text
1st highest
2nd highest
```

and returns the third distinct salary.

---

# 24. Employees Above Company Average

```sql
SELECT
    name,
    salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

Very common interview problem.

---

# 25. Employees Above Department Average

Using a correlated subquery:

```sql
SELECT
    e.name,
    e.salary,
    e.department_id
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

This is a classic correlated-subquery problem.

---

# 26. Departments Above Company Average Salary

First calculate department averages:

```sql
SELECT
    department_id,
    AVG(salary) AS department_average
FROM employees
GROUP BY department_id;
```

Then compare:

```sql
SELECT *
FROM (
    SELECT
        department_id,
        AVG(salary) AS department_average
    FROM employees
    GROUP BY department_id
) d
WHERE d.department_average > (
    SELECT AVG(salary)
    FROM employees
);
```

---

# 27. Employees in Engineering

If department names are stored separately:

```sql
SELECT *
FROM employees
WHERE department_id IN (
    SELECT id
    FROM departments
    WHERE name = 'Engineering'
);
```

If `name = 'Engineering'` is guaranteed unique, `=` can also be used.

---

# 28. Users Who Placed Orders

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

---

# 29. Users Who Never Placed Orders

Using `NOT EXISTS`:

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

# 30. Products Never Ordered

```sql
SELECT
    p.id,
    p.name
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.id
);
```

This is often safer than `NOT IN` when NULL behavior matters.

---

# 31. Products Ordered at Least Once

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

# 32. Users with Orders Above a Value

Example:

```text
Find users who have at least one order above 50,000.
```

```sql
SELECT
    u.id,
    u.name
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
      AND o.amount > 50000
);
```

---

# 33. Users with No Paid Orders

```sql
SELECT
    u.id,
    u.name
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
      AND o.status = 'PAID'
);
```

Notice the condition is inside the subquery.

This means:

```text
No matching PAID order
```

not necessarily:

```text
No orders at all
```

---

# 34. IN vs EXISTS

Example with `IN`:

```sql
SELECT *
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
);
```

Example with `EXISTS`:

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

Both can express:

```text
users who have orders
```

But their semantics differ in some edge cases, especially around NULLs for `IN`, and the optimizer may transform them depending on the database.

---

# 35. NOT IN vs NOT EXISTS

Potentially dangerous:

```sql
SELECT *
FROM users
WHERE id NOT IN (
    SELECT user_id
    FROM orders
);
```

If the subquery contains NULL, SQL's three-valued logic can cause unexpected results.

Safer existence pattern:

```sql
SELECT *
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

---

# 36. ANY

`ANY` compares a value against the values returned by a subquery.

Example:

```sql
SELECT *
FROM employees
WHERE salary > ANY (
    SELECT salary
    FROM employees
    WHERE department_id = 10
);
```

This means the salary is greater than at least one value returned by the subquery.

---

# 37. ALL

`ALL` requires the comparison to be true for every value returned.

Example:

```sql
SELECT *
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department_id = 10
);
```

This means:

```text
salary > every salary in department 10
```

Conceptually similar to being greater than the maximum value of that set, assuming the same NULL considerations are handled.

---

# 38. ANY vs ALL

Think:

```text
ANY
→ condition true for at least one value

ALL
→ condition true for every value
```

Example:

```text
salary > ANY(...)
```

means:

```text
greater than at least one
```

while:

```text
salary > ALL(...)
```

means:

```text
greater than every value
```

---

# 39. Subquery Returning Multiple Rows with =

This can fail:

```sql
SELECT *
FROM employees
WHERE department_id = (
    SELECT id
    FROM departments
    WHERE location = 'Bangalore'
);
```

If multiple departments are in Bangalore, the inner query returns multiple rows.

Use:

```sql
IN
```

instead:

```sql
WHERE department_id IN (
    SELECT id
    FROM departments
    WHERE location = 'Bangalore'
);
```

---

# 40. Subquery with UPDATE

Subqueries can also be used in `UPDATE`.

Example:

```sql
UPDATE employees
SET salary = salary * 1.10
WHERE department_id = (
    SELECT id
    FROM departments
    WHERE name = 'Engineering'
);
```

Whether this exact form is accepted efficiently depends on the database and query structure.

Always test data-changing queries carefully.

---

# 41. Subquery with DELETE

Example:

```sql
DELETE FROM users
WHERE id NOT IN (
    SELECT user_id
    FROM orders
);
```

Again, beware of NULL behavior.

A `NOT EXISTS` version is usually safer:

```sql
DELETE FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

Never run destructive queries in production without validating the affected rows first.

---

# 42. Subquery with INSERT

A subquery can supply rows for an insert.

Example:

```sql
INSERT INTO archived_users (id, name)
SELECT id, name
FROM users
WHERE status = 'INACTIVE';
```

This is technically a subquery-like query structure through `INSERT ... SELECT`.

---

# 43. Derived Tables vs JOINs

Suppose:

```sql
SELECT *
FROM (
    SELECT
        user_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY user_id
) x;
```

This creates an intermediate result.

You can then join it:

```sql
SELECT
    u.name,
    x.total_spent
FROM users u
JOIN (
    SELECT
        user_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY user_id
) x
    ON x.user_id = u.id;
```

This is useful when an aggregate result needs to be treated as a table.

---

# 44. Subquery vs JOIN

Many SQL problems can be solved using either.

Subquery:

```sql
SELECT *
FROM employees
WHERE department_id IN (
    SELECT id
    FROM departments
    WHERE location = 'Bangalore'
);
```

JOIN:

```sql
SELECT e.*
FROM employees e
JOIN departments d
    ON d.id = e.department_id
WHERE d.location = 'Bangalore';
```

Neither is universally better.

Choose based on:

```text
Readability
Intent
Result shape
NULL behavior
Execution plan
Database optimizer
```

---

# 45. Correlated Subquery Performance

Correlated subqueries can be expensive when the database executes the inner logic repeatedly.

Example:

```sql
SELECT *
FROM employees e
WHERE salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

For large datasets, compare alternatives such as:

```text
JOIN
Derived table
Window function
```

using `EXPLAIN`.

Do not assume every correlated subquery is slow; modern optimizers can transform some queries.

---

# 46. Rewriting Correlated Query with a Derived Table

Instead of:

```sql
SELECT
    e.name,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

You can calculate department averages first:

```sql
SELECT
    e.name,
    e.salary
FROM employees e
JOIN (
    SELECT
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) d
    ON d.department_id = e.department_id
WHERE e.salary > d.avg_salary;
```

This is an important interview comparison.

---

# 47. Window Function Alternative

A window function can solve the same problem more directly:

```sql
SELECT
    name,
    salary,
    department_id
FROM (
    SELECT
        name,
        salary,
        department_id,
        AVG(salary) OVER (
            PARTITION BY department_id
        ) AS department_avg
    FROM employees
) x
WHERE salary > department_avg;
```

Window functions will be covered separately.

---

# 48. Top Salary Per Department with Subquery

A common problem:

```text
Find employees with the highest salary in each department.
```

One approach:

```sql
SELECT
    e.name,
    e.department_id,
    e.salary
FROM employees e
WHERE e.salary = (
    SELECT MAX(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

This returns all employees tied for the highest salary in their department.

---

# 49. Top Salary Per Department with JOIN

Another approach:

```sql
SELECT
    e.name,
    e.department_id,
    e.salary
FROM employees e
JOIN (
    SELECT
        department_id,
        MAX(salary) AS max_salary
    FROM employees
    GROUP BY department_id
) x
    ON x.department_id = e.department_id
   AND x.max_salary = e.salary;
```

Both are useful interview patterns.

---

# 50. Nested Subqueries

Subqueries can be nested.

Example:

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id IN (
        SELECT id
        FROM departments
        WHERE location = 'Bangalore'
    )
);
```

There are multiple levels:

```text
outer query
    ↓
average salary query
    ↓
department query
```

Avoid excessive nesting if a clearer JOIN or CTE can express the logic.

---

# 51. Common Interview Query #1

**Find employees earning more than the company average.**

```sql
SELECT
    name,
    salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

---

# 52. Common Interview Query #2

**Find the second highest distinct salary.**

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

---

# 53. Common Interview Query #3

**Find users who have placed at least one order.**

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

---

# 54. Common Interview Query #4

**Find users who have never placed an order.**

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

# 55. Common Interview Query #5

**Find products that have never been ordered.**

```sql
SELECT
    p.id,
    p.name
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.id
);
```

---

# 56. Common Interview Query #6

**Find employees earning more than their department average.**

```sql
SELECT
    e.name,
    e.salary,
    e.department_id
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

---

# 57. Common Interview Query #7

**Find employees with the maximum salary.**

```sql
SELECT
    name,
    salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
);
```

---

# 58. Common Interview Query #8

**Find users who have an order greater than 50,000.**

```sql
SELECT
    u.id,
    u.name
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
      AND o.amount > 50000
);
```

---

# 59. Common Interview Query #9

**Find departments whose average salary is greater than 70,000.**

```sql
SELECT
    department_id,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 70000;
```

Notice this problem doesn't require a subquery at all.

Use the simplest correct SQL.

---

# 60. Common Interview Query #10

**Find users whose total spending is above the average user spending.**

First calculate spending per user:

```sql
SELECT
    user_id,
    SUM(amount) AS total_spending
FROM orders
GROUP BY user_id;
```

Then compare:

```sql
SELECT
    user_id,
    SUM(amount) AS total_spending
FROM orders
GROUP BY user_id
HAVING SUM(amount) > (
    SELECT AVG(total_spending)
    FROM (
        SELECT
            user_id,
            SUM(amount) AS total_spending
        FROM orders
        GROUP BY user_id
    ) x
);
```

This is a good example of a subquery inside a derived table.

---

# 61. Interview: What Is a Subquery?

> A subquery is a query nested inside another SQL query. I use it when the result of one query is needed by another query, such as finding employees whose salary is above the company average.

---

# 62. Interview: What Is a Correlated Subquery?

> A correlated subquery references a column from the outer query, so its result depends on the current outer row. A common example is comparing an employee's salary with the average salary of that employee's department.

---

# 63. Interview: IN vs EXISTS?

> `IN` compares a value against a set of values returned by a subquery. `EXISTS` checks whether at least one matching row exists. If I only care about existence, `EXISTS` often expresses the requirement more clearly and avoids duplicate parent rows.

---

# 64. Interview: NOT IN vs NOT EXISTS?

> I am careful with `NOT IN` because NULL values in the subquery can affect the result due to SQL's three-valued logic. For existence checks, I generally prefer `NOT EXISTS` when appropriate.

---

# 65. Interview: Can a Subquery Return Multiple Rows?

> Yes, but the outer operator must support multiple values. `IN`, `ANY` and `ALL` can work with multi-row subqueries. Using `=` requires a single compatible value.

---

# 66. Interview: Subquery vs JOIN?

> Both can solve overlapping problems. I choose based on readability, intent and result shape, then verify performance with the execution plan for important queries. For existence checks I often prefer `EXISTS`; when I need columns from both tables, a JOIN is usually natural.

---

# 67. Interview: What Is a Derived Table?

> A derived table is a subquery in the `FROM` clause. It produces an intermediate result set that the outer query can treat like a table.

Example:

```sql
SELECT *
FROM (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) d;
```

---

# 68. Interview: Can a Subquery Be Used in SELECT?

> Yes. A scalar subquery can appear in the SELECT list if it returns a single value. However, for complex reporting, joins or window functions may be clearer or more efficient depending on the query.

---

# 69. Interview: What Happens if = Receives Multiple Rows?

> A scalar comparison such as `=` expects one value. If the subquery returns multiple rows, the database generally raises a subquery cardinality error. If multiple values are expected, I use an operator such as `IN`.

---

# 70. Interview: Are Correlated Subqueries Always Slow?

> No. Performance depends on the database optimizer, indexes, data volume and query structure. I don't assume they are slow; for important queries I compare alternatives and inspect `EXPLAIN`.

---

# 71. Subquery Checklist

```text
□ Scalar subquery
□ Single-row subquery
□ Multi-row subquery
□ IN
□ NOT IN
□ EXISTS
□ NOT EXISTS
□ ANY
□ ALL
□ Correlated subquery
□ Non-correlated subquery
□ Subquery in WHERE
□ Subquery in SELECT
□ Subquery in FROM
□ Derived table
□ Subquery + aggregate
□ Second highest salary
□ Top salary per department
□ Above-average queries
□ Missing relationships
□ IN vs EXISTS
□ NOT IN vs NOT EXISTS
□ Subquery vs JOIN
□ Subquery performance
□ EXPLAIN
```

---

# 72. Final Mental Model

Think of subqueries as:

```text
Question A
   ↓
result from A
   ↓
Question B uses that result
```

Examples:

```text
Average salary
    ↓
Employees above average
```

```text
Existing orders
    ↓
Users who have orders
```

```text
Missing orders
    ↓
Users who never ordered
```

```text
Department average
    ↓
Employees above their department average
```

The key patterns to remember for interviews are:

```text
Scalar value
→ = (subquery)

Multiple values
→ IN (subquery)

At least one matching row
→ EXISTS

No matching row
→ NOT EXISTS

Outer-row-dependent calculation
→ Correlated subquery

Intermediate result
→ Subquery in FROM / derived table
```

> **Subqueries are not about memorizing nested syntax. The important skill is recognizing when one database question depends on the result of another. Once you can see that dependency, choosing between a subquery, JOIN, or window function becomes much easier.**
