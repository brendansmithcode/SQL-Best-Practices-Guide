# SQL-Best-Practices-Guide
This repository highlights essential SQL skills I utilize as a data analyst. Key topics include standardizing syntax and documentation for seamless team collabor,ation, 
as well as optimizing complex queries to improve execution times.

## SQL Syntax: Writing for Readability and Interoperability

SQL is a highly flexible language that will often execute successfully regardless of how poorly the query is formatted. Here are two examples of queries that generate the exact same result but utilize drastically different syntax:

Utilizing different syntax standards between members of the same team can cause issues when reviewing new projects for release, troubleshooting errors that prevent the
query from running, or when cross-training tasks between team members.

### Capitalization Standards

SQL keywords (such as SELECT, FROM, WHERE, etc.) should always be capitalized to visually differentiate SQL logic from table and column names. This rule also applies to aggregate functions like SUM() and operators like DISTINCT.

```sql
-- Incorrect (Hard to read)
select customer_id, count(order_id) as total_orders from sales_data where region = 'south' group by customer_id;

-- Corrected Version (Keywords and functions stand out)
SELECT customer_id, SUM(total)
FROM orders
GROUP BY customer_id;
```

### Indentation and Spacing

Consistent use of indentation and spacing is critical for identifying errors and assisting other team members in reviewing your code. The following example executes correctly, but the lack of formatting makes it incredibly difficult to spot the keywords and follow the logic.

(Example of Corrected Indentation)
```sql
-- Incorrect (The "Wall of Text"):
SELECT id, CASE WHEN status='active' THEN 1 ELSE 0 END as is_active, total_spent FROM users JOIN orders ON users.id=orders.user_id WHERE orders.total > 100;

-- Correct (Logical flow is immediately obvious)
SELECT 
    id, 
    CASE 
        WHEN status = 'active' THEN 1 
        ELSE 0 
    END AS is_active, 
    total_spent 
FROM users 
JOIN orders 
    ON users.id = orders.user_id 
WHERE orders.total > 100;
```

This lack of readability becomes especially pronounced when dealing with nested conditions or multiple JOIN clauses. By explicitly aligning your ON statements and CASE logic (see the corrected example above), you prevent structural errors and make it immediately obvious how different tables relate to one another.

## Renaming Fields / Managing Aliases

When working across multiple tables or databases, it can be easy to get confused by overlapping column names (such as multiple tables having an id or status field). To mitigate this, you can explicitly define the field using the table_name.column_name format. However, a cleaner approach is to assign meaningful table and column aliases using the AS keyword to prevent ambiguity.

(Example of Corrected Alias Code)

```sql
-- Ambiguous
SELECT a.id, b.date, c.name 
FROM users a 
JOIN orders b ON a.id = b.user_id 
JOIN products c ON b.product_id = c.id;

-- Correct (Contextual abbreviations)
SELECT u.user_id, o.order_date, p.product_name 
FROM users u 
JOIN orders o ON u.user_id = o.user_id 
JOIN products p ON o.product_id = p.product_id;
```

When utilizing column aliases, you may occasionally encounter an error where SQL fails to recognize your newly created field name. This happens because of the logical Order of Execution in SQL. If a column is renamed in the SELECT statement, SQL will not be able to identify that new alias inside the WHERE clause, because the database evaluates the WHERE filter before it ever processes the SELECT statement.

To avoid this, it is helpful to memorize the order in which a database engine reads a query:

1. FROM (and JOINs)

2. WHERE

3. GROUP BY

4. HAVING

5. SELECT

6. ORDER BY

## Documentation: Sharing the "Why" Over the "What"

Important code never stays stagnant. A SQL query will often undergo several iterations and changes as the scope of a project grows or business needs evolve. It is essential to communicate effectively within your code so the underlying logic remains understandable to other team members—or to your future self.

Header blocks are highly recommended when creating longer or more complex queries. The header functions as a brief overview detailing the script's purpose, the author, and the date the logic was last updated. Here is a strong example:

(Example SQL code here)

```sql
/* ========================================================================
   Author: [Your Name]
   Date: 2024-05-27
   Purpose: Extracts monthly recurring revenue (MRR) for active accounts.
   Dependencies: Rebuilds every morning at 6:00 AM for the Executive Dashboard.
   ======================================================================== */
```

