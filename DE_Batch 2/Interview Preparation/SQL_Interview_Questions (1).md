# SQL Interview Questions for Data Engineering 

## Sample Schema (used throughout this guide)

```sql
CREATE TABLE Employees (
    EmpID INT PRIMARY KEY,
    EmpName VARCHAR(50),
    DeptID INT,
    Salary DECIMAL(10,2),
    ManagerID INT,
    HireDate DATE
);

CREATE TABLE Departments (
    DeptID INT PRIMARY KEY,
    DeptName VARCHAR(50)
);
```

\---

## PART 1: BASICS (Conceptual)

### 1\. What is SQL, and what is a relational database?

SQL (Structured Query Language) is used to define, query, and manipulate data in a relational database — a database that organizes data into tables with rows and columns, related to each other through keys.

### 2\. What are the different types of SQL commands?

|Category|Purpose|Examples|
|-|-|-|
|DDL (Data Definition)|Defines structure|`CREATE`, `ALTER`, `DROP`, `TRUNCATE`|
|DML (Data Manipulation)|Modifies data|`INSERT`, `UPDATE`, `DELETE`|
|DQL (Data Query)|Retrieves data|`SELECT`|
|DCL (Data Control)|Permissions|`GRANT`, `REVOKE`|
|TCL (Transaction Control)|Manage transactions|`COMMIT`, `ROLLBACK`, `SAVEPOINT`|

### 3\. Difference between `DELETE`, `TRUNCATE`, and `DROP`

* **DELETE**: removes rows (can use `WHERE`), logged row-by-row, can be rolled back, triggers fire.
* **TRUNCATE**: removes all rows, minimally logged, faster, resets auto-increment, cannot use `WHERE`.
* **DROP**: removes the entire table structure + data — irreversible.

### 4\. Primary Key vs Foreign Key vs Unique Key?

* **Primary Key**: uniquely identifies each row; no NULLs; one per table.
* **Foreign Key**: references a Primary/Unique Key in another table to enforce referential integrity.
* **Unique Key**: enforces uniqueness but allows NULL(s); a table can have multiple.

### 5\. What are constraints in SQL?

`NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT` — rules that maintain data integrity.

### 6\. What is normalization? Name the normal forms.

Organizing data to reduce redundancy and dependency issues.

* **1NF**: atomic values, no repeating groups.
* **2NF**: 1NF + no partial dependency on a composite key.
* **3NF**: 2NF + no transitive dependency (non-key columns depend only on the key).

### 7\. What is denormalization, and when would a data engineer use it?

Intentionally introducing redundancy to improve read performance — common in data warehouses/star schemas where joins are expensive and read-heavy analytics matter more than write efficiency.

### 8\. Common data types across SQL databases

* Numeric: `INT`, `BIGINT`, `DECIMAL`/`NUMERIC`, `FLOAT`
* String: `CHAR`, `VARCHAR`, `TEXT`
* Date/Time: `DATE`, `DATETIME`/`TIMESTAMP`
* Other: `BOOLEAN`, `UUID`/`GUID`, `JSON`

### 9\. `CHAR` vs `VARCHAR`?

`CHAR(n)` is fixed length, padded with spaces. `VARCHAR(n)` is variable length, more storage-efficient — used for most text fields.

### 10\. What is NULL? How is it different from 0 or blank?

NULL means "unknown/missing," not zero or empty string. Any arithmetic or equality comparison with NULL returns NULL/unknown (use `IS NULL` / `IS NOT NULL`, never `= NULL`).

### 11\. Auto-incrementing primary keys across databases (good to know for interviews)

|Database|Syntax|
|-|-|
|MySQL|`AUTO\_INCREMENT`|
|PostgreSQL|`SERIAL` / `GENERATED ALWAYS AS IDENTITY`|
|SQL Server|`IDENTITY(1,1)`|
|Oracle|`GENERATED AS IDENTITY` / sequences|

\---

## PART 2: BASIC QUERY WRITING

**Q1. Get all employees with salary greater than 50000.**

```sql
SELECT \* FROM Employees WHERE Salary > 50000;
```

**Q2. Get employee names in uppercase, sorted by salary descending.**

```sql
SELECT UPPER(EmpName) AS EmpName, Salary
FROM Employees
ORDER BY Salary DESC;
```

