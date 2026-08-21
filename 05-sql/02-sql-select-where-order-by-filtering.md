# SQL — SELECT, WHERE, ORDER BY & Filtering

This file focuses on writing and understanding everyday SQL queries. These are some of the most frequently used SQL operations in Java backend development and SQL interviews.

---

# 1. SELECT

`SELECT` retrieves data from one or more tables.

Example:

```sql
SELECT *
FROM users;
```

This returns all columns and rows from `users`.

In application code, avoid `SELECT *` when you only need specific columns.

Prefer:

```sql
SELECT id, name, email
FROM users;
```

---

# 2. Selecting Specific Columns

Example:

```sql
SELECT name, price
FROM products;
```

Only these columns are returned:

```text
name
price
```

This can improve readability and may reduce unnecessary data transfer.

---

# 3. Column Aliases

Use `AS` to give a result column a different name.

```sql
SELECT
    name AS product_name,
    price AS product_price
FROM products;
```

Result:

```text
product_name | product_price
```

`AS` is optional in many SQL dialects:

```sql
SELECT name product_name
FROM products;
```

Using `AS` is generally clearer.

---

# 4. Table Aliases

Example:

```sql
SELECT p.id, p.name
FROM products AS p;
```

Now:

```text
p
```

represents:

```text
products
```

Aliases become especially useful with joins.

---

# 5. WHERE

`WHERE` filters rows.

```sql
SELECT id, name, price
FROM products
WHERE price > 50000;
```

Only products whose price is greater than 50,000 are returned.

---

# 6. Equality

```sql
SELECT *
FROM users
WHERE id = 10;
```

This finds the row where:

```text
id = 10
```

---

# 7. Not Equal

Common syntax:

```sql
WHERE status <> 'INACTIVE'
```

MySQL also supports:

```sql
WHERE status != 'INACTIVE'
```

`<>` is standard SQL syntax.

---

# 8. Greater Than

```sql
SELECT *
FROM products
WHERE price > 10000;
```

---

# 9. Less Than

```sql
SELECT *
FROM products
WHERE price < 10000;
```

---

# 10. Greater Than or Equal

```sql
SELECT *
FROM products
WHERE price >= 10000;
```

---

# 11. Less Than or Equal

```sql
SELECT *
FROM products
WHERE price <= 10000;
```

---

# 12. AND

`AND` requires both conditions to be true.

```sql
SELECT *
FROM products
WHERE price > 10000
  AND stock > 0;
```

Meaning:

```text
price > 10000
AND
stock > 0
```

---

# 13. OR

`OR` requires at least one condition to be true.

```sql
SELECT *
FROM products
WHERE category = 'Laptop'
   OR category = 'Monitor';
```

---

# 14. Combining AND and OR

Be careful with operator precedence.

Example:

```sql
SELECT *
FROM products
WHERE category = 'Laptop'
   OR category = 'Monitor'
  AND price > 50000;
```

This is interpreted according to SQL operator precedence, not necessarily how a reader might visually group it.

Use parentheses:

```sql
SELECT *
FROM products
WHERE (category = 'Laptop'
    OR category = 'Monitor')
  AND price > 50000;
```

This makes the intended logic explicit.

---

# 15. NOT

Example:

```sql
SELECT *
FROM products
WHERE NOT status = 'DISCONTINUED';
```

Another form:

```sql
SELECT *
FROM products
WHERE status <> 'DISCONTINUED';
```

---

# 16. BETWEEN

`BETWEEN` checks an inclusive range.

```sql
SELECT *
FROM products
WHERE price BETWEEN 10000 AND 50000;
```

Equivalent conceptually to:

```sql
WHERE price >= 10000
  AND price <= 50000
```

---

# 17. BETWEEN with Dates

Example:

```sql
SELECT *
FROM orders
WHERE order_date BETWEEN '2026-01-01' AND '2026-01-31';
```

Be careful when the column contains a time component.

For a `DATETIME`, a half-open range is often safer:

```sql
WHERE order_date >= '2026-01-01'
  AND order_date <  '2026-02-01'
```

This includes the entire month without relying on a particular end-of-day timestamp.

---

# 18. IN

`IN` checks whether a value belongs to a list.

```sql
SELECT *
FROM products
WHERE category IN (
    'Laptop',
    'Monitor',
    'Keyboard'
);
```