You should also utilize section headers above large blocks of code, as well as inline comments, to provide context for specific statements.

(Header and inline SQL comment code here)

```sql
WHERE account_type_id != 99 -- Filters out account type 99#
 
WHERE account_type_id != 99 -- Excludes internal QA/Testing accounts from revenue totals
```

It is critical to emphasize the purpose and goal of a section of code rather than simply explaining how the syntax functions. When returning to update a query, it is far more helpful to understand the why rather than the what. Comments shouldn't just translate the SQL into English; instead, they should explain the business logic driving the code.

Finally, commenting is an excellent tool for debugging complex queries. Block comments (/* ... */) are incredibly useful to test data integrity without having to rewrite or delete complex logic. Just be sure not to save or commit any queries with active logic left in block comments to avoid errors and future confusion.
      
## Query Logic: Understanding SQL to Improve Execution Times

Writing queries that simply return the right answer is only half the battle. As datasets scale into the millions of rows, how you structure your logic directly dictates whether a BI dashboard updates in two seconds or times out after ten minutes. Understanding how the database engine interprets your code is key to writing high-performance SQL.

### CTEs vs. Subqueries
Advocate for using Common Table Expressions (CTEs) using the WITH clause instead of nested subqueries. Structuring logic sequentially through CTEs prevents the "spaghetti code" effect common with deeply nested queries, as they read logically from top to bottom. This modular approach also allows you to easily isolate, test, and debug intermediate steps to verify data accuracy before generating the final output.

```sql
-- Incorrect (Nested subqueries must be read inside-out)
SELECT * FROM (
    SELECT customer_id, SUM(total) as revenue FROM (
        SELECT * FROM orders WHERE status = 'shipped'
    ) shipped_orders GROUP BY customer_id
) revenue_summary WHERE revenue > 1000;

-- Correct (CTEs read logically from top to bottom)
WITH ShippedOrders AS (
    SELECT * FROM orders WHERE status = 'shipped'
),
RevenueSummary AS (
    SELECT customer_id, SUM(total) AS revenue 
    FROM ShippedOrders 
    GROUP BY customer_id
)
SELECT * FROM RevenueSummary WHERE revenue > 1000;
```

### Window Functions
Functions like ROW_NUMBER(), RANK(), or SUM() OVER() can solve complex ranking or running-total problems much more cleanly than complicated self-joins. Utilizing window functions minimizes the need for computationally expensive temporary tables or grouping workarounds. They allow you to retain row-level details while simultaneously performing complex aggregations and chronological rankings.

Incorrect (Using a self-join to calculate a running total)
```sql
-- Incorrect (Wastes memory on unused columns)
SELECT * FROM customer_data;

-- Correct (Using a Window Function for a massive performance boost)
SELECT customer_id, first_name, email, lifetime_value 
FROM customer_data;
```

### The Danger of SELECT ***
Selecting everything from a table wastes critical memory and network resources, which is why you should only explicitly call the columns you need. Pulling unnecessary columns increases the payload size, which can severely bottleneck network performance and slow down automated ETL pipelines. Explicitly defining your columns also future-proofs the query against catastrophic failure if the underlying table schema changes unexpectedly.

```sql
-- Incorrect (Wastes memory on unused columns)
SELECT * FROM customer_data;

-- Correct (Only pulls the exact data required for the report)
SELECT customer_id, first_name, email, lifetime_value 
FROM customer_data;
```

### SARGable Queries (Search Argument Able)

Wrapping a column in a function inside a WHERE clause strips the database engine of its ability to use its indexes, forcing it to evaluate every single row in a full table scan. By isolating the column and moving the calculation to the other side of the operator, you make the query "SARGable." This allows the database to instantly locate the exact rows needed, unlocking massive performance gains on large datasets.

```sql
-- Incorrect (Forces a full table scan because it alters the column before comparing)
SELECT order_id, order_date 
FROM sales 
WHERE YEAR(order_date) = 2023;

-- Correct (Allows the database engine to utilize the date index)
SELECT order_id, order_date 
FROM sales 
WHERE order_date >= '2023-01-01' AND order_date < '2024-01-01';
```