**Q3. Find employees hired in the year 2023.**

```sql
SELECT \* FROM Employees
WHERE HireDate BETWEEN '2023-01-01' AND '2023-12-31';
-- or, portably: WHERE EXTRACT(YEAR FROM HireDate) = 2023
```

**Q4. Count the number of employees in each department.**

```sql
SELECT DeptID, COUNT(\*) AS EmpCount
FROM Employees
GROUP BY DeptID;
```

**Q5. Find departments having more than 5 employees.**

```sql
SELECT DeptID, COUNT(\*) AS EmpCount
FROM Employees
GROUP BY DeptID
HAVING COUNT(\*) > 5;
```

> `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation — a classic interview distinction.

**Q6. Get the top 3 highest paid employees.**

```sql
-- Standard SQL (MySQL, PostgreSQL)
SELECT \* FROM Employees
ORDER BY Salary DESC
LIMIT 3;

-- SQL Server
SELECT TOP 3 \* FROM Employees ORDER BY Salary DESC;

-- Oracle (12c+)
SELECT \* FROM Employees ORDER BY Salary DESC FETCH FIRST 3 ROWS ONLY;
```

**Q7. Find employees whose names start with 'A'.**

```sql
SELECT \* FROM Employees WHERE EmpName LIKE 'A%';
```

**Q8. Get distinct department IDs.**

```sql
SELECT DISTINCT DeptID FROM Employees;
```

**Q9. Find employees with no manager (top-level employees).**

```sql
SELECT \* FROM Employees WHERE ManagerID IS NULL;
```

\---

## PART 3: JOINS

### Conceptual

* **INNER JOIN**: only matching rows in both tables.
* **LEFT JOIN**: all rows from left table + matched rows from right (NULLs if no match).
* **RIGHT JOIN**: mirror of LEFT JOIN.
* **FULL OUTER JOIN**: all rows from both, matched where possible (MySQL lacks this natively — emulate with `UNION` of LEFT and RIGHT JOIN).
* **CROSS JOIN**: Cartesian product — every row from A paired with every row from B.
* **SELF JOIN**: a table joined with itself (e.g., employee-manager relationship).

**Q10. List each employee with their department name.**

```sql
SELECT E.EmpName, D.DeptName
FROM Employees E
INNER JOIN Departments D ON E.DeptID = D.DeptID;
```

**Q11. List all departments, even those with zero employees.**

```sql
SELECT D.DeptName, E.EmpName
FROM Departments D
LEFT JOIN Employees E ON D.DeptID = E.DeptID;
```

**Q12. Self join — list each employee along with their manager's name.**

```sql
SELECT E.EmpName AS Employee, M.EmpName AS Manager
FROM Employees E
LEFT JOIN Employees M ON E.ManagerID = M.EmpID;
```

**Q13. Find departments that have no employees.**

```sql
SELECT D.DeptName
FROM Departments D
LEFT JOIN Employees E ON D.DeptID = E.DeptID
WHERE E.EmpID IS NULL;
```

\---

## PART 4: SUBQUERIES \& SET OPERATIONS

**Q14. Find employees who earn more than the average salary.**

```sql
SELECT \* FROM Employees
WHERE Salary > (SELECT AVG(Salary) FROM Employees);
```

**Q15. Find the department with the highest total salary payout.**

```sql
SELECT DeptID, SUM(Salary) AS TotalSalary
FROM Employees
GROUP BY DeptID
ORDER BY TotalSalary DESC
LIMIT 1;
```

**Q16. Find the second highest salary (classic interview question — multiple portable approaches).**

```sql
-- Approach 1: OFFSET (MySQL / PostgreSQL)
SELECT DISTINCT Salary
FROM Employees
ORDER BY Salary DESC
LIMIT 1 OFFSET 1;

-- Approach 2: Subquery (works everywhere)
SELECT MAX(Salary) AS SecondHighest
FROM Employees
WHERE Salary < (SELECT MAX(Salary) FROM Employees);