This is cleaner than:

```sql
WHERE category = 'Laptop'
   OR category = 'Monitor'
   OR category = 'Keyboard';
```

---

# 19. NOT IN

```sql
SELECT *
FROM products
WHERE category NOT IN (
    'Laptop',
    'Monitor'
);
```

Be careful with `NULL` values when using `NOT IN`; SQL's three-valued logic can produce unexpected results.

---

# 20. LIKE

`LIKE` performs pattern matching.

Example:

```sql
SELECT *
FROM users
WHERE name LIKE 'Sud%';
```

`%` means:

```text
zero or more characters
```

---

# 21. LIKE with Prefix

```sql
WHERE name LIKE 'Sud%'
```

Matches values beginning with:

```text
Sud
```

Examples:

```text
Sudhir
Sudha
Sud
```

---

# 22. LIKE with Suffix

```sql
WHERE email LIKE '%@gmail.com';
```

Matches emails ending with:

```text
@gmail.com
```

---

# 23. LIKE with Contains

```sql
WHERE name LIKE '%dev%';
```

Matches values containing:

```text
dev
```

For example:

```text
developer
devops
backend-dev
```

---

# 24. Underscore `_`

`_` matches exactly one character.

Example:

```sql
WHERE name LIKE 'S_d%';
```

The underscore represents one character between:

```text
S
```

and:

```text
d
```

---

# 25. Case Sensitivity

Whether `LIKE` is case-sensitive depends on the database and, in MySQL, the character set/collation.

Do not blindly assume:

```text
LIKE = always case-sensitive
```

Check the column/database collation when behavior matters.

---

# 26. ORDER BY

`ORDER BY` sorts the result.

Ascending:

```sql
SELECT *
FROM products
ORDER BY price ASC;
```

Descending:

```sql
SELECT *
FROM products
ORDER BY price DESC;
```

---

# 27. ASC and DESC

`ASC`:

```text
small → large
A → Z
old → new
```

`DESC`:

```text
large → small
Z → A
new → old
```

For numbers:

```sql
ORDER BY price DESC;
```

returns the highest prices first.

---

# 28. Multiple ORDER BY Columns

Example:

```sql
SELECT *
FROM products
ORDER BY category ASC,
         price DESC;
```

The database first sorts by:

```text
category
```

and then sorts rows within each category by:

```text
price DESC
```

---

# 29. ORDER BY an Alias

You can often order by a selected alias.

```sql
SELECT
    name,
    price * 0.9 AS discounted_price
FROM products
ORDER BY discounted_price;
```

This is useful for computed values.

---

# 30. LIMIT

MySQL supports:

```sql
SELECT *
FROM products
LIMIT 10;
```

This returns at most 10 rows.

---

# 31. LIMIT with OFFSET

```sql
SELECT *
FROM products
ORDER BY id
LIMIT 10 OFFSET 20;
```

Conceptually:

```text
Skip 20
Return next 10
```

Always use a deterministic `ORDER BY` when implementing pagination.

---

# 32. MySQL LIMIT Syntax

MySQL also supports:

```sql
SELECT *
FROM products
ORDER BY id
LIMIT 20, 10;
```

Meaning:

```text
OFFSET = 20
LIMIT  = 10
```

For readability, many teams prefer:

```sql
LIMIT 10 OFFSET 20
```

---

# 33. DISTINCT

`DISTINCT` removes duplicate result combinations.

```sql
SELECT DISTINCT category
FROM products;
```

If categories are:

```text
Laptop
Laptop
Monitor
Monitor
Keyboard
```

the result is:

```text
Laptop
Monitor
Keyboard
```

---

# 34. DISTINCT on Multiple Columns

```sql
SELECT DISTINCT category, brand
FROM products;
```

Uniqueness is evaluated on the combination:

```text
category + brand
```

not independently on each column.

---

# 35. NULL

`NULL` represents missing or unknown data.

Example:

```text
phone = NULL
```

does not mean:

```text
phone = ''
```

and does not mean:

```text
phone = 0
```

---

# 36. IS NULL

Correct:

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

Incorrect:

```sql
WHERE phone = NULL;
```

---

# 37. IS NOT NULL

```sql
SELECT *
FROM users
WHERE phone IS NOT NULL;
```

This returns rows where phone has a non-NULL value.

---

# 38. COALESCE