### Filtering Early

It is crucial to reduce the dataset size as early as possible using WHERE clauses before applying heavy JOINs or GROUP BY aggregations. Applying restrictive date and category filters in a CTE or staging step minimizes the memory footprint required for subsequent operations. This ensures that the database engine is only spending processing power to join and group the exact records necessary for the final report.

```sql
-- Incorrect (Joins massive tables together first, then filters the results)
SELECT u.customer_name, SUM(s.amount)
FROM massive_sales_table s
JOIN massive_users_table u ON s.user_id = u.user_id
WHERE s.region = 'North America'
GROUP BY u.customer_name;

-- Correct (Filters the tables down in CTEs before joining and grouping)
WITH NASales AS (
    SELECT user_id, amount 
    FROM massive_sales_table 
    WHERE region = 'North America'
)
SELECT u.customer_name, SUM(s.amount)
FROM NASales s
JOIN massive_users_table u ON s.user_id = u.user_id
GROUP BY u.customer_name;
```

## Defensive Coding: Validating Data to Pull from the ‘Source of Truth’

Real-world business data is rarely perfectly clean. A strong data analyst anticipates messy inputs and writes queries that handle unexpected data gracefully. Defensive coding ensures that your queries act as a reliable "Source of Truth," preventing downstream errors in BI tools and guaranteeing that stakeholders are looking at accurate, deduplicated metrics.

### Taming NULL Values
Unexpected NULL values can easily break arithmetic calculations, skew aggregations, or cause blank visuals in dashboards. Utilize functions like COALESCE() or ISNULL() to set explicit default values (like converting a missing discount amount to 0). Handling these gaps proactively ensures that financial totals and critical metrics remain accurate rather than failing silently when data is missing.

```sql
-- Incorrect (If discount is NULL, the entire calculation becomes NULL)
SELECT 
    order_id, 
    base_price - discount AS final_price 
FROM product_sales;

-- Correct (Defaults NULLs to 0 to preserve the calculation)
SELECT 
    order_id, 
    base_price - COALESCE(discount, 0) AS final_price 
FROM product_sales;
```

### Deduplication Strategies

Joining a primary table to a one-to-many table without proper filtering can accidentally duplicate rows, artificially inflating revenue numbers and throwing off KPIs. While DISTINCT can sometimes patch this, using ROW_NUMBER() is a far more precise strategy. Relying on window functions to partition and order data guarantees that you are pulling the exact, most recent record required for your analysis.

```sql
-- Incorrect (Blindly joining causes duplicate revenue if the user has multiple updates)
SELECT 
    u.user_id, 
    u.lifetime_revenue, 
    s.account_status 
FROM users u
JOIN account_status_log s ON u.user_id = s.user_id;

-- Correct (Uses a CTE and ROW_NUMBER to deliberately grab only the most recent status)
WITH LatestStatus AS (
    SELECT 
        user_id, 
        account_status,
        ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY status_date DESC) as row_num
    FROM account_status_log
)
SELECT 
    u.user_id, 
    u.lifetime_revenue, 
    ls.account_status 
FROM users u
JOIN LatestStatus ls ON u.user_id = ls.user_id
WHERE ls.row_num = 1;
```

### Standardizing Formats

It is incredibly important to cast and standardize text fields (e.g., using TRIM(), UPPER(), or LOWER()) so they group correctly in final reports. Inconsistent casing or accidental trailing spaces created by manual data entry can cause aggregations to split what should be a single category into multiple distinct rows. By aggressively standardizing formats in your base queries, you guarantee uniform, reliable reporting across the entire organization.

```sql
-- Incorrect (Will result in multiple distinct rows for "New York", "new york", and "NEW YORK ")
SELECT 
    state, 
    COUNT(customer_id) AS total_customers 
FROM users 
GROUP BY state;

-- Correct (Standardizes casing and removes hidden trailing spaces before grouping)
SELECT 
    TRIM(UPPER(state)) AS clean_state, 
    COUNT(customer_id) AS total_customers 
FROM users 
GROUP BY TRIM(UPPER(state));
```