-- Approach 3: DENSE\_RANK (handles ties correctly, standard ANSI window function)
SELECT Salary FROM (
    SELECT Salary, DENSE\_RANK() OVER (ORDER BY Salary DESC) AS rnk
    FROM Employees
) t WHERE rnk = 2;
```

**Q17. `UNION` vs `UNION ALL`?**
`UNION` removes duplicates (implicit sort/distinct — slower); `UNION ALL` keeps all rows including duplicates (faster).

**Q18. `EXISTS` vs `IN` — when to use which?**
`EXISTS` stops at the first match (often faster with correlated subqueries, and handles NULLs safely). `IN` evaluates the full list and misbehaves if the subquery returns NULLs. Prefer `EXISTS` for correlated subqueries or large datasets.

```sql
-- Departments that have at least one employee
SELECT DeptName FROM Departments D
WHERE EXISTS (SELECT 1 FROM Employees E WHERE E.DeptID = D.DeptID);
```

\---

## PART 5: AGGREGATE FUNCTIONS \& GROUPING

**Q19. Find the highest, lowest, and average salary per department.**

```sql
SELECT DeptID,
       MAX(Salary) AS MaxSal,
       MIN(Salary) AS MinSal,
       AVG(Salary) AS AvgSal
FROM Employees
GROUP BY DeptID;
```

**Q20. Difference between `COUNT(\*)`, `COUNT(1)`, and `COUNT(column\_name)`?**
`COUNT(\*)` and `COUNT(1)` both count all rows regardless of NULLs (performance is effectively identical in most modern engines). `COUNT(column\_name)` counts only non-NULL values in that column.

**Q21. What does `GROUP BY` with `ROLLUP` do? (Intermediate)**
Produces subtotal and grand total rows in addition to normal grouped results — useful in reporting. Supported in SQL Server, MySQL 8+, PostgreSQL, Oracle.

```sql
SELECT DeptID, SUM(Salary) AS TotalSalary
FROM Employees
GROUP BY ROLLUP(DeptID);
```

\---

## PART 6: KEYS, INDEXES \& PERFORMANCE (Intermediate)

### 22\. What is an index? Clustered vs Non-clustered?

An index is a data structure that speeds up data retrieval at the cost of extra storage and slower writes.

* **Clustered Index**: determines the physical order of data in the table; only one per table (typically the Primary Key). In MySQL/InnoDB, the primary key *is* the clustered index by default.
* **Non-clustered Index (secondary index)**: a separate structure with pointers back to the data; a table can have many.

### 23\. When would you avoid creating too many indexes?

Indexes speed up `SELECT` but slow down `INSERT`/`UPDATE`/`DELETE` because each index must also be updated. Excess indexes add storage overhead too.

### 24\. What is a composite (multi-column) index?

An index on multiple columns; column order matters — the index is most effective when queries filter on the leading column(s), left to right.

### 25\. How would you debug a slow query? (Common practical question)

* Check the **execution plan** (`EXPLAIN` in MySQL/PostgreSQL, `EXPLAIN PLAN` in Oracle, graphical plan in SSMS).
* Look for full table scans instead of index seeks/lookups.
* Check for missing indexes, stale statistics, or implicit type conversions in `WHERE` clauses.

\---

## PART 7: VIEWS, STORED PROCEDURES, FUNCTIONS, TRIGGERS

### 26\. What is a View?

A virtual table based on a stored `SELECT` query; doesn't store data itself (unless it's a **materialized view**, supported in PostgreSQL/Oracle).

```sql
CREATE VIEW vw\_HighEarners AS
SELECT EmpName, Salary FROM Employees WHERE Salary > 80000;
```

### 27\. What is a Stored Procedure? Why use one?

A precompiled, reusable batch of SQL/procedural code. Benefits: performance (cached execution plan in some engines), security (grant execute rights without table access), reduced network round-trips, centralized business logic.

```sql
-- Generic example (syntax varies by engine)
CREATE PROCEDURE GetEmployeesByDept (IN dept\_id INT)
BEGIN
    SELECT \* FROM Employees WHERE DeptID = dept\_id;