`COALESCE` returns the first non-NULL expression.

Example:

```sql
SELECT
    name,
    COALESCE(phone, 'Not provided') AS phone
FROM users;
```

If `phone` is NULL:

```text
Not provided
```

is returned.

---

# 39. NULLIF

`NULLIF(a, b)` returns:

```text
NULL if a = b
```

otherwise:

```text
a
```

Example:

```sql
SELECT NULLIF(stock, 0)
FROM products;
```

This can be useful when handling values that should not be treated as meaningful zeros.

---

# 40. CASE

`CASE` provides conditional logic.

Example:

```sql
SELECT
    name,
    price,
    CASE
        WHEN price >= 50000 THEN 'EXPENSIVE'
        WHEN price >= 10000 THEN 'MEDIUM'
        ELSE 'BUDGET'
    END AS price_category
FROM products;
```

---

# 41. CASE in ORDER BY

You can use conditional expressions for custom sorting.

Example:

```sql
SELECT *
FROM orders
ORDER BY
    CASE
        WHEN status = 'PENDING' THEN 1
        WHEN status = 'PAID' THEN 2
        ELSE 3
    END;
```

This can place important states first.

---

# 42. Filtering Text

Example:

```sql
SELECT id, name
FROM users
WHERE name LIKE 'A%';
```

This finds users whose names start with `A`.

---

# 43. Filtering Numeric Values

```sql
SELECT id, name, price
FROM products
WHERE price >= 50000;
```

---

# 44. Filtering Dates

Example:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-08-01'
  AND created_at <  '2026-09-01';
```

This is often preferable to manipulating the date column with a function.

---

# 45. Why Avoid Functions on Indexed Columns?

Consider:

```sql
WHERE YEAR(created_at) = 2026
```

Depending on the database and available indexes, applying a function to the column can make efficient index access harder.

Prefer:

```sql
WHERE created_at >= '2026-01-01'
  AND created_at <  '2027-01-01'
```

This keeps the column directly comparable to a range.

---

# 46. Filtering Boolean Values

Depending on the database/schema:

```sql
SELECT *
FROM users
WHERE active = TRUE;
```

or:

```sql
WHERE active = 1;
```

Use the convention supported by your database and schema.

---

# 47. Operator Precedence

A useful simplified order to remember is:

```text
NOT
AND
OR
```

Therefore:

```sql
A OR B AND C
```

is generally interpreted as:

```sql
A OR (B AND C)
```

When in doubt:

```sql
(A OR B) AND C
```

Use parentheses.

---

# 48. SQL Injection Reminder

Never create SQL by concatenating untrusted user input.

Bad:

```java
String sql =
    "SELECT * FROM users WHERE name = '" + name + "'";
```

Prefer:

```text
PreparedStatement
Parameterized queries
JPA/Hibernate parameter binding
```

---

# 49. Prepared Statement Example

Conceptually:

```sql
SELECT *
FROM users
WHERE email = ?;
```

Then the application binds:

```text
email
```

as a parameter.

The value is treated as data rather than SQL syntax.

---

# 50. Query Readability

Prefer:

```sql
SELECT
    p.id,
    p.name,
    p.price
FROM products p
WHERE p.stock > 0
  AND p.price >= 10000
ORDER BY p.price DESC;
```

over a single long line.

Readable SQL is easier to review and troubleshoot.

---

# 51. Filtering Before Sorting

A query:

```sql
SELECT id, name, price
FROM products
WHERE stock > 0
ORDER BY price DESC;
```

logically:

```text
Find matching rows
      ↓
Sort result
```

The database optimizer may execute operations differently internally, but SQL semantics produce the same result.

---

# 52. Filtering Before LIMIT

Example:

```sql
SELECT *
FROM products
WHERE stock > 0
ORDER BY price DESC
LIMIT 10;
```

This means:

```text
Consider available products
      ↓
Sort them
      ↓
