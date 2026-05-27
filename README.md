# SQL-Best-Practices-Guide
This repository highlights essential SQL skills I utilize as a data analyst. Key topics include standardizing syntax and documentation for seamless team collabor,ation, 
as well as optimizing complex queries to improve execution times.

Query Logic: Understanding SQL to Improve Execution Times


Writing queries that simply return the right answer is only half the battle. As datasets scale into the millions of rows, how you structure your logic directly dictates whether a BI dashboard updates in two seconds or times out after ten minutes. Understanding how the database engine interprets your code is key to writing high-performance SQL.

CTEs vs. Subqueries
Advocate for using Common Table Expressions (CTEs) using the WITH clause instead of nested subqueries. Structuring logic sequentially through CTEs prevents the "spaghetti code" effect common with deeply nested queries, as they read logically from top to bottom. This modular approach also allows you to easily isolate, test, and debug intermediate steps to verify data accuracy before generating the final output.

Incorrect (Nested subqueries must be read inside-out)
Correct (CTEs read logically from top to bottom)

Window Functions
Functions like ROW_NUMBER(), RANK(), or SUM() OVER() can solve complex ranking or running-total problems much more cleanly than complicated self-joins. Utilizing window functions minimizes the need for computationally expensive temporary tables or grouping workarounds. They allow you to retain row-level details while simultaneously performing complex aggregations and chronological rankings.

Incorrect (Using a self-join to calculate a running total)

Correct (Using a Window Function for a massive performance boost)

The Danger of SELECT ***
Selecting everything from a table wastes critical memory and network resources, which is why you should only explicitly call the columns you need. Pulling unnecessary columns increases the payload size, which can severely bottleneck network performance and slow down automated ETL pipelines. Explicitly defining your columns also future-proofs the query against catastrophic failure if the underlying table schema changes unexpectedly.

Incorrect (Wastes memory on unused columns)
Correct (Only pulls the exact data required for the report)

SARGable Queries (Search Argument Able)
Wrapping a column in a function inside a WHERE clause strips the database engine of its ability to use its indexes, forcing it to evaluate every single row in a full table scan. By isolating the column and moving the calculation to the other side of the operator, you make the query "SARGable." This allows the database to instantly locate the exact rows needed, unlocking massive performance gains on large datasets.

Incorrect (Forces a full table scan because it alters the column before comparing)
Correct (Allows the database engine to utilize the date index)

Filtering Early
It is crucial to reduce the dataset size as early as possible using WHERE clauses before applying heavy JOINs or GROUP BY aggregations. Applying restrictive date and category filters in a CTE or staging step minimizes the memory footprint required for subsequent operations. This ensures that the database engine is only spending processing power to join and group the exact records necessary for the final report.

Incorrect (Joins massive tables together first, then filters the results)
Correct (Filters the tables down in CTEs before joining and grouping)


Defensive Coding: Validating Data to Pull from the ‘Source of Truth’


Real-world business data is rarely perfectly clean. A strong data analyst anticipates messy inputs and writes queries that handle unexpected data gracefully. Defensive coding ensures that your queries act as a reliable "Source of Truth," preventing downstream errors in BI tools and guaranteeing that stakeholders are looking at accurate, deduplicated metrics.

Taming NULL Values
Unexpected NULL values can easily break arithmetic calculations, skew aggregations, or cause blank visuals in dashboards. Utilize functions like COALESCE() or ISNULL() to set explicit default values (like converting a missing discount amount to 0). Handling these gaps proactively ensures that financial totals and critical metrics remain accurate rather than failing silently when data is missing.

Incorrect (If discount is NULL, the entire calculation becomes NULL)
Correct (Defaults NULLs to 0 to preserve the calculation)

Deduplication Strategies
Joining a primary table to a one-to-many table without proper filtering can accidentally duplicate rows, artificially inflating revenue numbers and throwing off KPIs. While DISTINCT can sometimes patch this, using ROW_NUMBER() is a far more precise strategy. Relying on window functions to partition and order data guarantees that you are pulling the exact, most recent record required for your analysis.

Incorrect (Blindly joining causes duplicate revenue if the user has multiple updates)
Correct (Uses a CTE and ROW_NUMBER to deliberately grab only the most recent status)

Standardizing Formats
It is incredibly important to cast and standardize text fields (e.g., using TRIM(), UPPER(), or LOWER()) so they group correctly in final reports. Inconsistent casing or accidental trailing spaces created by manual data entry can cause aggregations to split what should be a single category into multiple distinct rows. By aggressively standardizing formats in your base queries, you guarantee uniform, reliable reporting across the entire organization.

Incorrect (Will result in multiple distinct rows for "New York", "new york", and "NEW YORK ")
Correct (Standardizes casing and removes hidden trailing spaces before grouping)
Database Structure: Communicating for Structural Optimization

Business data needs are ever-growing and evolving, meaning today's unused fields often become critical for tomorrow's new projects. Keeping an open line of communication with the database administration team ensures that your reporting needs align with the physical architecture of the data warehouse. Proactively discussing structural optimization minimizes friction and prevents system bottlenecks as data volume scales.

Choosing the Right Data Types

Using a VARCHAR(255) when a VARCHAR(10) will suffice, or utilizing a BIGINT instead of an standard INT, bloats the database and drastically slows down processing. Over-allocating memory for data types drastically increases the storage footprint and degrades query performance during large data scans. Ensuring that data types perfectly match their actual contents preserves index efficiency, saves disk space, and speeds up overall processing times.

Incorrect (Wasting memory on generic, oversized data types)
Correct (Using precise data types to save storage and speed up memory allocation)

Understanding Indexes

Indexes work much like the index at the back of a textbook, allowing the database engine to locate and retrieve specific rows instantly rather than scanning millions of unrelated records. It is vital to understand the difference between a clustered index (how the data is physically sorted on the disk, usually the Primary Key) and a non-clustered index (a separate lookup table mapping to the physical data). Knowing which columns are heavily filtered and ensuring they are properly indexed is the difference between a query running in two seconds versus two hours.

Incorrect (Forcing a full table scan by frequently filtering on an unindexed column):
Correct (Communicating with the DBA to create a non-clustered index for faster lookups)

Execution Plans (EXPLAIN)

The ultimate best practice for structural optimization is reading the database's execution plan to identify hidden bottlenecks like full table scans or expensive nested loops. The execution plan acts as a diagnostic map, revealing exactly where the database engine is spending its computational resources and memory. Learning to read and interpret these plans allows you to rewrite inefficient logic, validate your index usage, and confidently deploy high-performance code to production.