END;
```

### 28\. Stored Procedure vs Function — key differences?

|Stored Procedure|Function|
|-|-|
|Can perform DML (INSERT/UPDATE/DELETE)|Generally read-only|
|Cannot be used inline in a `SELECT`|Can be used inline in `SELECT`/`WHERE`|
|Can return multiple result sets / output params|Must return a single value or table|
|Called with `CALL`/`EXEC`|Called within an expression|

### 29\. What is a Trigger? Give an example use case.

Code that automatically executes in response to an `INSERT`/`UPDATE`/`DELETE` event. Example: an **audit trigger** that logs changes to a history table whenever a salary is updated.

\---

## PART 8: WINDOW FUNCTIONS (Standard ANSI SQL — very common in data engineering interviews)

### 30\. What are window functions, and why are they important for data engineering?

They perform calculations across a set of rows related to the current row **without collapsing rows** (unlike `GROUP BY`). Essential for ranking, running totals, deduplication. Supported across MySQL 8+, PostgreSQL, SQL Server, Oracle, Snowflake, BigQuery, etc.

**Q31. Assign a rank to employees by salary within each department.**

```sql
SELECT EmpName, DeptID, Salary,
       RANK() OVER (PARTITION BY DeptID ORDER BY Salary DESC) AS SalaryRank