Return top 10
```

---

# 53. Top-N Query

Find the five most expensive products:

```sql
SELECT id, name, price
FROM products
ORDER BY price DESC
LIMIT 5;
```

This is a very common interview query.

---

# 54. Cheapest Products

```sql
SELECT id, name, price
FROM products
ORDER BY price ASC
LIMIT 5;
```

---

# 55. Latest Records

If `created_at` represents creation time:

```sql
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 10;
```

For deterministic ordering when timestamps can tie:

```sql
ORDER BY created_at DESC, id DESC;
```

---

# 56. Oldest Records

```sql
SELECT *
FROM orders
ORDER BY created_at ASC
LIMIT 10;
```

---

# 57. Find a Specific User

```sql
SELECT id, name, email
FROM users
WHERE id = 100;
```

If `id` is the primary key, this is an efficient lookup when the database uses the primary-key index.

---

# 58. Find Users by Email

```sql
SELECT id, name
FROM users
WHERE email = 'sudhir@example.com';
```

If email must be unique and is frequently queried, a unique index/constraint is typically appropriate.

---

# 59. Search Products by Price Range

```sql
SELECT id, name, price
FROM products
WHERE price >= 20000
  AND price <= 50000
ORDER BY price ASC;
```

---

# 60. Search Products by Multiple Categories

```sql
SELECT id, name, category
FROM products
WHERE category IN (
    'Laptop',
    'Monitor',
    'Keyboard'
)
ORDER BY name ASC;
```

---

# 61. Search by Name

Prefix search:

```sql
SELECT *
FROM products
WHERE name LIKE 'Mac%';
```

Contains search:

```sql
SELECT *
FROM products
WHERE name LIKE '%Mac%';
```

Prefix searches can often be more index-friendly than leading-wildcard searches, depending on the database and index.

---

# 62. Why `%keyword%` Can Be Expensive

Query:

```sql
WHERE name LIKE '%phone%'
```

starts with a wildcard.

A normal B-tree index generally cannot efficiently use the beginning of the string for this type of contains search.

For large-scale search requirements, consider:

```text
Full-text search
Search engine
Specialized indexing
```

depending on the use case.

---

# 63. Pagination Problem

Offset pagination:

```sql
SELECT *
FROM products
ORDER BY id
LIMIT 20 OFFSET 100000;
```

may become expensive for large offsets because the database may need to walk through many rows before returning the requested page.

---

# 64. Keyset Pagination Preview

Instead of:

```text
OFFSET 100000
```

use the last seen ID:

```sql
SELECT *
FROM products
WHERE id > 100000
ORDER BY id
LIMIT 20;
```

This is often called:

```text
Keyset pagination
Cursor pagination
```

We will cover it in more detail later.

---

# 65. SELECT with Expressions

SQL can calculate values:

```sql
SELECT
    name,
    price,
    price * 1.18 AS price_with_tax
FROM products;
```

This calculates the expression for each returned row.

---

# 66. String Concatenation in MySQL

MySQL supports:

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM users;
```

Result:

```text
Sudhir Sahoo
```

---

# 67. Numeric Expressions

```sql
SELECT
    name,
    price,
    price * stock AS inventory_value
FROM products;
```

This calculates:

```text
price × stock
```

---

# 68. Date Functions

MySQL provides functions such as:

```text
CURRENT_DATE()
CURRENT_TIMESTAMP()
YEAR()
MONTH()
DAY()
```

Use date functions carefully in filtering conditions, especially when indexed columns are involved.

---

# 69. DISTINCT vs GROUP BY Preview

These can sometimes produce similar results:

```sql
SELECT DISTINCT category
FROM products;
```

and:

```sql
SELECT category
FROM products
GROUP BY category;
```

But they communicate different intent.

Use:

```text
DISTINCT
```

when you want unique result rows.

Use:

```text
GROUP BY
```

when you are forming groups, usually for aggregation.

---

# 70. Common Interview Query #1

**Find products costing more than 50,000.**

```sql
SELECT *
FROM products
WHERE price > 50000;
```

---

# 71. Common Interview Query #2

**Find products between 10,000 and 50,000.**

```sql
SELECT *
FROM products
WHERE price BETWEEN 10000 AND 50000;
```

Remember:

```text
BETWEEN is inclusive.
```

---

# 72. Common Interview Query #3

**Find users whose name starts with S.**

```sql
SELECT *
FROM users
WHERE name LIKE 'S%';
```

---

# 73. Common Interview Query #4

**Find the top 3 highest-paid employees.**

