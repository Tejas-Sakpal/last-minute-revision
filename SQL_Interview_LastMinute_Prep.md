# SQL Interview — Last-Minute Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Query** to see it in action, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Built for a 2–4 hour final revision.
>
> *Examples use standard SQL; minor syntax differs across MySQL / PostgreSQL / SQL Server / Oracle — noted where it matters.*

---

## Table of Contents

1. [SQL Fundamentals & Command Categories](#1-sql-fundamentals--command-categories)
2. [Data Types, Keys & Constraints](#2-data-types-keys--constraints)
3. [Core Querying: SELECT, WHERE, ORDER BY](#3-core-querying)
4. [Aggregation: GROUP BY & HAVING](#4-aggregation-group-by--having)
5. [Joins](#5-joins)
6. [Subqueries & CTEs](#6-subqueries--ctes)
7. [Set Operators: UNION, INTERSECT, EXCEPT](#7-set-operators)
8. [NULL Handling](#8-null-handling)
9. [String, Date & Conditional Functions](#9-string-date--conditional-functions)
10. [Window Functions](#10-window-functions)
11. [Indexing & Performance](#11-indexing--performance)
12. [Normalization & Database Design](#12-normalization--database-design)
13. [Transactions & ACID](#13-transactions--acid)
14. [Views, Stored Procedures, Triggers](#14-views-stored-procedures-triggers)
15. [Common Query Programs (with solutions)](#15-common-query-programs)
16. [Tricky / Gotcha Questions](#16-tricky--gotcha-questions)
17. [Rapid-Fire One-Liners](#17-rapid-fire-one-liners)
18. [Last-Minute Checklist](#18-last-minute-checklist)

---

## 1. SQL Fundamentals & Command Categories

### Theory

**SQL (Structured Query Language)** is the standard language for managing and querying **relational databases**, where data is stored in **tables** (rows = records, columns = fields).

**The five command categories — know these cold:**

| Category | Full Form | Commands | Purpose |
|---|---|---|---|
| **DDL** | Data Definition | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Define/modify schema |
| **DML** | Data Manipulation | `INSERT`, `UPDATE`, `DELETE` | Modify data |
| **DQL** | Data Query | `SELECT` | Retrieve data |
| **DCL** | Data Control | `GRANT`, `REVOKE` | Permissions |
| **TCL** | Transaction Control | `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Manage transactions |

**DELETE vs TRUNCATE vs DROP** (a guaranteed question):

| | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Type | DML | DDL | DDL |
| Removes | Selected rows | All rows | Entire table |
| `WHERE`? | Yes | No | No |
| Rollback? | Yes | Usually no | No |
| Resets identity? | No | Yes | N/A |
| Speed | Slow (row by row) | Fast | Fast |

### Query

```sql
-- DDL
CREATE TABLE employees (
    id       INT PRIMARY KEY,
    name     VARCHAR(50) NOT NULL,
    dept_id  INT,
    salary   DECIMAL(10,2)
);

-- DML
INSERT INTO employees VALUES (1, 'Asha', 10, 50000);
UPDATE employees SET salary = 55000 WHERE id = 1;
DELETE FROM employees WHERE id = 1;

-- DQL
SELECT name, salary FROM employees;
```

### Interview Q&A

**Q: Difference between DELETE, TRUNCATE, and DROP?**
**DELETE** removes specific rows (with `WHERE`) and can be rolled back. **TRUNCATE** removes *all* rows quickly, resets identity, and usually can't be rolled back. **DROP** deletes the entire table structure plus data.

**Q: Is SQL case-sensitive?**
SQL **keywords** are not case-sensitive (`SELECT` = `select`). Data and identifiers may be case-sensitive depending on the database/collation.

**Q: What is the difference between SQL and MySQL?**
**SQL** is the language; **MySQL** is a relational database management system (RDBMS) that *uses* SQL. Others: PostgreSQL, Oracle, SQL Server, SQLite.

---

## 2. Data Types, Keys & Constraints

### Theory

**Common data types:** `INT`, `DECIMAL/NUMERIC`, `FLOAT`, `VARCHAR(n)`, `CHAR(n)`, `TEXT`, `DATE`, `DATETIME/TIMESTAMP`, `BOOLEAN`.

**Keys:**

- **Primary Key** — uniquely identifies each row; **NOT NULL + UNIQUE**; one per table.
- **Foreign Key** — a column referencing the primary key of another table; enforces **referential integrity**.
- **Unique Key** — ensures uniqueness but **allows one NULL**; can have many per table.
- **Candidate Key** — any column(s) that *could* be a primary key.
- **Composite Key** — a primary key made of **two or more columns**.
- **Super Key** — a set of columns that uniquely identifies a row (superset of a candidate key).

**Constraints:** `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`.

### Query

```sql
CREATE TABLE orders (
    order_id    INT PRIMARY KEY,
    cust_id     INT NOT NULL,
    amount      DECIMAL(10,2) CHECK (amount > 0),
    status      VARCHAR(20) DEFAULT 'pending',
    created_at  DATE,
    FOREIGN KEY (cust_id) REFERENCES customers(id)
);
```

### Interview Q&A

**Q: Primary key vs unique key?**
Both enforce uniqueness. A **primary key** is `NOT NULL` and there's only **one** per table. A **unique key** allows **one NULL** and you can have **multiple** per table.

**Q: What is a foreign key?**
A column that links to the primary key of another table, enforcing **referential integrity** — you can't insert a child row referencing a non-existent parent, and deletes can be restricted/cascaded.

**Q: Primary key vs candidate key?**
**Candidate keys** are all columns eligible to be the primary key. The DBA picks **one** candidate key as the **primary key**; the rest become alternate keys.

**Q: Can a primary key be NULL?**
No — a primary key must be unique **and** NOT NULL.

---

## 3. Core Querying

### Theory

**Logical order of execution** (different from written order — important!):

`FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`

This is why you **can't use a SELECT alias in WHERE** (WHERE runs before SELECT) but **can use it in ORDER BY**.

Key clauses: `WHERE` (filter rows), `DISTINCT` (remove duplicates), `ORDER BY` (sort, `ASC`/`DESC`), `LIMIT`/`TOP`/`FETCH` (cap rows), `BETWEEN`, `IN`, `LIKE`, `AND/OR/NOT`.

### Query

```sql
SELECT DISTINCT dept_id, name
FROM employees
WHERE salary BETWEEN 40000 AND 80000
  AND name LIKE 'A%'              -- starts with A
  AND dept_id IN (10, 20, 30)
ORDER BY salary DESC
LIMIT 5;                          -- MySQL/Postgres; SQL Server: SELECT TOP 5
```

### Interview Q&A

**Q: WHERE vs HAVING?**
**WHERE** filters **individual rows before** grouping and cannot use aggregate functions. **HAVING** filters **groups after** `GROUP BY` and can use aggregates like `COUNT()`, `SUM()`.

**Q: Why can't you use a column alias in WHERE?**
Because of execution order — `WHERE` runs **before** `SELECT` (where the alias is defined). Aliases work in `ORDER BY` since that runs last.

**Q: `LIKE` wildcards?**
`%` matches any sequence of characters; `_` matches exactly one character. E.g. `'A%'` = starts with A, `'_a%'` = second char is a.

**Q: Difference between `IN` and `EXISTS`?**
`IN` checks membership against a value list/subquery. `EXISTS` checks whether a subquery returns **any rows** and stops at the first match — often faster for large correlated subqueries.

---

## 4. Aggregation: GROUP BY & HAVING

### Theory

**Aggregate functions:** `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`. They collapse many rows into one summary value.

- `GROUP BY` — groups rows sharing a value so aggregates apply per group.
- `HAVING` — filters those groups.
- **`COUNT(*)` counts all rows; `COUNT(col)` ignores NULLs** in that column.

### Query

```sql
-- Departments with average salary above 60k, most-paid first
SELECT dept_id,
       COUNT(*)      AS headcount,
       AVG(salary)   AS avg_salary,
       MAX(salary)   AS top_salary
FROM employees
GROUP BY dept_id
HAVING AVG(salary) > 60000
ORDER BY avg_salary DESC;
```

### Interview Q&A

**Q: `COUNT(*)` vs `COUNT(column)` vs `COUNT(DISTINCT column)`?**
`COUNT(*)` counts **all rows** including NULLs. `COUNT(column)` counts non-NULL values in that column. `COUNT(DISTINCT column)` counts unique non-NULL values.

**Q: Can you use an aggregate in WHERE?**
No — use **HAVING**. `WHERE salary > AVG(salary)` is invalid; aggregates are evaluated after grouping.

**Q: Every non-aggregated SELECT column must be...?**
In the `GROUP BY` clause (strict/ANSI mode). MySQL historically allowed exceptions, but it's bad practice.

---

## 5. Joins

### Theory

A **JOIN** combines rows from two or more tables based on a related column.

| Join | Returns |
|---|---|
| **INNER JOIN** | Only matching rows in both tables |
| **LEFT (OUTER) JOIN** | All left rows + matches from right (NULLs if none) |
| **RIGHT (OUTER) JOIN** | All right rows + matches from left |
| **FULL (OUTER) JOIN** | All rows from both; NULLs where no match |
| **CROSS JOIN** | Cartesian product (every combination) |
| **SELF JOIN** | A table joined to itself (e.g. employee → manager) |

> Visual mnemonic: INNER = intersection, LEFT/RIGHT = one full circle + overlap, FULL = both circles entirely.

### Query

```sql
-- INNER: employees with a matching department
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- LEFT: all employees, even those with no department
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;

-- SELF JOIN: each employee with their manager's name
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- Find employees with NO department (anti-join)
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id
WHERE d.id IS NULL;
```

### Interview Q&A

**Q: INNER JOIN vs LEFT JOIN?**
**INNER JOIN** returns only rows with matches in both tables. **LEFT JOIN** returns **all** rows from the left table; unmatched right-side columns are NULL.

**Q: What is a self join?**
A table joined to itself using aliases — used for hierarchical/related data within one table, e.g. matching employees to their managers.

**Q: What does CROSS JOIN do?**
Produces the **Cartesian product** — every row of table A combined with every row of table B (m × n rows). Rarely used intentionally; often an accidental result of a missing join condition.

**Q: How do you find rows in A that have no match in B?**
A **LEFT JOIN** with `WHERE B.key IS NULL` (anti-join), or `NOT EXISTS` / `NOT IN`.

**Q: Difference between ON and WHERE in a join?**
`ON` defines the **join condition**; `WHERE` filters the **result** afterward. For OUTER joins this matters — a filter in `WHERE` on the right table can effectively turn it into an inner join.

---

## 6. Subqueries & CTEs

### Theory

- **Subquery** — a query nested inside another. Can appear in `SELECT`, `FROM`, or `WHERE`.
- **Scalar subquery** — returns a single value.
- **Correlated subquery** — references the outer query; runs once **per outer row** (slower).
- **CTE (Common Table Expression)** — a named temporary result set via `WITH`, improving readability and enabling **recursion**.

### Query

```sql
-- Non-correlated: employees earning above the company average
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Correlated: employees earning above their OWN department's average
SELECT name, salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary) FROM employees
    WHERE dept_id = e.dept_id
);

-- CTE version (cleaner)
WITH dept_avg AS (
    SELECT dept_id, AVG(salary) AS avg_sal
    FROM employees GROUP BY dept_id
)
SELECT e.name, e.salary
FROM employees e
JOIN dept_avg da ON e.dept_id = da.dept_id
WHERE e.salary > da.avg_sal;
```

### Interview Q&A

**Q: Correlated vs non-correlated subquery?**
A **non-correlated** subquery runs **once** independently. A **correlated** subquery references the outer query and executes **once per outer row**, so it's typically slower.

**Q: What is a CTE and why use it?**
A **Common Table Expression** (`WITH ... AS`) is a named temporary result set used within a single statement. It improves **readability**, avoids repeating subqueries, and supports **recursion** (e.g. org charts).

**Q: CTE vs temporary table vs view?**
A **CTE** exists only for the duration of one statement. A **temp table** is physically stored for the session. A **view** is a saved query definition reusable across sessions.

---

## 7. Set Operators

### Theory

Combine results of two `SELECT` statements (must have same number/compatible types of columns):

- **`UNION`** — combines and **removes duplicates**.
- **`UNION ALL`** — combines and **keeps duplicates** (faster).
- **`INTERSECT`** — rows present in **both**.
- **`EXCEPT`** / **`MINUS`** — rows in the first but not the second.

### Query

```sql
SELECT city FROM customers
UNION                       -- distinct cities across both
SELECT city FROM suppliers;

SELECT product_id FROM orders_2024
INTERSECT                   -- products ordered in both years
SELECT product_id FROM orders_2025;
```

### Interview Q&A

**Q: UNION vs UNION ALL?**
**UNION** removes duplicate rows (extra sort/dedup cost). **UNION ALL** keeps all rows and is **faster**. Use UNION ALL when you know there are no duplicates or don't care.

**Q: UNION vs JOIN?**
**JOIN** combines columns **horizontally** (adds columns from another table). **UNION** stacks rows **vertically** (combines result sets with the same columns).

---

## 8. NULL Handling

### Theory

**NULL means "unknown / missing"** — *not* zero and *not* an empty string. NULL is never equal to anything, including another NULL.

- Test with **`IS NULL`** / **`IS NOT NULL`** — never `= NULL`.
- `COALESCE(a, b, ...)` returns the first non-NULL value.
- `NULLIF(a, b)` returns NULL if `a = b`, else `a`.
- `IFNULL()` (MySQL) / `ISNULL()` (SQL Server) / `NVL()` (Oracle) substitute a default.
- Aggregates (except `COUNT(*)`) **ignore NULLs**.

### Query

```sql
SELECT name,
       COALESCE(phone, 'N/A')        AS phone,    -- default if NULL
       salary + COALESCE(bonus, 0)   AS total_pay -- avoid NULL math
FROM employees
WHERE manager_id IS NOT NULL;
```

### Interview Q&A

**Q: Why does `WHERE col = NULL` return nothing?**
Because comparisons with NULL yield **UNKNOWN**, not TRUE. You must use `IS NULL` / `IS NOT NULL`.

**Q: What does `COALESCE` do?**
Returns the **first non-NULL** value from its arguments — handy for default/fallback values.

**Q: Do aggregate functions count NULLs?**
No — `SUM`, `AVG`, `MIN`, `MAX`, `COUNT(column)` all **skip NULLs**. Only `COUNT(*)` counts every row. (Note: `AVG` divides by the count of non-NULLs.)

---

## 9. String, Date & Conditional Functions

### Theory

**String:** `CONCAT()`, `SUBSTRING()`, `LENGTH()/LEN()`, `UPPER()/LOWER()`, `TRIM()`, `REPLACE()`, `LEFT()/RIGHT()`.

**Date:** `NOW()/GETDATE()/CURRENT_DATE`, `DATEDIFF()`, `DATEADD()`, `EXTRACT()/DATEPART()`, `DATE_FORMAT()`.

**Conditional:** `CASE WHEN ... THEN ... ELSE ... END` — the SQL "if/else".

### Query

```sql
SELECT name,
       UPPER(name)                         AS upper_name,
       LENGTH(name)                        AS name_len,
       CASE
           WHEN salary >= 80000 THEN 'High'
           WHEN salary >= 50000 THEN 'Mid'
           ELSE 'Entry'
       END                                 AS salary_band
FROM employees;
```

### Interview Q&A

**Q: What is a CASE statement?**
SQL's conditional expression — evaluates conditions in order and returns the value for the first true `WHEN`, else the `ELSE` value. Useful for bucketing, pivoting, and conditional aggregation.

**Q: How would you do conditional aggregation?**
Combine `SUM`/`COUNT` with `CASE`: `SUM(CASE WHEN status='paid' THEN amount ELSE 0 END)`.

---

## 10. Window Functions

### Theory

A **window function** performs a calculation across a set of rows **related to the current row**, *without collapsing* them (unlike `GROUP BY`). Defined by `OVER (PARTITION BY ... ORDER BY ...)`.

**Key functions:**

- **Ranking:** `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE(n)`.
- **Offset:** `LAG()`, `LEAD()` (previous/next row).
- **Aggregate windows:** `SUM() OVER(...)`, `AVG() OVER(...)` for running totals/moving averages.

**ROW_NUMBER vs RANK vs DENSE_RANK** (classic question): with a tie at rank 1,
- `ROW_NUMBER` → 1, 2, 3 (always unique)
- `RANK` → 1, 1, 3 (skips after ties)
- `DENSE_RANK` → 1, 1, 2 (no gaps)

### Query

```sql
-- Top earner per department
SELECT name, dept_id, salary,
       RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
FROM employees;

-- Running total of salary by department
SELECT name, dept_id, salary,
       SUM(salary) OVER (PARTITION BY dept_id ORDER BY id) AS running_total
FROM employees;

-- Compare each row to the previous one
SELECT name, salary,
       LAG(salary) OVER (ORDER BY salary) AS prev_salary
FROM employees;
```

**Nth highest salary using a window function:**

```sql
SELECT name, salary
FROM (
    SELECT name, salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 2;          -- 2nd highest
```

### Interview Q&A

**Q: Window function vs GROUP BY?**
`GROUP BY` **collapses** rows into one per group. A window function keeps **every row** and adds the calculated value alongside it — so you can show detail *and* the aggregate together.

**Q: ROW_NUMBER vs RANK vs DENSE_RANK?**
All assign rankings over an ordered partition. `ROW_NUMBER` gives a **unique** sequential number. `RANK` gives ties the same rank and **skips** subsequent numbers. `DENSE_RANK` gives ties the same rank **without gaps**.

**Q: What does PARTITION BY do?**
It divides rows into groups; the window function restarts its calculation for each partition (similar to `GROUP BY`, but rows aren't merged).

---

## 11. Indexing & Performance

### Theory

An **index** is a data structure (usually a **B-tree**) that speeds up data retrieval — like a book's index, the DB jumps straight to matching rows instead of a full table scan. Trade-off: indexes **speed reads but slow writes** (INSERT/UPDATE/DELETE) and use extra storage.

- **Clustered index** — defines the **physical order** of rows; **one per table** (usually the primary key).
- **Non-clustered index** — a separate structure with pointers to rows; **many allowed**.
- **Composite index** — on multiple columns; order matters (leftmost-prefix rule).
- **Unique index** — enforces uniqueness.

**When to index:** columns in `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`, and foreign keys.
**When NOT to:** small tables, columns with low cardinality (many duplicates like gender), heavily-written columns.

**Query optimization tips:** select only needed columns (avoid `SELECT *`), filter early, index join/filter columns, avoid functions on indexed columns in `WHERE` (breaks index use), use `EXPLAIN`/execution plan to diagnose.

### Query

```sql
CREATE INDEX idx_emp_dept ON employees(dept_id);
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

-- Composite index (helps WHERE dept_id=? AND salary>?)
CREATE INDEX idx_dept_salary ON employees(dept_id, salary);

-- Diagnose a slow query
EXPLAIN SELECT * FROM employees WHERE dept_id = 10;
```

### Interview Q&A

**Q: What is an index and what's the trade-off?**
A structure that accelerates **reads** by avoiding full table scans. The cost is **slower writes** and additional disk space, since every index must be updated on data changes.

**Q: Clustered vs non-clustered index?**
A **clustered** index sorts and stores the actual rows in index order — only **one** per table. A **non-clustered** index is a separate lookup structure pointing to the rows — you can have **many**.

**Q: Why might an index not be used?**
Applying a function/calculation to the indexed column in `WHERE` (e.g. `WHERE YEAR(date)=2025`), leading wildcard `LIKE '%abc'`, low selectivity, or the optimizer deciding a scan is cheaper.

**Q: How do you find why a query is slow?**
Use `EXPLAIN` / `EXPLAIN ANALYZE` (or the execution plan) to see scans vs index seeks, then add indexes, rewrite the query, or avoid `SELECT *`.

---

## 12. Normalization & Database Design

### Theory

**Normalization** organizes data to **minimize redundancy** and prevent **insert/update/delete anomalies** by splitting tables based on dependencies.

| Form | Rule |
|---|---|
| **1NF** | Atomic (indivisible) values; no repeating groups; each row unique |
| **2NF** | 1NF + no **partial dependency** (non-key fully depends on the whole composite key) |
| **3NF** | 2NF + no **transitive dependency** (non-key depends only on the key, not other non-keys) |
| **BCNF** | Stricter 3NF: every determinant is a candidate key |

**Denormalization** — deliberately adding redundancy (e.g. duplicating columns) to **improve read performance**, at the cost of more storage and update complexity. Common in reporting/data-warehouse/OLAP systems.

### Query

```sql
-- Un-normalized (bad): repeating data
-- orders(order_id, customer_name, customer_city, product1, product2)

-- Normalized (3NF):
CREATE TABLE customers (id INT PRIMARY KEY, name VARCHAR(50), city VARCHAR(50));
CREATE TABLE products  (id INT PRIMARY KEY, name VARCHAR(50), price DECIMAL(10,2));
CREATE TABLE orders    (id INT PRIMARY KEY, customer_id INT REFERENCES customers(id));
CREATE TABLE order_items (
    order_id   INT REFERENCES orders(id),
    product_id INT REFERENCES products(id),
    qty        INT,
    PRIMARY KEY (order_id, product_id)
);
```

### Interview Q&A

**Q: What is normalization and why do it?**
Structuring tables to **reduce redundancy** and avoid anomalies. Benefits: data integrity, smaller storage, easier maintenance. Cost: more joins, which can slow reads.

**Q: Explain 1NF, 2NF, 3NF simply.**
**1NF:** atomic values, no repeating groups. **2NF:** 1NF + every non-key column depends on the *entire* primary key (no partial dependency). **3NF:** 2NF + non-key columns depend *only* on the key (no transitive dependency).

**Q: Normalization vs denormalization — when to use each?**
**Normalize** for transactional systems (OLTP) where data integrity and writes matter. **Denormalize** for read-heavy analytics/reporting (OLAP) to cut down on expensive joins.

**Q: What are anomalies?**
Problems from redundancy: **insert** anomaly (can't add data without unrelated data), **update** anomaly (must change many rows), **delete** anomaly (losing data unintentionally).

---

## 13. Transactions & ACID

### Theory

A **transaction** is a unit of work executed as a single logical operation — all-or-nothing. Controlled by `BEGIN`/`START TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`.

**ACID properties:**

- **Atomicity** — all operations succeed or none do.
- **Consistency** — the database moves from one valid state to another (constraints hold).
- **Isolation** — concurrent transactions don't interfere; controlled by **isolation levels**.
- **Durability** — once committed, changes survive crashes (persisted to disk).

**Isolation levels & read phenomena:**

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|---|---|---|---|
| Read Uncommitted | ✓ possible | ✓ | ✓ |
| Read Committed | ✗ | ✓ | ✓ |
| Repeatable Read | ✗ | ✗ | ✓ |
| Serializable | ✗ | ✗ | ✗ |

### Query

```sql
START TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    -- if both succeed:
COMMIT;
    -- on error, undo everything:
-- ROLLBACK;
```

### Interview Q&A

**Q: What does ACID stand for?**
**A**tomicity, **C**onsistency, **I**solation, **D**urability — the four guarantees that keep transactions reliable.

**Q: COMMIT vs ROLLBACK?**
`COMMIT` permanently saves all changes in the transaction. `ROLLBACK` undoes them, returning to the last committed state (or a `SAVEPOINT`).

**Q: What is a dirty read?**
Reading data that another transaction has modified but **not yet committed** — which might be rolled back. Prevented at Read Committed and above.

**Q: What is a deadlock?**
Two transactions each holding a lock the other needs, waiting forever. The DB detects it and aborts one (the "victim"). Mitigate by accessing tables in a consistent order and keeping transactions short.

---

## 14. Views, Stored Procedures, Triggers

### Theory

- **View** — a saved `SELECT` query that acts as a virtual table. Simplifies complex queries, restricts column access (security), doesn't store data (unless **materialized**).
- **Stored Procedure** — precompiled reusable SQL (can take parameters, contain logic/loops). Reduces network traffic, centralizes logic.
- **Function (UDF)** — returns a value, usable inside queries (unlike most procedures).
- **Trigger** — SQL that **automatically runs** in response to `INSERT`/`UPDATE`/`DELETE` events (e.g. audit logging).

### Query

```sql
-- View
CREATE VIEW high_earners AS
SELECT name, salary FROM employees WHERE salary > 80000;

-- Stored procedure (MySQL syntax)
DELIMITER //
CREATE PROCEDURE raise_salary(IN emp_id INT, IN pct DECIMAL(5,2))
BEGIN
    UPDATE employees
    SET salary = salary * (1 + pct/100)
    WHERE id = emp_id;
END //
DELIMITER ;

-- Trigger: log every salary change
CREATE TRIGGER log_salary_change
AFTER UPDATE ON employees
FOR EACH ROW
INSERT INTO salary_audit(emp_id, old_sal, new_sal)
VALUES (OLD.id, OLD.salary, NEW.salary);
```

### Interview Q&A

**Q: What is a view? Can you update through it?**
A virtual table from a stored query. **Simple** single-table views are often updatable; views with joins, aggregates, `DISTINCT`, or `GROUP BY` generally are **not**.

**Q: View vs materialized view?**
A regular **view** runs its query each time it's accessed (always fresh, no storage). A **materialized view** stores the result physically (fast reads) but must be **refreshed** to stay current.

**Q: Stored procedure vs function?**
A **function** must return a value and can be used inside a `SELECT`; it usually can't modify data. A **procedure** can perform actions (DML), may return zero or many results, and is called with `CALL`/`EXEC`.

**Q: What is a trigger and a risk of using them?**
Code that fires automatically on data events. Risk: **hidden side effects** that make debugging harder and can hurt performance on bulk operations.

---

## 15. Common Query Programs

The queries interviewers most often ask you to write. *(Assume an `employees` table with `id, name, dept_id, salary, manager_id`.)*

### 1. Second highest salary

```sql
-- Method A: subquery
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Method B: window function (handles Nth easily)
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t WHERE rnk = 2;

-- Method C: LIMIT/OFFSET (MySQL/Postgres)
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC LIMIT 1 OFFSET 1;
```

### 2. Nth highest salary (N = 3)

```sql
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;          -- OFFSET = N-1
```

### 3. Find duplicate records

```sql
SELECT name, COUNT(*) AS cnt
FROM employees
GROUP BY name
HAVING COUNT(*) > 1;
```

### 4. Delete duplicate rows (keep one)

```sql
DELETE FROM employees
WHERE id NOT IN (
    SELECT MIN(id) FROM employees GROUP BY name
);
```

### 5. Employees earning more than their manager

```sql
SELECT e.name
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

### 6. Top earner per department

```sql
SELECT name, dept_id, salary FROM (
    SELECT *, RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM employees
) t WHERE rnk = 1;
```

### 7. Count employees per department (incl. zero)

```sql
SELECT d.dept_name, COUNT(e.id) AS headcount
FROM departments d
LEFT JOIN employees e ON e.dept_id = d.id
GROUP BY d.dept_name;
```

### 8. Employees with no manager

```sql
SELECT name FROM employees WHERE manager_id IS NULL;
```

### 9. Department-wise total & average salary

```sql
SELECT dept_id, SUM(salary) AS total, AVG(salary) AS avg_sal
FROM employees
GROUP BY dept_id;
```

### 10. Running total / cumulative sum

```sql
SELECT name, salary,
       SUM(salary) OVER (ORDER BY id) AS running_total
FROM employees;
```

### 11. Odd / even rows

```sql
-- Even ids
SELECT * FROM employees WHERE id % 2 = 0;
```

### 12. Swap two column values (e.g. gender 'm' <-> 'f')

```sql
UPDATE employees
SET gender = CASE gender WHEN 'm' THEN 'f' WHEN 'f' THEN 'm' END;
```

### 13. Find Nth record / pagination

```sql
SELECT * FROM employees
ORDER BY id
LIMIT 10 OFFSET 20;        -- page 3, 10 per page
```

---

## 16. Tricky / Gotcha Questions

Know the *why* — these trip people up.

**1. `NULL` comparisons.**
`WHERE col = NULL` returns **nothing**. Use `IS NULL`. Also `NULL = NULL` is **UNKNOWN**, not TRUE.

**2. `COUNT(column)` skips NULLs.**
`COUNT(*)` counts all rows; `COUNT(bonus)` ignores rows where bonus is NULL. They can differ.

**3. `NOT IN` with NULLs.**
If the subquery returns any NULL, `WHERE x NOT IN (subquery)` returns **no rows** (because comparisons go UNKNOWN). Prefer `NOT EXISTS`.

**4. WHERE vs HAVING with aggregates.**
You cannot filter an aggregate in `WHERE`. `WHERE COUNT(*) > 1` is invalid — use `HAVING`.

**5. Integer division.**
In some databases `5/2` = `2` (integer division). Cast to decimal: `5.0/2` = `2.5`.

**6. `UNION` removes duplicates (and sorts).**
If you don't want the dedup overhead, use `UNION ALL`.

**7. LEFT JOIN filter trap.**
Putting a right-table condition in `WHERE` (not `ON`) turns a LEFT JOIN into an effective INNER JOIN, because NULL rows fail the filter.

**8. `DISTINCT` applies to the whole row.**
`SELECT DISTINCT a, b` dedups on the **combination** of a and b, not just a.

**9. Aggregate ignores rows but `AVG` divides by non-NULL count.**
`AVG(col)` = `SUM(col)/COUNT(col)`, ignoring NULLs — not divided by total rows.

**10. `TRUNCATE` can't be rolled back (usually) and resets identity** — unlike `DELETE`.

---

## 17. Rapid-Fire One-Liners

- **DDL** changes structure; **DML** changes data; **DQL** queries; **DCL** permissions; **TCL** transactions.
- **PRIMARY KEY** = unique + not null + one per table; **UNIQUE** = allows one NULL, many per table.
- **WHERE** filters rows (pre-group); **HAVING** filters groups (post-aggregate).
- **INNER** = matches only; **LEFT** = all left + matches; **FULL** = everything.
- **UNION** dedups; **UNION ALL** keeps duplicates (faster).
- **`IS NULL`**, never `= NULL`.
- **`COALESCE`** = first non-NULL; **`NULLIF(a,b)`** = NULL when equal.
- **Index** = faster reads, slower writes; **clustered** = one, **non-clustered** = many.
- **`ROW_NUMBER`** unique, **`RANK`** skips ties, **`DENSE_RANK`** no gaps.
- **ACID** = Atomicity, Consistency, Isolation, Durability.
- **`DELETE`** = rows + rollback; **`TRUNCATE`** = all rows, fast, reset; **`DROP`** = whole table.
- **`CHAR`** fixed length, **`VARCHAR`** variable length.
- **Subquery** can be correlated (per-row) or not (once).
- **CTE** = `WITH`, single-statement temp result, supports recursion.
- **View** = stored query (no data); **materialized view** stores results.
- **Stored procedure** = precompiled reusable SQL; **trigger** = auto-runs on events.
- **`GROUP BY`** collapses; **window functions** keep rows.
- **`LIMIT n OFFSET m`** for pagination (SQL Server: `OFFSET ... FETCH`).
- **Execution order:** FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT.
- **`EXISTS`** stops at first match; often faster than `IN` for big subqueries.

---

## 18. Last-Minute Checklist

Run through these the hour before your interview:

- [ ] DDL/DML/DQL/DCL/TCL and DELETE vs TRUNCATE vs DROP.
- [ ] Primary vs unique vs foreign vs composite key.
- [ ] WHERE vs HAVING; logical execution order.
- [ ] All join types + self join + anti-join (find non-matching rows).
- [ ] GROUP BY + aggregates; COUNT(*) vs COUNT(col).
- [ ] Subquery (correlated vs not) and CTE syntax.
- [ ] NULL handling: `IS NULL`, `COALESCE`, NOT IN trap.
- [ ] Window functions: ROW_NUMBER/RANK/DENSE_RANK, PARTITION BY, running total.
- [ ] Indexing: clustered vs non-clustered, when (not) to index, why an index is skipped.
- [ ] Normalization 1NF/2NF/3NF + denormalization trade-off.
- [ ] ACID, transactions, isolation levels, deadlocks.
- [ ] Views, stored procedures, triggers.
- [ ] Be ready to write: 2nd/Nth highest salary, find/delete duplicates, top per group, running total, employees > manager.

**Interview tips:** Restate the question and confirm the schema, think aloud, start simple then refine, prefer `JOIN`s over correlated subqueries when performance matters, handle NULLs and ties explicitly, and mention indexing when asked about performance.

---

*Good luck, Tejas — read it top-to-bottom once, then drill Section 15 (write each query from memory) and Section 16 (gotchas). Those win interviews.*