FROM Employees;
```

**`RANK` vs `DENSE\_RANK` vs `ROW\_NUMBER`?**

* `ROW\_NUMBER()`: unique sequential number, no ties.
* `RANK()`: same rank for ties, but skips subsequent ranks (1,1,3).
* `DENSE\_RANK()`: same rank for ties, no skipping (1,1,2).

**Q32. Remove duplicate rows, keeping only the latest record (common ETL/data-cleaning task).**

```sql
WITH CTE AS (
    SELECT \*, ROW\_NUMBER() OVER (PARTITION BY EmpID ORDER BY HireDate DESC) AS rn
    FROM Employees
)
DELETE FROM Employees
WHERE EmpID IN (SELECT EmpID FROM CTE WHERE rn > 1);
-- Note: SQL Server allows DELETE directly from the CTE; MySQL/PostgreSQL often need
-- the pattern above or a supported "USING"/subquery form.
```

**Q33. Running total of salaries ordered by hire date.**

```sql
SELECT EmpName, HireDate, Salary,
       SUM(Salary) OVER (ORDER BY HireDate
                          ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS RunningTotal
FROM Employees;
```

**Q34. `LEAD()` and `LAG()` — compare each employee's salary to the previous hire's salary.**

```sql
SELECT EmpName, HireDate, Salary,
       LAG(Salary) OVER (ORDER BY HireDate) AS PrevSalary
FROM Employees;
```

\---

## PART 9: CTEs, TEMP TABLES \& SUBQUERIES (Intermediate)

### 35\. What is a CTE (Common Table Expression)? How is it different from a subquery?

A CTE (`WITH` clause) is a named temporary result set, readable and reusable within the same query — improves readability over nested subqueries, and supports **recursion**.

**Q36. Recursive CTE — find the entire management chain for an employee (org hierarchy).**

```sql
WITH RECURSIVE EmpHierarchy AS (   -- SQL Server/Oracle: omit "RECURSIVE" keyword
    SELECT EmpID, EmpName, ManagerID, 1 AS Level
    FROM Employees
    WHERE ManagerID IS NULL

    UNION ALL

    SELECT E.EmpID, E.EmpName, E.ManagerID, EH.Level + 1
    FROM Employees E
    INNER JOIN EmpHierarchy EH ON E.ManagerID = EH.EmpID
)
SELECT \* FROM EmpHierarchy;
```

### 37\. Temp Table vs CTE vs Derived Table (subquery)?

||Temp Table|CTE|Derived Table (subquery)|
|-|-|-|-|
|Scope|Session/connection|Single query|Single query|
|Can be indexed|Yes|No|No|
|Reused multiple times|Yes|Within same query only|No, must repeat|
|Supports recursion|No|Yes|No|

\---

## PART 10: TRANSACTIONS \& ISOLATION (Intermediate)

### 38\. What is a transaction? What are the ACID properties?

A transaction is a unit of work that's all-or-nothing.

* **Atomicity**: all steps succeed or none do.
* **Consistency**: database moves from one valid state to another.
* **Isolation**: concurrent transactions don't interfere.
* **Durability**: once committed, changes persist even after a crash.

```sql
BEGIN TRANSACTION;
    UPDATE Employees SET Salary = Salary \* 1.1 WHERE DeptID = 1;
COMMIT;
-- or ROLLBACK; if something goes wrong
```

### 39\. What are isolation levels?

`READ UNCOMMITTED`, `READ COMMITTED` (default in most engines), `REPEATABLE READ`, `SERIALIZABLE` — trade off consistency vs concurrency, addressing dirty reads, non-repeatable reads, and phantom reads.

### 40\. What is a deadlock? How can it be avoided?

Two transactions each waiting on a resource the other holds, so neither can proceed. The database detects and kills one (the "deadlock victim"). Avoid by accessing tables/resources in a consistent order, keeping transactions short, and using appropriate indexing.

\---

## PART 11: DATA ENGINEERING-SPECIFIC BASICS

### 41\. What is ETL/ELT?

Extract, Transform, Load (or Extract, Load, Transform) — moving data from source systems into a destination, often a data warehouse, with transformation logic applied either before or after loading.

### 42\. How do you handle incremental data loads? (Common practical question)

Common patterns: watermark columns (`LastModifiedDate`), Change Data Capture (CDC), or `MERGE`/`UPSERT` statements to update only new/changed rows instead of reloading everything.

**Q43. Write a query to upsert (insert-or-update) data from a staging table into a target table.**

```sql
-- ANSI SQL / SQL Server / Oracle: MERGE
MERGE INTO Employees AS Target
USING Staging\_Employees AS Source
ON Target.EmpID = Source.EmpID
WHEN MATCHED THEN
    UPDATE SET Target.Salary = Source.Salary, Target.DeptID = Source.DeptID
WHEN NOT MATCHED THEN
    INSERT (EmpID, EmpName, DeptID, Salary, HireDate)
    VALUES (Source.EmpID, Source.EmpName, Source.DeptID, Source.Salary, Source.HireDate);

-- PostgreSQL equivalent
INSERT INTO Employees (EmpID, EmpName, DeptID, Salary, HireDate)
SELECT EmpID, EmpName, DeptID, Salary, HireDate FROM Staging\_Employees
ON CONFLICT (EmpID) DO UPDATE
SET Salary = EXCLUDED.Salary, DeptID = EXCLUDED.DeptID;

-- MySQL equivalent
INSERT INTO Employees (EmpID, EmpName, DeptID, Salary, HireDate)
SELECT EmpID, EmpName, DeptID, Salary, HireDate FROM Staging\_Employees
ON DUPLICATE KEY UPDATE Salary = VALUES(Salary), DeptID = VALUES(DeptID);
```

### 44\. Star schema vs Snowflake schema (data warehousing basics)?

* **Star schema**: central fact table connected directly to denormalized dimension tables — simpler, faster reads.
* **Snowflake schema**: dimension tables are further normalized into sub-dimensions — saves storage, more joins needed.

### 45\. What is a surrogate key vs a natural key?

A **natural key** is a business-meaningful identifier already in the data (e.g., email, SSN). A **surrogate key** is a system-generated, meaningless identifier (e.g., auto-increment integer) — preferred in warehouses because it's stable even if business attributes change.

\---

## PART 12: MIXED PRACTICE QUERIES (Try these yourself before checking the answer)

**Q46.** Find employees who work in a department that doesn't exist in the Departments table (data quality check).

```sql
SELECT E.\* FROM Employees E
LEFT JOIN Departments D ON E.DeptID = D.DeptID
WHERE D.DeptID IS NULL;
```

**Q47.** Get the count of employees hired each year.

```sql
SELECT EXTRACT(YEAR FROM HireDate) AS HireYear, COUNT(\*) AS TotalHires
FROM Employees
GROUP BY EXTRACT(YEAR FROM HireDate)
ORDER BY HireYear;
```

**Q48.** Find the 3 most recently hired employees in each department (window function + filtering).

```sql
WITH Ranked AS (
    SELECT \*, ROW\_NUMBER() OVER (PARTITION BY DeptID ORDER BY HireDate DESC) AS rn
    FROM Employees
)
SELECT \* FROM Ranked WHERE rn <= 3;
```

**Q49.** Find employees who earn more than their department's average salary.

```sql
SELECT E.EmpName, E.Salary, D.DeptName
FROM Employees E
JOIN Departments D ON E.DeptID = D.DeptID
WHERE E.Salary > (
    SELECT AVG(Salary) FROM Employees WHERE DeptID = E.DeptID
);
```

**Q50.** Find the number of employees who report to each manager.

```sql
SELECT ManagerID, COUNT(\*) AS DirectReports
FROM Employees
WHERE ManagerID IS NOT NULL
GROUP BY ManagerID;
```



