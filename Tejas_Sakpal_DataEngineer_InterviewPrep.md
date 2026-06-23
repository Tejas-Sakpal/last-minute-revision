# DATA ENGINEER — Last-Minute Interview Revision Guide

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white) ![SQL](https://img.shields.io/badge/SQL-4479A1?style=flat&logo=postgresql&logoColor=white) ![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=flat&logo=apachespark&logoColor=white) ![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=flat&logo=databricks&logoColor=white) ![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat&logo=microsoftazure&logoColor=white) ![Git](https://img.shields.io/badge/Git-F05032?style=flat&logo=git&logoColor=white) ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)

**Prepared for:** Tejas Sakpal&nbsp;&nbsp;|&nbsp;&nbsp;**Stack:** Python · SQL · PySpark · Azure Databricks · ADF · ADLS&nbsp;&nbsp;|&nbsp;&nbsp;**Experience:** 4 years&nbsp;&nbsp;|&nbsp;&nbsp;**Date:** June 2026

## How to use this guide

- Sections 1-4 cover theory + hands-on questions for Python, SQL, PySpark, and Azure Databricks/Cloud - the core technical stack on your resume.
- Section 5 maps directly to YOUR resume bullets (Accenture, SLO/AdvaRisk, Innoplexus) - practice these out loud, with numbers.
- Section 6 covers supporting tools (Docker, Git, Gen AI), your certifications, and general/HR questions.
- Read every concept for the explanation, but rehearse the project (Section 5) and coding questions (Sections 1-3) hands-on - those decide the interview.
- Q&A blocks are collapsed by default (click "Show Q&A" to expand) so you can scan the guide quickly.

## Table of Contents

- [1. PYTHON](#1-python)
  - [1.1 Core Concepts to Revise](#11-core-concepts-to-revise)
  - [1.2 Theory Q&A](#12-theory-qa)
  - [1.3 Pandas Essentials](#13-pandas-essentials)
  - [1.4 Coding / Hands-on Questions](#14-coding-hands-on-questions)
- [2. SQL](#2-sql)
  - [2.1 Core Concepts to Revise](#21-core-concepts-to-revise)
  - [2.2 Theory Q&A](#22-theory-qa)
  - [2.3 Practice Query Patterns](#23-practice-query-patterns)
- [3. PYSPARK](#3-pyspark)
  - [3.1 Core Concepts to Revise](#31-core-concepts-to-revise)
  - [3.2 Theory Q&A](#32-theory-qa)
  - [3.3 Coding Questions](#33-coding-questions)
- [4. AZURE DATABRICKS, ADF, ADLS & CLOUD](#4-azure-databricks-adf-adls-cloud)
  - [4.1 Core Concepts to Revise](#41-core-concepts-to-revise)
  - [4.2 Theory Q&A](#42-theory-qa)
  - [4.3 Useful Commands / Snippets](#43-useful-commands-snippets)
- [5. PROJECT / RESUME-BASED QUESTIONS](#5-project-resume-based-questions)
  - [5.1 Accenture — Packaged App Development Analyst (Jan 2025–Present)](#51-accenture-packaged-app-development-analyst-jan-2025present)
  - [5.2 SLO Technologies (AdvaRisk) — Jr. Data Engineer (Sep 2023–Jan 2025)](#52-slo-technologies-advarisk-jr-data-engineer-sep-2023jan-2025)
  - [5.3 Innoplexus Consulting — Associate Software Engineer (Jul 2022–Sep 2023)](#53-innoplexus-consulting-associate-software-engineer-jul-2022sep-2023)
  - [5.4 General Project / Behavioral Follow-ups (be ready regardless of which project you discuss)](#54-general-project-behavioral-follow-ups-be-ready-regardless-of-which-project-you-discuss)
- [6. OTHER LIKELY TOPICS (Resume-Driven)](#6-other-likely-topics-resume-driven)
  - [6.1 Docker](#61-docker)
  - [6.2 Git](#62-git)
  - [6.3 Gen AI (listed on resume — be ready for at least a high-level conversation)](#63-gen-ai-listed-on-resume-be-ready-for-at-least-a-high-level-conversation)
  - [6.4 Certifications — be ready to defend these in conversation](#64-certifications-be-ready-to-defend-these-in-conversation)
  - [6.5 Elasticsearch (from Innoplexus experience)](#65-elasticsearch-from-innoplexus-experience)
  - [6.6 General / HR Round Prep](#66-general-hr-round-prep)
- [7. 5-Minute-Before-the-Interview Cheat Sheet](#7-5-minute-before-the-interview-cheat-sheet)

---

# 1. PYTHON

Focus: core language fundamentals, OOP, file/data handling, Pandas — all framed for data engineering use (ETL scripts, automation, API integration).

## 1.1 Core Concepts to Revise

- Data types & mutability — list and dict are mutable (changeable in place); tuple and str are immutable (any "change" creates a new object). Immutable types like tuple/frozenset are hashable and can be used as dict keys or set members; mutable types cannot. This matters in pipeline code because mutating a list inside a function silently changes the caller's copy too, while immutable types are always safe to pass around.
- Memory model — every variable is a reference (a name bound to an object), not the object itself. a = b makes both names point to the same object, so mutating through one is visible through the other. `is` checks whether two references point to the same object in memory; `==` checks whether the underlying values are equal — two different list objects with identical contents are `==` but not `is`.
- Functions — *args collects extra positional arguments into a tuple and **kwargs collects extra keyword arguments into a dict, which is how you write flexible wrapper functions (e.g., decorators that work on any function signature). Functions are first-class objects — they can be assigned to variables, passed as arguments, and returned from other functions, which is the foundation for decorators and functional patterns like map()/filter().
- Comprehensions — list/dict/set comprehensions ([x for x in y], {k:v for ...}) build the entire collection in memory immediately and are usually faster and more readable than an equivalent for-loop. A generator expression (x for x in y) uses the same syntax with parentheses but produces values lazily, one at a time — critical when processing data too large to fit in memory.
- Iterators & Generators — a generator function (one using yield) pauses and resumes execution, producing one value at a time instead of building a full list, keeping memory usage flat regardless of input size. This is the standard pattern for reading huge files or paginating through an API inside a pipeline.
- Decorators — a function that takes another function and returns a wrapped version of it, adding behavior (logging, timing, retries) without touching the original code. @staticmethod marks a method needing no access to the instance/class; @classmethod gives access to the class (cls) instead of the instance, often used for alternate constructors; @property lets a method be accessed like a plain attribute, typically for computed/read-only fields.
- OOP — classes bundle data (attributes) and behavior (methods); inheritance lets a subclass reuse/extend a parent's behavior; polymorphism means different classes can be used interchangeably if they implement the same methods; encapsulation hides internal state behind a controlled interface. Abstract base classes (abc module) define a required interface subclasses must implement — useful for enforcing a consistent structure across multiple data-source connector classes. Dunder methods like __init__ (constructor), __str__/__repr__ (string representation), and __eq__ (equality) let custom objects integrate naturally with Python's built-in syntax.
- Exception handling — try/except catches and handles errors; else runs only if no exception occurred; finally always runs, which is why it's used for guaranteed cleanup like closing a file or DB connection. Custom exceptions (subclassing Exception) let you raise domain-specific errors — e.g., SchemaValidationError — so calling code can react to your pipeline's specific failure modes instead of generic exceptions.
- Context managers — objects implementing __enter__/__exit__, used with the `with` statement to guarantee setup and teardown even if an exception is raised inside the block. This is exactly the pattern used for file handles, DB connections, and Spark/DB sessions — the resource always gets closed, even on failure.
- Modules used in DE — pandas/NumPy for in-memory data manipulation, os/sys for environment and path handling, json/re for parsing, datetime for time logic, logging for structured pipeline logs, requests for calling APIs, sqlalchemy/pyodbc for database connectivity, and multiprocessing/threading/concurrent.futures for parallelizing I/O- or CPU-bound work.
- GIL (Global Interpreter Lock) — allows only one thread to execute Python bytecode at a time, so multithreading doesn't give true CPU parallelism for pure-Python code, though it still helps for I/O-bound work since the GIL is released during I/O waits. For CPU-bound parallelism, use multiprocessing instead, which runs separate processes with separate memory and bypasses the GIL entirely.
- Shallow copy vs deep copy — copy.copy() makes a new outer object but reuses references to nested objects inside it, so mutating a nested list inside a shallow copy also changes the original. copy.deepcopy() recursively copies every nested object, producing a fully independent structure — important when you need to manipulate a data structure without side effects on the original.
- Python memory management — CPython uses reference counting (an object is freed once nothing references it) plus a cyclic garbage collector that cleans up reference cycles a simple counter can't catch (e.g., two objects pointing at each other). You rarely manage this manually, but it's why large structures should be dereferenced (e.g., del df) when no longer needed in a long-running process.

## 1.2 Theory Q&A

<details>
<summary><strong>Show Q&amp;A (13 questions)</strong></summary>

| Question | Answer |
|---|---|
| What is the difference between a list and a tuple? | • List is mutable, tuple is immutable.<br>• Tuples are slightly faster and hashable (can be dict keys); lists cannot be.<br>• Use tuples for fixed records (e.g., a row), lists for collections that change. |
| What is the difference between deepcopy and shallow copy? | • Shallow copy (copy.copy) creates a new object but nested objects still reference the same memory.<br>• Deep copy (copy.deepcopy) recursively copies nested objects too, fully independent. |
| Explain Python's GIL. How do you achieve true parallelism? | • GIL allows only one thread to execute Python bytecode at a time, even on multi-core CPUs.<br>• Threading helps with I/O-bound tasks (API calls, file I/O) since GIL is released during I/O waits.<br>• For CPU-bound tasks (heavy transformations), use multiprocessing (separate processes, separate memory) or offload to PySpark/Pandas (which use C extensions that release the GIL). |
| What are *args and **kwargs? | • *args collects extra positional arguments into a tuple.<br>• **kwargs collects extra keyword arguments into a dict.<br>• Used for writing flexible/generic functions, e.g., wrapper functions in decorators. |
| What is a generator and why is it useful in data engineering? | • A function using yield that produces values lazily, one at a time, without storing the whole sequence in memory.<br>• Critical for processing large files/streams (e.g., reading a multi-GB CSV line by line) without exhausting memory.<br>• Example: reading a huge log file or paginating through an API. |
| What is a decorator? Give a practical DE example. | • A decorator wraps a function to add behavior (logging, timing, retry, auth) without changing its code.<br>• Practical example: a @retry decorator around an API call to a source system, or @timing_logger around an ETL function to log execution duration. |
| Difference between @staticmethod, @classmethod, and instance methods? | • Instance method: takes self, operates on instance data.<br>• @classmethod: takes cls, operates on the class itself (e.g., alternate constructors).<br>• @staticmethod: takes neither; just a utility function grouped inside the class. |
| How does exception handling work in Python? How do you create a custom exception? | • try/except catches errors; else runs if no exception; finally always runs (cleanup, closing connections).<br>• Custom exception: class MyError(Exception): pass — used to raise domain-specific errors in pipelines (e.g., SchemaValidationError). |
| What is the difference between is and ==? | • == checks value equality.<br>• is checks identity (same object in memory). |
| What are Python context managers and where would you use one in ETL code? | • Objects implementing __enter__/__exit__, used with 'with' to guarantee setup/cleanup (e.g., closing a DB connection or file handle even if an exception occurs).<br>• Example: with open('file.csv') as f: ... or a custom context manager wrapping a DB session/Spark session. |
| List comprehension vs generator expression — when do you choose which? | • List comprehension [x for x in y] builds the full list in memory immediately.<br>• Generator expression (x for x in y) is lazy — use it when iterating once over large data to save memory. |
| How do you handle missing/null values in Python before loading data? | • pandas: isna(), fillna(), dropna(), interpolate().<br>• Decide strategy based on business context: default value, forward-fill, drop row, or flag for review. |
| What is the difference between Python 'multithreading' and 'multiprocessing'? Which would you use for parallel file processing? | • Multithreading shares memory, limited by GIL — good for I/O bound work (downloading multiple files).<br>• Multiprocessing spawns separate processes with separate memory/interpreters — good for CPU-bound transformation work, bypasses GIL. |

</details>

## 1.3 Pandas Essentials

- read_csv()/read_sql()/read_json() load data into a DataFrame from a file, database query, or JSON; to_csv()/to_sql() write it back out. For files too large to fit in memory at once, the chunksize parameter makes read_csv() return an iterator of smaller DataFrames so you can process a file piece by piece instead of loading it whole.
- merge() is the general-purpose, SQL-style join (inner/left/right/outer) on one or more key columns; concat() simply stacks DataFrames row-wise or column-wise without matching on keys; join() is a convenience method that merges on the index by default. Pick merge() for combining related tables and concat() for appending similarly shaped data.
- groupby().agg() mirrors SQL's GROUP BY plus aggregate functions, letting you compute several different aggregations per group in one call (e.g., {'amount': 'sum', 'date': 'max'}). pivot_table() reshapes long data into a wide, spreadsheet-style summary. apply() runs a custom function row-wise or column-wise (flexible but not vectorized, so slower); map() applies a function element-wise to a single Series; applymap() applies element-wise across an entire DataFrame.
- Vectorized operations (e.g., df['amount'] * 1.1) push computation into NumPy's compiled C code and operate on whole columns at once instead of looping through rows in Python. apply() still loops in Python under the hood, so it's almost always slower — vectorize first, and reach for apply() only when there's no vectorized equivalent.
- duplicated() returns a boolean mask flagging rows that repeat an earlier row (by default across all columns); drop_duplicates() removes those rows. Both accept a subset parameter to check/dedupe on specific columns only — useful for catching records a source system resent more than once.
- Memory optimization matters once a DataFrame gets large: downcasting numeric dtypes (int64 to int32, float64 to float32) when the smaller range is sufficient cuts memory directly, and converting a low-cardinality text column (e.g., a status column with only a few distinct values) to the category dtype stores each value once and uses small integer codes for the rest — often a dramatic memory reduction for that column.

## 1.4 Coding / Hands-on Questions

### Q1. Remove duplicates from a list while preserving order

```
def remove_duplicates(lst):
    seen = set()
    result = []
    for item in lst:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result
```

### Q2. Find duplicate records in a list of dicts (rows) by a key

```
from collections import Counter
def find_duplicate_keys(rows, key):
    counts = Counter(r[key] for r in rows)
    return [k for k, c in counts.items() if c > 1]
```

### Q3. Read a large file line-by-line using a generator (memory efficient)

```
def read_large_file(path):
    with open(path) as f:
        for line in f:
            yield line.strip()

for row in read_large_file('big_data.csv'):
    process(row)  # process one row at a time, O(1) memory
```

### Q4. Write a decorator to log execution time of a function

```
import time, functools
def timing_logger(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f'{func.__name__} took {time.time()-start:.2f}s')
        return result
    return wrapper

@timing_logger
def load_data(path):
    ...
```

### Q5. Flatten a nested list/dictionary (common for semi-structured JSON sources)

```
def flatten_dict(d, parent_key='', sep='_'):
    items = {}
    for k, v in d.items():
        new_key = f'{parent_key}{sep}{k}' if parent_key else k
        if isinstance(v, dict):
            items.update(flatten_dict(v, new_key, sep))
        else:
            items[new_key] = v
    return items
```

### Q6. Find the second largest number in a list without using sort()

```
def second_largest(nums):
    first = second = float('-inf')
    for n in nums:
        if n > first:
            first, second = n, first
        elif first > n > second:
            second = n
    return second
```

### Q7. Word frequency count from a text/log file

```
from collections import Counter
def word_freq(path):
    with open(path) as f:
        words = f.read().split()
    return Counter(words).most_common(10)
```

### Q8. Retry wrapper for a flaky API call (resume-relevant: data ingestion automation)

```
import time
def retry(times=3, delay=2):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f'Attempt {attempt+1} failed: {e}')
                    time.sleep(delay)
            raise Exception('All retries failed')
        return wrapper
    return decorator
```

---

# 2. SQL

Focus: joins, window functions, aggregation, query optimization, and Azure SQL (used at Innoplexus per resume).

## 2.1 Core Concepts to Revise

- Joins — INNER JOIN returns only rows with matches in both tables; LEFT/RIGHT JOIN keep all rows from one side and NULL-fill the other when there's no match; FULL OUTER JOIN keeps unmatched rows from both sides; SELF JOIN joins a table to itself (e.g., comparing employees to their managers within the same table); CROSS JOIN returns the full Cartesian product of both tables, which is rarely intentional and usually signals a missing join condition.
- Aggregations — GROUP BY collapses rows into groups so aggregate functions (SUM, COUNT, AVG, MIN, MAX) can be applied per group. HAVING filters those groups after aggregation (e.g., HAVING COUNT(*) > 1), while WHERE filters individual rows before any grouping happens — that's why an aggregate function can't be referenced inside WHERE.
- Window functions — unlike GROUP BY, window functions (ROW_NUMBER(), RANK(), LAG()/LEAD(), SUM() OVER(...)) compute a value per row while still having access to a related "window" of rows (PARTITION BY groups the window, ORDER BY defines order within it), without collapsing the result set — so row-level detail and the aggregate/rank value sit side by side.
- Subqueries vs CTEs vs temp tables — a subquery is nested inside another query, fine for a single filter but unreadable once nested several levels deep. A CTE (WITH clause) names an intermediate result for the rest of the query, making multi-step logic much easier to follow, and can be recursive. A temp table physically materializes the result, worth doing when it's large, reused multiple times, or needs its own index.
- Set operations — UNION combines result sets and removes duplicates, which requires an implicit sort/dedup step and is therefore more expensive. UNION ALL keeps every row including duplicates and is faster — use it whenever the inputs are already known not to overlap. INTERSECT returns rows common to both queries; EXCEPT (or MINUS) returns rows in the first query but not the second.
- Indexes — a separate data structure (commonly a B-tree) that lets the database find rows without scanning the whole table. A clustered index physically sorts and stores the table's data in index order (only one per table); a non-clustered index is a separate structure pointing back to the actual rows (a table can have several). Indexes speed up reads but slow down writes, since every INSERT/UPDATE/DELETE must also update each index — over-indexing a write-heavy staging table can hurt more than it helps.
- Normalization vs denormalization — normalization (1NF, 2NF, 3NF) organizes data to remove redundancy and update anomalies, good for OLTP systems with frequent small writes. Denormalization deliberately reintroduces redundancy (wider tables, pre-joined columns) to cut the number of joins needed at query time — the standard approach for OLAP/warehouse Gold-layer tables, where read performance matters more than write-side efficiency.
- ACID — the guarantees a transactional database gives so concurrent operations don't corrupt or see inconsistent data (full breakdown in the Q&A below). Isolation levels (Read Committed, Repeatable Read, Serializable, etc.) control exactly how much concurrent transactions can "see" of each other's uncommitted changes, trading consistency against concurrency/performance.
- Constraints — PRIMARY KEY uniquely identifies each row (implicitly NOT NULL + UNIQUE); FOREIGN KEY enforces that a column's values must exist in another table's primary key, preserving referential integrity; UNIQUE prevents duplicate values without making a column the primary key; CHECK enforces a custom rule (e.g., salary > 0); NOT NULL guarantees a column is always populated. These protect data quality at the database level, independent of any application code.
- Query execution order — even though a query is written SELECT...FROM...WHERE...GROUP BY...HAVING...ORDER BY, the engine evaluates it differently: FROM/JOINs build the working row set first, then WHERE filters rows, then GROUP BY forms groups, then HAVING filters groups, then SELECT computes the output columns, then ORDER BY sorts, then LIMIT trims. This is why a SELECT alias can't be referenced inside the same query's WHERE clause — WHERE runs before SELECT.
- Query optimization — read the execution plan to see whether the engine is doing a full table scan (slow) or an index seek (fast). Avoid SELECT * (forces reading every column), filter as early as possible so fewer rows flow into later joins, and watch for wrapping an indexed column in a function in WHERE (e.g., YEAR(date_col) = 2026), which usually prevents the index from being used at all.
- Stored procedures/views/triggers — a view is a saved, reusable SELECT statement that looks like a virtual table; a materialized view actually stores the computed result and needs periodic refreshing, trading staleness for query speed. A stored procedure is precompiled, parameterized SQL logic stored in the database, useful for encapsulating repeated business logic. A trigger automatically fires on an INSERT/UPDATE/DELETE — handy for auditing or cascading updates, but easy to overuse and make data flows hard to trace.
- Slowly Changing Dimensions (SCD) — describes how a dimension table (e.g., customers) handles attribute values that change over time. Type 1 simply overwrites the old value with no history; Type 2 inserts a brand-new row with effective/expiry dates and a current flag, preserving the full history of every change — a very common data-warehouse design question (full example in the Q&A below).

## 2.2 Theory Q&A

<details>
<summary><strong>Show Q&amp;A (11 questions)</strong></summary>

| Question | Answer |
|---|---|
| What's the difference between WHERE and HAVING? | • WHERE filters rows before aggregation (cannot use aggregate functions).<br>• HAVING filters groups after GROUP BY/aggregation. |
| Difference between RANK(), DENSE_RANK(), and ROW_NUMBER()? | • ROW_NUMBER(): unique sequential number, no gaps, ties broken arbitrarily.<br>• RANK(): same rank for ties, but leaves gaps afterward (1,2,2,4).<br>• DENSE_RANK(): same rank for ties, no gaps (1,2,2,3). |
| How would you find the 2nd highest salary per department? | • Use RANK() or DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) in a CTE, then filter rank = 2.<br>• Avoid nested MAX(salary WHERE salary < MAX(salary)) — works but doesn't scale to 'Nth highest'. |
| CTE vs Subquery vs Temp Table — when do you use each? | • CTE: improves readability for multi-step logic, can be recursive, scoped to one query.<br>• Subquery: fine for simple one-off filters, gets unreadable when nested deeply.<br>• Temp table: useful when the intermediate result is reused multiple times or is very large (materialized, can be indexed). |
| What is a Slowly Changing Dimension (SCD), and what's the difference between Type 1 and Type 2? | • SCD handles how a dimension table reacts to changing attribute values over time.<br>• Type 1: overwrite the old value — no history kept (e.g., correcting a typo).<br>• Type 2: insert a new row with effective/expiry dates and a current flag — full history preserved (e.g., tracking a customer's address changes over time). |
| How does indexing improve performance, and when can it hurt? | • Indexes speed up reads (SELECT/WHERE/JOIN/ORDER BY) by avoiding full table scans.<br>• They slow down writes (INSERT/UPDATE/DELETE) since indexes must also be updated — over-indexing a write-heavy staging table is counterproductive. |
| Explain ACID properties. | • Atomicity: transaction fully completes or fully rolls back.<br>• Consistency: data remains valid per constraints before/after the transaction.<br>• Isolation: concurrent transactions don't interfere with each other.<br>• Durability: once committed, changes persist even after a crash. |
| How do you optimize a slow SQL query? | • Check the execution plan for full table scans/missing indexes.<br>• Avoid SELECT *, filter early, push predicates down, avoid functions on indexed columns in WHERE.<br>• Reduce data volume early (filter/partition before joining), use appropriate join order, partition large tables, pre-aggregate where possible. |
| UNION vs UNION ALL? | • UNION removes duplicates (requires an implicit sort/dedup — more expensive).<br>• UNION ALL keeps duplicates and is faster — prefer it when you know there's no overlap or duplicates are fine. |
| What is denormalization and why use it in a data warehouse / Gold layer? | • Combining tables / adding redundant columns to reduce joins and speed up read-heavy analytical queries.<br>• Trade-off: more storage and potential update anomalies, but big read-performance win — typical of star schema fact/dim tables in the Gold layer. |
| Write a query to find duplicate rows in a table. | • SELECT col1, col2, COUNT(*) FROM table GROUP BY col1, col2 HAVING COUNT(*) > 1; |

</details>

## 2.3 Practice Query Patterns

### Find the Nth highest value per group

```
WITH ranked AS (
  SELECT *, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
  FROM employees
)
SELECT * FROM ranked WHERE rnk = 2;
```

### Running total / cumulative sum

```
SELECT order_date, amount,
       SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

### Month-over-month comparison using LAG()

```
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
       revenue - LAG(revenue) OVER (ORDER BY month) AS mom_change
FROM monthly_sales;
```

### Find employees earning more than their manager (self join)

```
SELECT e.name AS employee, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

### Delete duplicate rows but keep one copy

```
WITH cte AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY col1, col2 ORDER BY id) AS rn
  FROM my_table
)
DELETE FROM cte WHERE rn > 1;
```

### Gaps-and-islands: find consecutive active days per user

```
WITH t AS (
  SELECT user_id, activity_date,
         DATEDIFF(day, 0, activity_date) - ROW_NUMBER() OVER (
             PARTITION BY user_id ORDER BY activity_date) AS grp
  FROM user_activity
)
SELECT user_id, MIN(activity_date) AS streak_start, MAX(activity_date) AS streak_end
FROM t GROUP BY user_id, grp;
```

---

# 3. PYSPARK

Direct match to resume: "Migrated legacy pipelines from Talend to Databricks using PySpark", "Optimized Spark jobs — achieved a 30% increase in processing efficiency." Expect deep-dive questions here.

## 3.1 Core Concepts to Revise

- Spark architecture — the Driver runs your main program, builds the logical plan, and schedules work; Executors are processes on worker nodes that run tasks and hold cached data; the Cluster Manager (YARN, Kubernetes, or Databricks' own) allocates executors to your application. Each action triggers a Job, Spark breaks it into Stages at shuffle boundaries, and each Stage splits into parallel Tasks — the whole plan is internally a DAG (directed acyclic graph).
- RDD vs DataFrame vs Dataset — RDDs are the original low-level, untyped distributed collection API: flexible but with no built-in query optimization. DataFrames add a schema, letting Spark SQL's Catalyst optimizer understand and optimize operations (predicate pushdown, column pruning) — this is the default choice. Datasets add compile-time type safety on top of DataFrames but are Scala/Java-only; PySpark DataFrames already get the Catalyst/Tungsten benefits.
- Transformations vs Actions; Lazy Evaluation — transformations (select, filter, withColumn, join, groupBy) don't execute anything immediately, they just extend a logical plan. Actions (count(), collect(), show(), write()) trigger execution of the whole accumulated plan. Because Spark waits for an action, Catalyst can optimize the entire chain together rather than executing each step in isolation.
- Narrow vs wide transformations — a narrow transformation (map, filter, select) is computed on each partition independently, with no data movement between executors. A wide transformation (groupBy, join, distinct, orderBy) requires rows with the same key to be brought together across partitions — a shuffle — making wide transformations the most expensive part of most Spark jobs.
- Partitioning — data is split into partitions, the unit of parallelism; more partitions generally means more parallel tasks, up to cluster capacity. repartition(n) does a full shuffle and can increase or decrease partition count while balancing data evenly; coalesce(n) only decreases partition count and avoids a full shuffle by merging existing partitions — the cheaper choice when you just need fewer output files after a filter.
- Shuffle — any wide transformation causes a shuffle: data is written to disk, sent across the network, and read back grouped by key on a different executor. This disk + network I/O is the most expensive operation type in Spark, which is why so much performance tuning is really about minimizing or working around shuffles. spark.sql.shuffle.partitions (default 200) controls how many partitions are created after a shuffle — too few means huge slow partitions, too many means excessive overhead from tiny tasks.
- Joins — a standard join between two large tables uses a shuffle (sort-merge join): both sides are shuffled so matching keys land on the same executor, then sorted and merged. A broadcast join skips the shuffle entirely by sending a full copy of the smaller table to every executor when it's small enough to fit in memory (spark.sql.autoBroadcastJoinThreshold) — the single biggest join optimization when one side is a small dimension table.
- Data skew — happens when a few partition keys hold a disproportionate share of the data (e.g., one customer_id with millions of rows), so the tasks processing those partitions become stragglers the whole job waits on. Fixes include salting (adding a random suffix to the skewed key to spread it across more partitions, then aggregating the salted results back together), using a broadcast join if one side is small, or letting Adaptive Query Execution (AQE) detect and split skewed partitions automatically.
- Caching/Persistence — cache() (shorthand for persist(MEMORY_AND_DISK) in most versions) stores a DataFrame's computed result so it isn't recomputed from scratch on every use. Because of Spark's laziness, a DataFrame used in three different actions would otherwise be recomputed three separate times — caching pays a one-time cost so every later action reads from the cache instead of redoing the entire upstream DAG.
- Catalyst Optimizer — every DataFrame/SQL query passes through Analysis (resolve column/table names against the schema), Logical Optimization (rule-based rewrites like predicate pushdown and constant folding), Physical Planning (cost multiple execution strategies and pick the cheapest, e.g., broadcast vs sort-merge join), and Code Generation (compile the chosen plan into optimized JVM bytecode) — this is what makes DataFrame/SQL code far faster than equivalent hand-written RDD code.
- Adaptive Query Execution (AQE) — re-optimizes the physical plan at runtime using real statistics from completed stages instead of relying only on pre-execution estimates. In practice it dynamically coalesces too-small shuffle partitions into fewer larger ones, can switch a sort-merge join to a broadcast join mid-query if a table turns out smaller than expected, and can split skewed partitions automatically — three common manual tuning tasks, now handled by Spark itself.
- UDFs vs built-in functions — a Python UDF is a black box to Catalyst: it can't be pushed down or reordered, and every row is serialized from the JVM to a Python process and back, which is slow. Built-in Spark SQL functions (pyspark.sql.functions) run entirely inside the JVM and are fully optimizable. When a UDF can't be avoided, a Pandas UDF (vectorized, using Apache Arrow to batch rows) is dramatically faster than a row-at-a-time UDF.
- File formats — Parquet is columnar, compressed, and splittable, and supports predicate/column pushdown (the engine can skip whole column chunks or row groups that don't match a filter), which is why it's the default for analytical workloads. CSV/JSON are row-based, human-readable, but slower to scan and lack a strongly enforced schema. Delta/ORC/Avro build further capabilities on top of columnar storage — Delta in particular layers ACID transactions and time travel on top of Parquet files.
- Structured Streaming — Spark processes streaming data as a sequence of small micro-batches rather than one record at a time, using the same DataFrame API as batch jobs. A watermark tells Spark how long to wait for late-arriving data before finalizing a windowed aggregation, and triggers control how often a new micro-batch runs (e.g., every 10 seconds). This unifies the batch and streaming programming model — the same transformation code largely works for both.
- Memory management & OOM — executor memory splits into storage memory (cached data) and execution memory (shuffles, joins, aggregations); when execution memory runs out, Spark spills intermediate data to disk rather than crashing, which slows things down significantly. Out-of-memory errors usually trace back to partitions that are too large, a skewed key, or a broadcast join attempted on a table that turned out too big — diagnosed via the Spark UI's stage/task memory and spill metrics.

## 3.2 Theory Q&A

<details>
<summary><strong>Show Q&amp;A (11 questions)</strong></summary>

| Question | Answer |
|---|---|
| What's the difference between repartition() and coalesce()? | • repartition(n) does a full shuffle, can increase or decrease partitions, gives evenly sized partitions.<br>• coalesce(n) merges existing partitions without a full shuffle (only decreases partitions) — cheaper, but can lead to uneven partition sizes.<br>• Use coalesce() when writing fewer output files after a filter; use repartition() when you need balanced parallelism, e.g., before a wide transformation. |
| What is data skew and how did you (or would you) fix it? | • Skew = one or few partitions have disproportionately more data (e.g., a join key with a dominant value), causing a few tasks to run far longer than others — straggler problem.<br>• Fixes: salting the skewed key (add a random suffix to spread it across partitions, then aggregate), broadcasting the smaller table to avoid the shuffle entirely, or enabling Adaptive Query Execution skew join handling (spark.sql.adaptive.skewJoin.enabled). |
| Explain lazy evaluation in Spark. Why does Spark do this? | • Transformations (map, filter, select, join) just build up a logical plan (DAG); nothing executes until an action (count, collect, write) is called.<br>• This lets Catalyst optimize the entire chain of operations together (predicate pushdown, combining filters, choosing best join strategy) instead of executing step by step. |
| Difference between cache() and persist()? | • cache() is shorthand for persist(StorageLevel.MEMORY_AND_DISK) (or MEMORY_ONLY depending on version).<br>• persist() lets you choose the storage level explicitly (MEMORY_ONLY, MEMORY_AND_DISK, DISK_ONLY, with/without serialization or replication).<br>• Use when a DataFrame is reused across multiple actions, to avoid recomputing the whole DAG each time. |
| What is a broadcast join, and when do you use it? | • When one DataFrame is small enough to fit in executor memory, Spark sends (broadcasts) a full copy to every executor, avoiding the expensive shuffle of the large table.<br>• Triggered automatically below spark.sql.autoBroadcastJoinThreshold, or explicitly via broadcast(df) — typical for fact-vs-small-dimension joins. |
| How does the Catalyst Optimizer work? | • Analysis: resolves column/table references against the catalog.<br>• Logical Optimization: applies rule-based optimizations (predicate pushdown, constant folding, column pruning).<br>• Physical Planning: generates multiple physical plans and picks the most efficient using cost-based optimization.<br>• Code Generation: generates optimized JVM bytecode (whole-stage code generation) for execution. |
| Why are UDFs discouraged in PySpark? What's the alternative? | • UDFs are a black box to Catalyst (can't be optimized/pushed down) and incur Python<->JVM serialization overhead per row, which is slow.<br>• Prefer built-in Spark SQL functions (pyspark.sql.functions) wherever possible; if a UDF is unavoidable, use a Pandas UDF (vectorized, uses Apache Arrow) for much better performance. |
| What is the role of the Driver vs Executors? | • Driver: runs the main program, builds the DAG, schedules tasks, and coordinates executors; also collects final results for actions like collect().<br>• Executors: run on worker nodes, execute the assigned tasks (transform partitions of data), and report status/results back to the driver. |
| How did you migrate pipelines from Talend to Databricks/PySpark? What challenges did you face? | • Re-implemented Talend job logic (lookups, joins, transformations, scheduling) as PySpark DataFrame transformations/notebooks orchestrated via ADF.<br>• Challenges typically include: replicating Talend's row-level/cell-level transformations in a distributed, partition-based engine; reworking incremental load logic; performance tuning (avoiding unnecessary shuffles, right-sizing clusters); and validating output parity between old and new pipelines (reconciliation). |
| You said you improved Spark job efficiency by 30% — what levers did you pull? | • Likely combination of: switching from default to broadcast joins for small dimension tables, repartitioning/coalescing appropriately to avoid too many small tasks or too few big ones, caching reused DataFrames, pruning unnecessary columns early, choosing Parquet/Delta with partition pruning, tuning cluster size/executor memory, and removing UDFs in favor of built-ins.<br>• Be ready to describe the before/after cluster config, the job's DAG, and how you identified the bottleneck (Spark UI: stages, shuffle read/write, task skew). |
| What's the difference between map() and mapPartitions()? | • map() applies a function row by row.<br>• mapPartitions() applies a function once per partition (an iterator of rows) — useful to reduce per-row overhead (e.g., opening a DB connection once per partition instead of once per row). |

</details>

## 3.3 Coding Questions

### Q1. Create a SparkSession and read a Parquet file

```
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('DECoding').getOrCreate()
df = spark.read.parquet('/mnt/data/silver/orders')
df.printSchema()
```

### Q2. Find duplicate rows in a DataFrame

```
from pyspark.sql import functions as F
dups = (df.groupBy('order_id').count()
          .filter(F.col('count') > 1))
dups.show()
```

### Q3. Window function — running total per customer

```
from pyspark.sql.window import Window
from pyspark.sql import functions as F
w = Window.partitionBy('customer_id').orderBy('order_date')
df = df.withColumn('running_total', F.sum('amount').over(w))
```

### Q4. Broadcast join example (small dimension table)

```
from pyspark.sql.functions import broadcast
result = fact_df.join(broadcast(dim_df), 'product_id', 'left')
```

### Q5. Handle skewed join key via salting

```
from pyspark.sql import functions as F
N = 10
fact_salted = fact_df.withColumn('salt', (F.rand() * N).cast('int'))
dim_exploded = dim_df.withColumn('salt', F.explode(F.array([F.lit(i) for i in range(N)])))
result = fact_salted.join(dim_exploded, ['join_key', 'salt'])
```

### Q6. Incremental / upsert (merge) pattern into a Delta table

```
from delta.tables import DeltaTable
target = DeltaTable.forPath(spark, '/mnt/silver/customers')
(target.alias('t')
   .merge(source_df.alias('s'), 't.customer_id = s.customer_id')
   .whenMatchedUpdateAll()
   .whenNotMatchedInsertAll()
   .execute())
```

### Q7. Read incrementally with Auto Loader (cloudFiles) — Bronze layer ingestion

```
df = (spark.readStream.format('cloudFiles')
      .option('cloudFiles.format', 'json')
      .option('cloudFiles.schemaLocation', '/mnt/schema/orders')
      .load('/mnt/raw/orders'))
(df.writeStream.format('delta')
   .option('checkpointLocation', '/mnt/checkpoints/orders')
   .start('/mnt/bronze/orders'))
```

---

# 4. AZURE DATABRICKS, ADF, ADLS & CLOUD

Directly tied to resume: Azure Databricks, Microsoft Fabric, Azure Data Factory, Azure Data Lake Storage, medallion architecture (Bronze/Silver/Gold), plus GCP/BigQuery certifications.

## 4.1 Core Concepts to Revise

- Lakehouse architecture — a traditional setup forces a choice between a data lake (cheap, flexible-schema object storage, but no transactions or strong performance guarantees) and a data warehouse (fast, ACID-compliant, governed, but expensive and rigid-schema). The Lakehouse pattern stores everything in open file formats in cheap object storage but adds a transactional metadata layer on top (Delta Lake), giving warehouse-like reliability, performance, and governance directly on the lake — one platform instead of stitching two together.
- Delta Lake — a storage layer sitting on top of plain Parquet files, adding a transaction log (the _delta_log folder) that records every change. That log enables ACID transactions (a write fully succeeds or fully fails, even with concurrent writers), schema enforcement and controlled evolution, time travel (querying any previous version of the table), and efficient upserts/deletes via MERGE INTO — none of which plain Parquet files support on their own.
- Medallion Architecture — a three-layer pattern for organizing a lakehouse. Bronze holds raw data ingested as-is from source systems, preserving the original structure for traceability and reprocessing. Silver applies cleansing, deduplication, type casting, and joins/conformance across sources, producing validated, queryable tables. Gold holds business-level aggregates and dimensional models shaped for BI tools and reporting — each layer trades some "rawness" for usability, and reprocessing can always restart from Bronze if something downstream needs fixing.
- Cluster types — an All-Purpose cluster is long-running and shared across users/notebooks for interactive development, so you pay for it the whole time it's running, even idle. A Job cluster is created fresh just for one scheduled job and terminates automatically when the job finishes — cheaper and safer for production. Cluster pools keep idle VMs on standby so job clusters start in seconds instead of minutes, at the cost of paying for that idle capacity.
- Unity Catalog — before Unity Catalog, permissions and metadata were scoped per-workspace (separate Hive metastores), making governance across multiple workspaces inconsistent. Unity Catalog centralizes this at the account level with a three-level namespace (catalog.schema.table), so the same governance rules, lineage tracking, and audit logs apply everywhere a table is used, across every workspace.
- Notebooks & Workflows — a Databricks Job (Workflow) chains multiple notebook/script tasks into a DAG with explicit dependencies (task B only runs after task A succeeds), passes parameters between tasks, and runs on a cron-like schedule. This is how interactive, ad hoc notebook development turns into a reliable, scheduled production pipeline.
- Time Travel — because every change to a Delta table is recorded in its transaction log, you can query the table as it looked at a previous version (VERSION AS OF) or timestamp (TIMESTAMP AS OF), and DESCRIBE HISTORY shows the full list of operations. Invaluable for auditing exactly what changed and when, rolling back a bad write, or debugging a downstream data-quality issue by comparing today's data against yesterday's.
- OPTIMIZE & Z-ORDER / VACUUM — frequent small writes create many tiny files, which slows down reads because the engine has to open and read metadata from each one individually (the "small file problem"). OPTIMIZE compacts small files into fewer, larger ones. ZORDER BY co-locates rows with similar values in a chosen column physically close together on disk, so filters on that column skip reading far more data. VACUUM removes old, no-longer-referenced data files to reclaim storage, since Delta keeps old file versions around to support time travel.
- Azure Data Factory (ADF) — a Linked Service stores connection information for a system ADF talks to. A Dataset points to a specific structure within that linked service (a file path, a table). An Activity is the actual unit of work — Copy Activity moves data, a Databricks Notebook Activity triggers a notebook run, Lookup reads a small amount of data, ForEach loops over a collection, If/Switch add conditional branching. A Trigger decides when the pipeline runs — fixed schedule, tumbling window (time-sliced with built-in retry/backfill), or event-based (e.g., a new file landing in storage).
- Azure Data Lake Storage Gen2 (ADLS) — combines Blob Storage's low cost and scale with a hierarchical namespace (real folders/directories, not just a flat key-value namespace), making operations like renaming or deleting a "directory" fast and atomic instead of requiring a rewrite of every key. Access is controlled through Azure RBAC (container level) and POSIX-style ACLs (file/folder level); Databricks reads/writes it via a mount point (workspace-wide path alias) or directly via abfss:// paths.
- Microsoft Fabric — Microsoft's newer, unified SaaS analytics platform: one underlying storage layer (OneLake, built on open Delta/Parquet) shared across Lakehouse, Data Warehouse, and Power BI items, so storage doesn't need to be managed separately or moved between tools. It overlaps conceptually with the Databricks Lakehouse but trades some of Databricks' deep Spark-engine control and ecosystem maturity for a simpler, more integrated, lower-config experience.
- Secrets management — Databricks secret scopes (especially ones backed by Azure Key Vault) let notebooks retrieve credentials at runtime via dbutils.secrets.get(scope, key) without ever hardcoding a password or connection string in source code — important for security and so notebooks can be shared or version-controlled without leaking credentials.
- Mount points vs direct access — dbutils.fs.mount() creates a persistent alias in the workspace pointing at a storage location, so every notebook can reference /mnt/something; simple, but the credentials behind the mount are effectively shared by every user with workspace access, a governance weak point. The modern approach uses Unity Catalog external locations/storage credentials with direct abfss:// paths, giving fine-grained, per-user/per-group access control.
- GCP/BigQuery — a fully serverless, columnar data warehouse with no infrastructure to manage, billed either by bytes scanned (on-demand) or reserved "slots" (compute capacity). Partitioning a table (commonly by date) and clustering it by frequently filtered columns dramatically cuts bytes scanned per query, directly reducing cost and query time. External tables let BigQuery query data sitting in GCS directly without first loading it in.
- Monitoring/cost — job clusters cost less than all-purpose clusters for the same workload since they only run for the job's duration, but right-sizing matters both ways: too small and jobs run slowly or spill/OOM; too large and you pay for idle capacity. The Spark UI (and Ganglia metrics) show per-stage timing, shuffle read/write volume, and task-level skew — the standard way to diagnose where a slow job is spending its time before deciding what to tune.

## 4.2 Theory Q&A

<details>
<summary><strong>Show Q&amp;A (12 questions)</strong></summary>

| Question | Answer |
|---|---|
| Walk me through the medallion architecture you implemented. | • Bronze: raw ingestion from source systems (APIs, files, DBs) as-is, often append-only, preserving original format/structure for traceability.<br>• Silver: cleansing, deduplication, type casting, schema enforcement, joins/conformance across sources — queryable, validated data.<br>• Gold: business-level aggregates and dimensional models (facts/dimensions) tailored for reporting/BI consumption, optimized for read performance.<br>• Each layer is a Delta table; Databricks notebooks orchestrated by ADF pipelines move data layer by layer. |
| What is Delta Lake and why use it over plain Parquet? | • Delta Lake adds a transaction log (_delta_log) on top of Parquet files, giving ACID transactions, schema enforcement and evolution, time travel, and efficient upserts/deletes (MERGE INTO) — none of which plain Parquet supports natively.<br>• It also unifies batch and streaming reads/writes on the same table. |
| What is the small file problem and how do you solve it in Databricks? | • Frequent small writes (e.g., streaming micro-batches) create many tiny files, hurting read performance (overhead of opening many files).<br>• Solved with OPTIMIZE (compacts small files into larger ones) and Z-ORDER (co-locates related data by column to speed up filtering); auto-compaction/optimized writes can also be enabled. |
| Difference between an All-Purpose cluster and a Job cluster? | • All-purpose: long-running, shared by multiple users/notebooks for interactive development — more expensive when idle.<br>• Job cluster: spun up only for a specific scheduled job and terminated after — cheaper, recommended for production pipelines. |
| What is Unity Catalog and what problem does it solve? | • Centralized, account-level governance layer across workspaces — single place for access control, auditing, lineage, and data discovery using a 3-level namespace (catalog.schema.table), replacing workspace-scoped Hive metastores. |
| How does Auto Loader (cloudFiles) help with incremental ingestion in the Bronze layer? | • Automatically and incrementally detects new files landing in cloud storage (ADLS/S3/GCS) without listing the whole directory each time, using checkpointing — efficient for continuously arriving raw files versus full-directory scans. |
| Explain an ADF pipeline you've built: Linked Service, Dataset, Activity, Trigger. | • Linked Service: the connection string/credentials to a source or sink (e.g., ADLS, SQL DB, REST API).<br>• Dataset: points to the specific data structure within that linked service (a file path, table).<br>• Activity: the unit of work — Copy Activity (move data), Databricks Notebook Activity (run transformation), Lookup, ForEach (loop), If/Switch (conditional branching).<br>• Trigger: what kicks off the pipeline — schedule trigger (cron-like), tumbling window (time-sliced, supports backfill/retry), or event-based trigger (e.g., file arrival in ADLS). |
| How do you handle schema drift / schema evolution in your pipelines? | • Delta Lake supports schema evolution (mergeSchema option) for adding new columns; for stricter control, enforce schema validation in Silver layer and quarantine/flag records that don't conform.<br>• Auto Loader also supports schema inference and evolution tracking via schemaLocation. |
| How do you secure credentials/secrets in Databricks notebooks? | • Use Azure Key Vault-backed secret scopes (dbutils.secrets.get(scope, key)) instead of hardcoding connection strings/passwords in notebooks. |
| What's the difference between a mount point and using abfss:// paths directly to access ADLS from Databricks? | • Mount points (dbutils.fs.mount) create a persistent, workspace-wide alias to the storage path — simpler syntax but a legacy approach with governance/security drawbacks (shared across all users).<br>• Direct abfss:// access with Unity Catalog-managed credentials (storage credentials / external locations) is the modern, more secure, fine-grained approach. |
| How is Microsoft Fabric different from Azure Databricks? | • Fabric is a unified SaaS analytics suite (OneLake storage, Lakehouse/Warehouse items, Data Factory-like pipelines, Power BI built-in) aimed at simplifying the stack with one underlying storage format (Delta/Parquet in OneLake).<br>• Databricks is a more open, engine-focused lakehouse platform with deeper Spark control, broader ecosystem/language support, and more mature governance (Unity Catalog) and ML tooling. |
| From your GCP/BigQuery certification — how would you optimize a BigQuery table for cost and performance? | • Partition the table (commonly by date) and cluster by frequently filtered columns to reduce bytes scanned.<br>• Avoid SELECT *, use approximate aggregation functions where exact precision isn't needed, and materialize frequently-used query results. |

</details>

## 4.3 Useful Commands / Snippets

### Delta Lake time travel

```
SELECT * FROM customers VERSION AS OF 5;
SELECT * FROM customers TIMESTAMP AS OF '2026-06-01';
DESCRIBE HISTORY customers;
```

### Optimize + Z-Order (Bronze/Silver maintenance)

```
OPTIMIZE silver.orders ZORDER BY (customer_id);
VACUUM silver.orders RETAIN 168 HOURS;
```

### Create table with Unity Catalog 3-level namespace

```
CREATE TABLE main.gold.sales_summary (
  region STRING, total_sales DOUBLE, sales_month DATE
) USING DELTA;
```

---

# 5. PROJECT / RESUME-BASED QUESTIONS

These are mapped directly to your resume bullets. Prepare a crisp STAR-style answer (Situation, Task, Action, Result with numbers) for each — interviewers will probe deeper on whichever one you lead with.

## 5.1 Accenture — Packaged App Development Analyst (Jan 2025–Present)

<details>
<summary><strong>Show Q&amp;A (5 questions)</strong></summary>

| Question | Answer |
|---|---|
| Walk me through the Talend-to-Databricks migration project end-to-end. | • Start with why: Talend was likely costlier/slower to scale or harder to maintain than a cloud-native Spark engine.<br>• Explain your approach: mapped each Talend job's transformation logic (joins, lookups, aggregations) to PySpark DataFrame operations; rebuilt orchestration in Databricks Workflows/ADF.<br>• Mention validation: how you ensured data parity between old (Talend) and new (Databricks) outputs — row counts, checksum/hash comparisons, sample-level reconciliation.<br>• Result: quantify — efficiency/processing speed improvement, reduced infra cost, reduced manual maintenance. |
| Why move away from Talend? What are the trade-offs of ETL tools like Talend vs a code-first Spark/Databricks approach? | • Talend (low-code/GUI ETL) is easier for non-engineers but harder to scale, version control, and unit test; Spark/Databricks is code-first (PySpark/SQL), more scalable, integrates with CI/CD and Git, and leverages distributed compute directly.<br>• Trade-off: steeper learning curve and dependency on engineering skill vs visual simplicity. |
| Describe your medallion architecture (Bronze/Silver/Gold) implementation with ADF + Databricks + ADLS. | • ADF pipeline triggers ingestion of source data into ADLS Bronze (raw, append-only).<br>• Databricks notebook (PySpark) reads Bronze, applies cleansing/validation/dedup rules, writes to Silver as Delta tables.<br>• Another notebook aggregates/joins Silver tables into business-ready Gold tables for reporting/Power BI consumption.<br>• ADF orchestrates the whole chain with dependency, retry, and alerting logic. |
| You said Spark job optimization gave a 30% efficiency gain — explain exactly what you changed and how you measured it. | • Be ready to describe a concrete before/after: e.g., switched a large shuffle join to a broadcast join, reduced unnecessary repartition() calls, removed a Python UDF in favor of a built-in function, or right-sized cluster/executor memory.<br>• Measurement: job duration in Databricks job runs, Spark UI stage/task timings (shuffle read/write bytes), or cluster cost/DBU usage before vs after. |
| How did you implement distributed processing with PySpark for scalable transformation? | • Explain partitioning strategy for the dataset, use of DataFrame transformations instead of row-by-row Python loops, and how Spark distributes those transformations as parallel tasks across executors. |

</details>

## 5.2 SLO Technologies (AdvaRisk) — Jr. Data Engineer (Sep 2023–Jan 2025)

<details>
<summary><strong>Show Q&amp;A (5 questions)</strong></summary>

| Question | Answer |
|---|---|
| Describe an Azure Data Factory pipeline you designed for data integration. | • Mention source systems integrated, the activities used (Copy, Lookup, ForEach for multiple files/tables, conditional branching), and how failures/retries were handled.<br>• Mention parameterization for reusability across environments (dev/test/prod) or across multiple similar source feeds. |
| How did you use Azure Data Lake Storage for big data analytics? | • ADLS Gen2 hierarchical namespace for organizing raw/curated zones, partitioning by date/source for efficient downstream querying, and integration with Databricks via mount points or direct paths. |
| You refined Python and SQL logic in Databricks workflows to improve pipeline performance — give a specific example. | • Example: replacing a slow Python loop with vectorized Pandas/PySpark operations, optimizing a SQL query's join order/indexes, or pushing filtering logic earlier in the pipeline to reduce data volume downstream. |
| You automated data collection pipelines reducing manual intervention by 30% — what was manual before, and what did you automate? | • Describe the original manual process (e.g., manually triggered scripts/files dropped by users), then the orchestration you built (scheduled ADF triggers, Python scripts with error handling/logging/alerting) that removed the need for manual steps.<br>• Tie back to a measurable result: time saved, reduced errors, faster turnaround. |
| AdvaRisk is a risk-assessment company — what kind of data sensitivity/compliance considerations did you have to account for? | • Be ready to discuss data security practices: access control (RBAC on ADLS), secrets management (Key Vault), and data masking/anonymization if PII was involved. |

</details>

## 5.3 Innoplexus Consulting — Associate Software Engineer (Jul 2022–Sep 2023)

<details>
<summary><strong>Show Q&amp;A (4 questions)</strong></summary>

| Question | Answer |
|---|---|
| How did you use Azure SQL for relational data management? | • Discuss schema design, normalization, indexing strategy, and how Azure SQL was used as either a source, staging, or serving layer in the pipeline. |
| You oversaw pipelines integrating into MongoDB and Elasticsearch — why two different stores, and how did you decide what goes where? | • MongoDB (document store): flexible/semi-structured data, fast key-based lookups, good for operational/application-facing data.<br>• Elasticsearch: built for full-text search and near-real-time analytics/aggregations over large volumes — used when search/filtering UX or log/event analytics was the goal.<br>• Explain the ingestion/transformation pipeline that fed both — likely a shared transformation step branching into two sinks based on access pattern needs. |
| What's the difference between MongoDB and a relational database like Azure SQL? When would you pick one over the other? | • MongoDB: schema-less/flexible documents (JSON-like BSON), horizontal scaling, good for unstructured/evolving data.<br>• Azure SQL: strict schema, ACID transactions, strong relational integrity (foreign keys) — better for structured transactional data needing strong consistency. |
| What SDLC practices did you follow (testing, deployment)? | • Mention unit/integration testing of pipeline logic, code reviews, version control (Git), and deployment process (CI/CD pipeline or manual promotion across environments). |

</details>

## 5.4 General Project / Behavioral Follow-ups (be ready regardless of which project you discuss)

- Tell me about a time a pipeline failed in production. What happened and how did you fix/prevent it?
- How do you monitor and alert on pipeline failures or data quality issues?
- How do you handle late-arriving or duplicate data in an incremental pipeline?
- Describe a disagreement with a teammate/lead on a technical approach — how did you resolve it?
- How do you ensure data quality across Bronze -> Silver -> Gold?
- What would you do differently if you rebuilt this pipeline today?
- How do you decide cluster size / compute resources for a Databricks job to balance cost and performance?
- How do you version-control and deploy notebooks/pipelines (CI/CD for data engineering)?

---

# 6. OTHER LIKELY TOPICS (Resume-Driven)

## 6.1 Docker

- Docker packages an application and all its dependencies into a single, portable image, so a pipeline that runs correctly on your laptop runs identically on a teammate's machine or in production — eliminating "works on my machine" environment drift, which matters when a script depends on a specific Python/library version.
- An image is the read-only, packaged blueprint (your code + dependencies + OS layer); a container is a running instance of that image. You can run many containers from the same image, each isolated from the others and from the host.
- docker build creates an image from a Dockerfile; docker run starts a container from an image; docker ps lists running containers; docker-compose defines and starts multiple related containers together (e.g., a Spark master + worker + Jupyter stack) from a single YAML file instead of running each docker run command by hand.
- To containerize a Python ETL script: write a Dockerfile that starts from a Python base image, copies the script and a requirements.txt, installs dependencies with pip, and sets the script as the container's entrypoint — docker build + docker run then reproduces the exact same environment anywhere.

## 6.2 Git

- A common branching strategy keeps main always deployable, creates a short-lived feature branch per change, and merges back via pull request after review — this isolates risky, in-progress work from what's actually running in production.
- git merge creates a new commit combining two branches' histories, preserving exactly what happened (including a merge commit). git rebase replays your branch's commits on top of another branch's latest commit, producing a cleaner linear history but rewriting commit hashes — avoid rebasing commits that have already been pushed and shared with others.
- A merge conflict happens when the same lines of a file were changed differently on both branches; Git marks the conflicting sections with <<<<<<< / ======= / >>>>>>> markers, and you manually choose (or combine) the correct version before committing the resolution.
- A typical PR review process: open a pull request with a clear description, have at least one teammate review the diff and check CI (tests, linting), address feedback, then merge — this catches bugs and keeps pipeline code consistent before it reaches main/production.

## 6.3 Gen AI (listed on resume — be ready for at least a high-level conversation)

- How can Gen AI/LLMs be applied in data engineering? (e.g., auto-generating documentation/data dictionaries, natural-language-to-SQL assistants, anomaly detection narratives, code review copilots.)
- Have you used Databricks' Mosaic AI / Genie, or any LLM API, in a data pipeline context? Be ready with a specific example or say plainly if it was exploratory/POC level.
- What is RAG (Retrieval-Augmented Generation) at a high level, and how might it relate to a data platform (e.g., vector search over a knowledge base built from your data)?

## 6.4 Certifications — be ready to defend these in conversation

- Databricks Data Engineer Associate: be ready to discuss Delta Lake, medallion architecture, Databricks SQL, basic Spark architecture, and job orchestration — this exam's core syllabus.
- Google Cloud Professional Data Engineer: be ready to discuss BigQuery, Dataflow/Dataproc, Pub/Sub, data pipeline design, and ML pipeline basics on GCP.
- Google Cloud Professional Database Engineer: Cloud SQL, Spanner, AlloyDB basics, migration strategies, choosing the right GCP database for a workload.
- Microsoft Azure AI Fundamentals (AI-900): basic ML concepts, Azure Cognitive Services, responsible AI principles — keep this lightweight/conceptual.

## 6.5 Elasticsearch (from Innoplexus experience)

- What is Elasticsearch used for? (Distributed search & analytics engine, built on Lucene, inverted index for fast full-text search.)
- Index vs document vs shard — basic terminology.
- How is data typically ingested into Elasticsearch from a pipeline (Logstash/Beats, or direct bulk API calls)?

## 6.6 General / HR Round Prep

- Walk me through your resume / Tell me about yourself — prepare a tight 90-second version covering: 4 years as a Data Engineer, current role at Accenture, core stack (Python, SQL, PySpark, Azure Databricks), and one standout achievement (Talend migration or the 30% efficiency gain).
- Why are you looking to switch roles right now?
- Why this company / why this role?
- Where do you see yourself in 3-5 years?
- Strengths and weaknesses — pick a real, minor weakness with a concrete improvement action.
- How do you stay updated with new data engineering tools/trends? (mention certifications, hands-on POCs, following Databricks/Azure release notes.)
- Salary expectations / notice period — have a number/range ready.
- Any questions for us? — prepare 2-3 genuine questions about the team's tech stack, data scale, or current challenges.

---

# 7. 5-Minute-Before-the-Interview Cheat Sheet

## Say this confidently if asked to introduce yourself

> "I'm a Data Engineer with 4 years of experience, currently at Accenture, where I migrated legacy Talend pipelines to Databricks using PySpark and improved Spark job efficiency by 30%. Before that, at SLO Technologies (AdvaRisk), I built ADF pipelines and automated data collection, cutting manual effort by 30%. My core stack is Python, SQL, PySpark, and Azure Databricks, and I follow the medallion (Bronze/Silver/Gold) architecture for building reliable, scalable pipelines."

## Numbers to keep ready

- 30% increase in Spark processing efficiency (Accenture).
- 30% reduction in manual intervention via automation (SLO Technologies / AdvaRisk).
- 4 years total experience; 3 companies; Databricks Data Engineer Associate + 2x GCP + Azure AI-900 certified.

## Fast recall - PySpark performance levers

- Broadcast join for small tables, avoid shuffle.
- repartition() vs coalesce() - pick based on increasing/decreasing partitions.
- Cache reused DataFrames; avoid UDFs; use built-in functions.
- Salting to fix data skew.
- Parquet/Delta + partition pruning + Z-ORDER + OPTIMIZE.

## Fast recall - SQL

- ROW_NUMBER (no ties) vs RANK (gaps) vs DENSE_RANK (no gaps).
- WHERE filters rows before grouping; HAVING filters after.
- SCD Type 1 = overwrite, Type 2 = history with effective/expiry dates.

## Fast recall - Databricks / Architecture

- Bronze = raw, Silver = cleansed/conformed, Gold = business aggregates.
- Delta Lake = ACID + schema evolution + time travel + MERGE on top of Parquet.
- Unity Catalog = governance, 3-level namespace (catalog.schema.table).

> Good luck, Tejas. Speak in specifics, lead with numbers, and tie every answer back to a real pipeline you built.