```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

---

# 74. Common Interview Query #5

**Find employees whose salary is between 50,000 and 100,000.**

```sql
SELECT *
FROM employees
WHERE salary BETWEEN 50000 AND 100000;
```

---

# 75. Common Interview Query #6

**Find users who belong to specific cities.**

```sql
SELECT *
FROM users
WHERE city IN (
    'Bangalore',
    'Mumbai',
    'Delhi'
);
```

---

# 76. Common Interview Query #7

**Find users without a phone number.**

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

---

# 77. Common Interview Query #8

**Find the latest 10 orders.**

```sql
SELECT *
FROM orders
ORDER BY created_at DESC, id DESC
LIMIT 10;
```

---

# 78. Common Interview Query #9

**Find distinct product categories.**

```sql
SELECT DISTINCT category
FROM products;
```

---

# 79. Common Interview Query #10

**Find products with stock available and price above 20,000.**

```sql
SELECT id, name, price
FROM products
WHERE stock > 0
  AND price > 20000;
```

---

# 80. Interview: WHERE vs HAVING

> `WHERE` filters individual rows before grouping. `HAVING` filters groups after `GROUP BY`. For example, I would use `WHERE salary > 50,000` to filter employees and `HAVING COUNT(*) > 5` to filter groups with more than five records.

---

# 81. Interview: WHERE vs ON

> `ON` defines how rows are matched in a join, while `WHERE` filters the resulting rows. With outer joins, moving a condition between `ON` and `WHERE` can change the result, so the placement matters.

---

# 82. Interview: BETWEEN Inclusive or Exclusive?

> `BETWEEN` is inclusive of both boundaries. For date-time ranges, I often prefer a half-open range such as `>= start AND < nextStart` because it avoids end-of-day precision problems.

---

# 83. Interview: IN vs OR?

> `IN` is a concise way to check whether a value belongs to a list and is usually easier to read than multiple equality conditions joined by OR. The optimizer can often handle both efficiently, but the actual plan should be checked for performance-sensitive queries.

---

# 84. Interview: Why Avoid SELECT *?

> I prefer selecting only the columns I need because it reduces unnecessary data transfer, makes the contract clearer and avoids accidentally pulling large or newly added columns. It can also make covering-index strategies easier in some cases.

---

# 85. Interview: What Is NULL?

> NULL represents missing or unknown data. It isn't equal to zero or an empty string, and comparisons such as `column = NULL` don't work as expected. I use `IS NULL` or `IS NOT NULL`.

---

# 86. Interview: Why Can LIKE Be Slow?

> A pattern such as `%phone%` starts with a wildcard, so a normal B-tree index generally cannot efficiently use the beginning of the string. For large search workloads, I would consider full-text search or a dedicated search solution.

---

# 87. Interview: How Do You Implement Pagination?

> For small datasets, offset pagination using `LIMIT` and `OFFSET` is straightforward. For large or frequently changing datasets, I prefer keyset or cursor pagination because it avoids large offsets and can provide more stable traversal.

---

# 88. Practical Query Pattern

A common backend query looks like:

```sql
SELECT
    p.id,
    p.name,
    p.price
FROM products p
WHERE p.category = 'Laptop'
  AND p.price >= 30000
  AND p.stock > 0
ORDER BY p.price DESC, p.id DESC
LIMIT 20;
```

Think about it as:

```text
SELECT  → what do I need?
FROM    → where is the data?
WHERE   → which rows?
ORDER   → in what order?
LIMIT   → how many?
```

---

# 89. Query Writing Checklist

Before running a query ask:

```text
□ Which table(s) do I need?
□ Which columns do I need?
□ What rows should match?
□ Could NULL affect the condition?
□ Do I need DISTINCT?
□ Do I need sorting?
□ Is the sort deterministic?
□ Do I need pagination?
□ Could LIMIT/OFFSET become expensive?
□ Is the filter index-friendly?
□ Am I accidentally updating/deleting everything?
```

---

# 90. Final Mental Model

For basic SQL querying:

```text
SELECT
  ↓
FROM
  ↓
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

For filtering:

```text
=
<>
>
<
>=
<=
BETWEEN
IN
LIKE
IS NULL
IS NOT NULL
AND
OR
NOT
```

For a Java backend developer:

```text
API request
    ↓
Controller
    ↓
Service
    ↓
Repository
    ↓
SQL query
    ↓
Database
    ↓
Filtered/sorted result
```

> **Strong SQL starts with being precise about which rows you want, which columns you need, and how the result should be ordered. Once this becomes natural, joins and aggregation become much easier.**
