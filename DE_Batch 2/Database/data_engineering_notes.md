#  Data Engineering — Complete Notes

> **Course:** Data Engineering Fundamentals  
> **Prepared by:** AkumenbyQ  
> **Last Updated:** June 2026  
> **Audience:** Students / Beginners to Intermediate  

---

##  Table of Contents

1. [Database Management Systems (DBMS)](#1-database-management-systems-dbms)
2. [Types of Databases — RDBMS vs NoSQL](#2-types-of-databases--rdbms-vs-nosql)
3. [ACID Properties](#3-acid-properties)
4. [Entity-Relationship (ER) Modelling](#4-entity-relationship-er-modelling)
5. [Normalization & Denormalization](#5-normalization--denormalization)
6. [Data Warehousing](#6-data-warehousing)
7. [OLTP vs OLAP](#7-oltp-vs-olap)
8. [Data Warehousing Architecture](#8-data-warehousing-architecture)
9. [Facts and Dimensions](#9-facts-and-dimensions)
10. [Star and Snowflake Schema](#10-star-and-snowflake-schema)
11. [Slowly Changing Dimensions (SCDs)](#11-slowly-changing-dimensions-scds)
12. [Indexing & Clustering](#12-indexing--clustering)
13. [Sharding](#13-sharding)
14. [Query Optimization](#14-query-optimization)

---

## 1. Database Management Systems (DBMS)

### 1.1 What is a DBMS?

A **Database Management System (DBMS)** is software that provides a systematic way to create, retrieve, update, and manage data. It acts as an intermediary between the user and the database.

```
User / Application
       │
       ▼
  ┌─────────┐
  │  DBMS   │  ← Query processing, security, concurrency control
  └─────────┘
       │
       ▼
  ┌─────────┐
  │  DATA   │  ← Physical storage (disk / memory)
  └─────────┘
```

### 1.2 Why DBMS over File Systems?

| Problem with File Systems | DBMS Solution |
|---|---|
| Data redundancy (same data in multiple files) | Centralized storage, single source of truth |
| Data inconsistency | Enforces integrity constraints |
| Difficult concurrent access | Concurrency control (locking, MVCC) |
| Security is hard to enforce | Role-based access control (RBAC) |
| No crash recovery | Transaction logs and recovery mechanisms |
| Hard to query complex data | SQL and query engines |

### 1.3 Core Functions of a DBMS

- **Data Definition** — Create, alter, and drop schema objects (DDL)
- **Data Manipulation** — Insert, update, delete, query data (DML)
- **Data Control** — Grant/revoke permissions (DCL)
- **Transaction Management** — ACID compliance
- **Concurrency Control** — Multiple users accessing simultaneously
- **Recovery Management** — Restore data after crashes
- **Data Integrity** — Constraints (PK, FK, NOT NULL, UNIQUE, CHECK)
- **Security** — Authentication and authorization

### 1.4 DBMS Architecture (3-Tier / ANSI-SPARC)

```
┌──────────────────────────────────┐
│    EXTERNAL LEVEL (View Level)   │  ← What individual users see
│    User View 1 | User View 2     │
├──────────────────────────────────┤
│  CONCEPTUAL LEVEL (Logical)      │  ← Overall logical structure (schema)
│  Tables, Relationships           │
├──────────────────────────────────┤
│    INTERNAL LEVEL (Physical)     │  ← How data is physically stored
│    Indexes, File Blocks          │
└──────────────────────────────────┘
```

This separation is called **data independence**:
- **Logical independence** — Change the conceptual schema without affecting external views
- **Physical independence** — Change storage without affecting the logical schema

### 1.5 Key Terminology

| Term | Definition |
|---|---|
| **Schema** | The structure/blueprint of a database (table definitions, data types) |
| **Instance** | The actual data in the database at a point in time |
| **Metadata** | Data about data (column names, data types, constraints) |
| **DDL** | Data Definition Language — `CREATE`, `ALTER`, `DROP` |
| **DML** | Data Manipulation Language — `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** | Data Control Language — `GRANT`, `REVOKE` |
| **TCL** | Transaction Control Language — `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

---

## 2. Types of Databases — RDBMS vs NoSQL

### 2.1 Relational Databases (RDBMS)

Data is organized into **tables (relations)** with rows and columns. Tables are linked via **foreign keys**.

**Examples:** MySQL, PostgreSQL, Oracle, SQL Server, SQLite

```sql
-- Example: Two related tables
CREATE TABLE customers (
    customer_id   INT PRIMARY KEY,
    name          VARCHAR(100),
    email         VARCHAR(100) UNIQUE
);

CREATE TABLE orders (
    order_id      INT PRIMARY KEY,
    customer_id   INT REFERENCES customers(customer_id),
    order_date    DATE,
    total_amount  DECIMAL(10,2)
);
```

**Strengths:**
- Structured data with clear relationships
- ACID compliance out of the box
- Powerful query language (SQL)
- Mature ecosystem and tooling
- Strong consistency

**Weaknesses:**
- Rigid schema — changes are expensive
- Vertical scaling is hard (scale up, not out)
- Poor fit for unstructured / semi-structured data
- Performance degrades with very large JOINs

---

### 2.2 NoSQL Databases

"**Not Only SQL**" — designed for flexibility, scale, and varied data models. Four main types:

#### 2.2.1 Key-Value Stores

Data stored as simple key → value pairs. Extremely fast lookups.

```
Key         Value
─────────── ──────────────────────────────
"user:1001" { "name": "Priya", "age": 28 }
"session:x" "eyJhbGciOiJIUzI1NiJ9..."
```

**Use cases:** Session management, caching, real-time leaderboards  
**Examples:** Redis, DynamoDB, Riak

---

#### 2.2.2 Document Stores

Data stored as self-describing documents (JSON, BSON, XML). Flexible schema.

```json
{
  "_id": "order_001",
  "customer": { "name": "Priya", "email": "priya@example.com" },
  "items": [
    { "product": "Laptop", "qty": 1, "price": 75000 },
    { "product": "Mouse",  "qty": 2, "price": 500   }
  ],
  "total": 76000,
  "status": "shipped"
}
```

**Use cases:** Catalogs, CMS, user profiles, event logging  
**Examples:** MongoDB, CouchDB, Firestore

---

#### 2.2.3 Column-Family / Wide-Column Stores

Data organized by columns rather than rows. Efficient for analytical queries on sparse data.

```
Row Key    │ Column Family: profile     │ Column Family: activity
───────────┼────────────────────────────┼─────────────────────────
"user:1"   │ name="Arjun" city="Kochi"  │ last_login="2026-06-01"
"user:2"   │ name="Nisha"               │ last_login="2026-05-20"
```

**Use cases:** Time-series data, IoT telemetry, write-heavy analytics  
**Examples:** Apache Cassandra, HBase, Google Bigtable

---

#### 2.2.4 Graph Databases

Data stored as **nodes** (entities) and **edges** (relationships). Best for highly connected data.

```
(Priya) ──FOLLOWS──▶ (Arjun) ──FOLLOWS──▶ (Nisha)
  │                                            │
  └──────────LIKES──────────────────────────▶(Post)
```

**Use cases:** Social networks, fraud detection, recommendation engines, knowledge graphs  
**Examples:** Neo4j, Amazon Neptune, ArangoDB

---

### 2.3 RDBMS vs NoSQL — Comparison Table

| Feature | RDBMS | NoSQL |
|---|---|---|
| **Data Model** | Tables with rows/columns | Key-value, document, column, graph |
| **Schema** | Fixed (strict) | Flexible (schema-on-read) |
| **Query Language** | SQL (standardized) | Varies by DB (some support SQL-like) |
| **ACID** | Full ACID | Typically BASE (Basically Available, Soft state, Eventually consistent) |
| **Scaling** | Vertical (scale up) | Horizontal (scale out) |
| **Consistency** | Strong | Eventual (tunable in some) |
| **Joins** | Native, efficient | Not natively supported (denormalized) |
| **Best For** | Structured data, complex queries, transactions | Large-scale, unstructured, high-velocity data |

### 2.4 CAP Theorem (Brief)

Any distributed data store can guarantee only **2 of 3**:

```
           Consistency
               /\
              /  \
             /    \
            /      \
      Availability──Partition Tolerance
```

- **CA:** Traditional RDBMS (PostgreSQL, MySQL) — consistent and available, but no partition tolerance
- **CP:** HBase, MongoDB (strong mode) — consistent under partition, may sacrifice availability
- **AP:** Cassandra, CouchDB — always available under partition, eventually consistent

---

## 3. ACID Properties

ACID is the set of properties that guarantee **reliable database transactions**, especially in the event of errors, power failures, or concurrent access.

### 3.1 Atomicity

> **"All or nothing"** — a transaction either completes fully or has no effect at all.

```
Transaction: Transfer ₹5000 from Account A to Account B

Step 1: Debit ₹5000 from A   ✓
Step 2: Credit ₹5000 to B    ✗ ← FAILURE HERE

Without Atomicity:  A loses ₹5000, B gets nothing  ❌
With Atomicity:     Both steps rolled back, no change ✓
```

**Mechanism:** Write-Ahead Log (WAL) / Undo logs — the DBMS records what it's about to do before doing it, so it can undo on failure.

---

### 3.2 Consistency

> **"Data must always be in a valid state"** — every transaction takes the database from one consistent state to another, respecting all defined rules and constraints.

```
Rule: account_balance >= 0

Before: A = ₹10,000  (valid)
Tx:     Withdraw ₹15,000
After:  A = -₹5,000  (INVALID — violates constraint)

With Consistency: Transaction is rejected ✓
```

Enforced through:
- **Primary key** constraints
- **Foreign key** constraints
- **CHECK** constraints
- **NOT NULL** constraints
- **Application-level** business rules

---

### 3.3 Isolation

> **"Concurrent transactions don't interfere with each other"** — intermediate states of a transaction are invisible to other transactions.

**Problem without Isolation — Dirty Read:**
```
Time  Transaction T1                Transaction T2
────  ─────────────────────────     ──────────────────────
T1    UPDATE balance = 5000
T2                                  READ balance → 5000  ← reads uncommitted!
T3    ROLLBACK (balance → 3000)
T4                                  Uses 5000 (WRONG ❌)
```

**Isolation Levels (from least to most strict):**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| **Read Uncommitted** | ✅ Possible | ✅ Possible | ✅ Possible |
| **Read Committed** | ❌ Prevented | ✅ Possible | ✅ Possible |
| **Repeatable Read** | ❌ Prevented | ❌ Prevented |  Possible |
| **Serializable** | ❌ Prevented | ❌ Prevented | ❌ Prevented |

**Definitions:**
- **Dirty Read:** Reading uncommitted changes from another transaction
- **Non-Repeatable Read:** Same row returns different value when read twice in the same transaction
- **Phantom Read:** Same query returns different set of rows when run twice (due to inserts/deletes by another transaction)

---

### 3.4 Durability

> **"Committed data survives failures"** — once a transaction is committed, it stays committed even after a crash, power loss, or error.

**Mechanism:**
- **Write-Ahead Logging (WAL):** Changes are written to a log file on disk *before* they're applied to the actual data files
- **Checkpointing:** Periodically flushing dirty pages from memory to disk
- **Replication:** Keeping copies of data on multiple nodes

```
COMMIT ──▶ WAL written to disk ──▶ OK returned to user
                                      │
               (crash here is fine — WAL can replay the commit on restart)
```

### 3.5 ACID in Practice

```sql
BEGIN TRANSACTION;

  UPDATE accounts SET balance = balance - 5000 WHERE id = 'A';  -- Atomicity
  UPDATE accounts SET balance = balance + 5000 WHERE id = 'B';  -- Atomicity

  -- Consistency: CHECK constraints fire here
  -- Isolation: Other transactions can't see these changes yet

COMMIT;  -- Durability: now written to WAL and permanent
```

---

## 4. Entity-Relationship (ER) Modelling

### 4.1 What is ER Modelling?

An **Entity-Relationship (ER) Model** is a conceptual blueprint of a database. It describes the **entities** (things), their **attributes** (properties), and the **relationships** between them — before any SQL is written.

It was introduced by **Peter Chen in 1976** and is the standard tool for database design.

---

### 4.2 Core Components

#### Entities
An **entity** is a real-world object or concept that has data stored about it.

```
Strong Entity:    [ STUDENT ]    ← has its own primary key
Weak Entity:     [[ ENROLLMENT ]]  ← depends on a strong entity for identity
```

#### Attributes

| Type | Description | Example |
|---|---|---|
| **Simple** | Atomic, indivisible | `age`, `student_id` |
| **Composite** | Can be split into parts | `full_name` = first + last |
| **Derived** | Computed from other attributes | `age` derived from `dob` |
| **Multi-valued** | Can have multiple values | `phone_numbers` |
| **Key** | Uniquely identifies an entity | `student_id` |

#### Relationships
A **relationship** describes how two or more entities are associated.

```
[ STUDENT ] ──enrolls in── [ COURSE ]
```

Relationships can also have attributes:
```
[ STUDENT ] ──enrolls in── < ENROLLMENT > ──── [ COURSE ]
                                 │
                              enroll_date
                              grade
```

---

### 4.3 Cardinality (Relationship Ratios)

#### One-to-One (1:1)
```
[ PERSON ] ──────── [ PASSPORT ]
  1 person has exactly 1 passport
  1 passport belongs to exactly 1 person
```

#### One-to-Many (1:N)
```
[ DEPARTMENT ] ──────── [ EMPLOYEE ]
  1 department has many employees
  1 employee belongs to 1 department
```

#### Many-to-Many (M:N)
```
[ STUDENT ] ──────── [ COURSE ]
  1 student enrolls in many courses
  1 course has many students
  → Resolved with a junction table: ENROLLMENT(student_id, course_id)
```

---

### 4.4 ER Diagram Notation (Chen Notation)

```
Rectangle      = Entity
Ellipse        = Attribute
Diamond        = Relationship
Double ellipse = Multi-valued attribute
Dashed ellipse = Derived attribute
Double rect    = Weak entity
Double diamond = Identifying relationship
```

**Example ER Diagram (text form):**
```
       student_id   name     dob (derived: age)
           │          │           │
    ┌──────┴──────────┴───────────┴──┐
    │           STUDENT              │
    └────────────────┬───────────────┘
                     │
               enrolls in  ── (enroll_date, grade)
                     │
    ┌────────────────┴───────────────┐
    │            COURSE              │
    └──────┬──────────┬──────────────┘
           │          │
      course_id    course_name
```

---

### 4.5 ER to Relational Mapping Rules

| ER Construct | Relational Mapping |
|---|---|
| Strong entity | Table with PK |
| Weak entity | Table with PK = (owner's PK + partial key) |
| 1:1 relationship | Add FK to either side; or merge into one table |
| 1:N relationship | Add FK on the "many" side |
| M:N relationship | Create a junction table with FKs to both sides |
| Multi-valued attribute | Create a new table (entity_id, value) |
| Composite attribute | Break into simple columns |
| Derived attribute | Usually not stored; computed in queries |

**M:N Example:**
```sql
-- Students and Courses (many-to-many)
CREATE TABLE students  (student_id INT PRIMARY KEY, name VARCHAR(100));
CREATE TABLE courses   (course_id  INT PRIMARY KEY, title VARCHAR(100));
CREATE TABLE enrollments (
    student_id   INT REFERENCES students(student_id),
    course_id    INT REFERENCES courses(course_id),
    enroll_date  DATE,
    grade        CHAR(2),
    PRIMARY KEY (student_id, course_id)
);
```

---

## 5. Normalization & Denormalization

### 5.1 What is Normalization?

**Normalization** is the process of organizing a relational database to **reduce data redundancy** and **improve data integrity** by decomposing tables into smaller, well-structured ones.

**Goal:** Eliminate anomalies:
- **Insertion anomaly:** Can't insert data without inserting unrelated data
- **Update anomaly:** Changing one fact requires updates in multiple rows
- **Deletion anomaly:** Deleting a row accidentally loses unrelated data

---

### 5.2 Functional Dependency (FD)

A **functional dependency** X → Y means: if you know X, you can uniquely determine Y.

```
student_id → student_name     (knowing the ID gives you the name)
{student_id, course_id} → grade   (knowing both gives you the grade)
```

---

### 5.3 Normal Forms

#### First Normal Form (1NF)

**Rules:**
1. Each column must have **atomic (indivisible) values** — no lists or sets
2. Each column must have a **single data type**
3. Each row must be **uniquely identifiable** (has a PK)

```
❌ NOT 1NF:
┌────────────┬──────────┬───────────────────────────┐
│ student_id │ name     │ phone_numbers             │
├────────────┼──────────┼───────────────────────────┤
│ 1          │ Priya    │ 9876543210, 9123456789    │  ← multi-value!
└────────────┴──────────┴───────────────────────────┘

✅ 1NF:
┌────────────┬──────────┬──────────────┐
│ student_id │ name     │ phone        │
├────────────┼──────────┼──────────────┤
│ 1          │ Priya    │ 9876543210   │
│ 1          │ Priya    │ 9123456789   │
└────────────┴──────────┴──────────────┘
```

---

#### Second Normal Form (2NF)

**Rules:**
1. Must be in **1NF**
2. Every **non-key attribute** must be **fully functionally dependent** on the **entire** primary key (no partial dependencies)

> Only relevant when the PK is composite.

```
❌ NOT 2NF — Partial Dependency:
Table: ORDER_ITEMS(order_id, product_id, quantity, product_name)
PK = (order_id, product_id)

product_name depends only on product_id (part of the PK) → partial dependency!

✅ 2NF — Decompose:
ORDER_ITEMS(order_id, product_id, quantity)   ← quantity needs both keys
PRODUCTS(product_id, product_name)             ← product_name needs only product_id
```

---

#### Third Normal Form (3NF)

**Rules:**
1. Must be in **2NF**
2. No **transitive dependencies** — non-key attributes must not depend on other non-key attributes

```
❌ NOT 3NF — Transitive Dependency:
Table: EMPLOYEES(emp_id, emp_name, dept_id, dept_name)
PK = emp_id

emp_id → dept_id → dept_name   ← dept_name depends on dept_id, not on emp_id directly!

✅ 3NF — Decompose:
EMPLOYEES(emp_id, emp_name, dept_id)
DEPARTMENTS(dept_id, dept_name)
```

---

#### Boyce-Codd Normal Form (BCNF)

**Rules:**
1. Must be in **3NF**
2. For every functional dependency X → Y, X must be a **superkey**

BCNF is stricter than 3NF and handles edge cases where 3NF still allows anomalies (rare in practice).

```
❌ NOT BCNF:
Table: COURSE_TEACHERS(student, course, teacher)
FDs: {student, course} → teacher   AND   teacher → course

teacher → course: teacher is NOT a superkey → violates BCNF

✅ BCNF — Decompose:
TEACHER_COURSE(teacher, course)
STUDENT_TEACHER(student, teacher)
```

---

#### Higher Normal Forms (Brief)

| Form | Key Rule |
|---|---|
| **4NF** | No multi-valued dependencies beyond the key |
| **5NF** | No join dependencies that aren't implied by keys |

> In practice, **3NF or BCNF** is sufficient for most database designs.

---

### 5.4 Normalization Summary

```
Unnormalized Table
       │
       ▼  Remove multi-valued & composite attributes
      1NF
       │
       ▼  Remove partial dependencies (on part of composite PK)
      2NF
       │
       ▼  Remove transitive dependencies
      3NF
       │
       ▼  Every determinant is a superkey
     BCNF
```

---

### 5.5 Denormalization

**Denormalization** is the deliberate **introduction of redundancy** into a database to **improve read performance**, at the cost of some data integrity overhead.

**When to denormalize:**
- Read performance is critical (reports, dashboards)
- Data is mostly read, rarely updated
- Joins are too expensive (large tables, complex queries)
- Working in a Data Warehouse (OLAP) environment

```
Normalized (OLTP-friendly):
  orders ── order_items ── products ── categories
  → Need 4-table JOIN to get "order total with category"

Denormalized (OLAP-friendly):
  order_facts(order_id, customer_name, product_name, category_name, total_amount, order_date)
  → Single table scan, no JOINs
```

**Trade-offs:**

| | Normalization | Denormalization |
|---|---|---|
| Redundancy | Low | High |
| Update performance | Fast (one place) | Slow (multiple copies) |
| Read performance | Slower (JOINs) | Faster (pre-joined) |
| Storage | Less | More |
| Data integrity | Higher | Needs careful management |
| Best for | OLTP | OLAP / Analytics |

---

## 6. Data Warehousing

### 6.1 What is a Data Warehouse?

A **Data Warehouse (DW)** is a centralized repository that stores **integrated, historical, and subject-oriented data** from multiple source systems, designed specifically for **analytical reporting and decision support**.

> Coined and defined by **Bill Inmon** in the 1990s:  
> *"A data warehouse is subject-oriented, integrated, non-volatile, and time-variant collection of data in support of management's decision-making process."*

### 6.2 The 4 Characteristics (Inmon)

| Characteristic | Meaning |
|---|---|
| **Subject-Oriented** | Organized around business subjects (Sales, Finance, HR) not applications |
| **Integrated** | Data from heterogeneous sources is cleaned, transformed, and unified |
| **Non-Volatile** | Data is never deleted/updated; only appended (historical record) |
| **Time-Variant** | Always has a time dimension; stores point-in-time snapshots |

### 6.3 Why Data Warehouses?

**Problems with reporting directly on OLTP systems:**
- Analytics queries (aggregations, historical trends) lock operational tables
- Source systems have different formats, naming, encoding
- No history — OLTP only stores current state
- Impossible to see trends across multiple systems (ERP + CRM + finance)

**Data Warehouse solves this by:**
- Decoupling analytics from operations
- Harmonizing data from all source systems
- Preserving history for trend analysis
- Enabling fast analytical queries via optimized schemas

---

## 7. OLTP vs OLAP

### 7.1 OLTP — Online Transaction Processing

**Purpose:** Day-to-day operational transactions  
**Users:** Clerks, sales reps, customer-facing apps  
**Query type:** Simple lookups and updates on few rows

```sql
-- Typical OLTP query
SELECT * FROM orders WHERE order_id = 10045;

INSERT INTO payments (order_id, amount, method)
VALUES (10045, 2500, 'UPI');
```

### 7.2 OLAP — Online Analytical Processing

**Purpose:** Historical analysis and reporting  
**Users:** Analysts, executives, data scientists  
**Query type:** Aggregations across millions of rows, multiple dimensions

```sql
-- Typical OLAP query
SELECT
    d.year,
    d.quarter,
    p.category,
    SUM(f.sales_amount) AS total_sales,
    COUNT(DISTINCT f.customer_id) AS unique_customers
FROM fact_sales f
JOIN dim_date    d ON f.date_key    = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
WHERE d.year BETWEEN 2022 AND 2026
GROUP BY d.year, d.quarter, p.category
ORDER BY d.year, d.quarter;
```

### 7.3 OLTP vs OLAP — Full Comparison

| Feature | OLTP | OLAP |
|---|---|---|
| **Full Form** | Online Transaction Processing | Online Analytical Processing |
| **Purpose** | Run business operations | Analyze business performance |
| **Data** | Current, operational | Historical, consolidated |
| **Operations** | INSERT, UPDATE, DELETE, SELECT | Mostly SELECT (heavy aggregations) |
| **Query complexity** | Simple, predictable | Complex, ad hoc |
| **Query response** | Milliseconds | Seconds to minutes |
| **Rows per query** | Few (1–100) | Millions to billions |
| **Schema** | Normalized (3NF) | Denormalized (Star/Snowflake) |
| **Users** | Thousands (concurrent) | Hundreds (analysts) |
| **Database size** | GBs | TBs to PBs |
| **Examples** | MySQL, PostgreSQL | Snowflake, BigQuery, Redshift |
| **Update frequency** | Continuous | Batch (hourly/daily/weekly) |

---

## 8. Data Warehousing Architecture

### 8.1 The ETL Pipeline

The backbone of any data warehouse is the **ETL process**:

```
Source Systems          ETL                    Data Warehouse
──────────────    ─────────────────────    ──────────────────────
  ERP             Extract   Transform        Staging Area
  CRM          ──────────▶ Clean       ──▶  ODS (Optional)
  Web Logs              Validate            Core DW (Facts + Dims)
  Flat Files            Enrich              Data Marts
  APIs                  Aggregate
```

**Extract:** Pull raw data from source systems (full load or incremental)  
**Transform:** Clean, validate, standardize, enrich, aggregate  
**Load:** Write to target tables in the warehouse

> Modern variant: **ELT** (Extract → Load → Transform) — load raw data first, transform inside the warehouse using its compute power. Used in cloud warehouses (BigQuery, Snowflake, Redshift).

---

### 8.2 Three-Layer Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   SOURCE LAYER                          │
│  CRM │ ERP │ Web Logs │ Flat Files │ APIs │ Databases  │
└─────────────────────┬───────────────────────────────────┘
                      │  Extract
                      ▼
┌─────────────────────────────────────────────────────────┐
│               INTEGRATION LAYER                         │
│   Staging Area → ODS → Transformation → Data Vault     │
│   (Raw, uncleaned data lives here temporarily)          │
└─────────────────────┬───────────────────────────────────┘
                      │  Load
                      ▼
┌─────────────────────────────────────────────────────────┐
│              PRESENTATION LAYER                         │
│  Core DW (Star/Snowflake) → Data Marts → OLAP Cubes    │
│  Reporting Tools, Dashboards, Ad-hoc Analytics          │
└─────────────────────────────────────────────────────────┘
```

#### Key Zones:

| Zone | Purpose |
|---|---|
| **Staging Area** | Temporary landing zone for raw extracted data; discarded after ETL |
| **ODS (Operational Data Store)** | Near-real-time integrated data; current state only; optional |
| **Core DW (EDW)** | Enterprise Data Warehouse — clean, historical, subject-oriented |
| **Data Mart** | Subject-specific subset (e.g., Sales Mart, Finance Mart); serves specific teams |
| **OLAP Cube** | Pre-aggregated multi-dimensional data for fast slicing and dicing |

---

### 8.3 Inmon vs Kimball Approach

| Approach | Inmon (Top-Down) | Kimball (Bottom-Up) |
|---|---|---|
| **Start with** | Enterprise-wide DW | Individual data marts |
| **Schema** | Normalized (3NF) | Denormalized (Star Schema) |
| **Complexity** | Higher upfront design cost | Easier to build incrementally |
| **Flexibility** | More flexible over time | Faster to deliver |
| **Widely used** | Large enterprises | Most modern DW implementations |

---

## 9. Facts and Dimensions

The **dimensional model** is the standard way to structure a data warehouse. It consists of **facts** and **dimensions**.

### 9.1 Fact Tables

A **fact table** contains the **measurable, quantitative data** about business events — the things you want to analyze.

**Characteristics:**
- Contains **numeric measures** (metrics)
- Contains **foreign keys** to dimension tables
- Often very large (millions to billions of rows)
- Each row represents one business event (a sale, a click, a payment)

```sql
CREATE TABLE fact_sales (
    sale_key       BIGINT PRIMARY KEY,      -- surrogate key
    date_key       INT REFERENCES dim_date,
    product_key    INT REFERENCES dim_product,
    customer_key   INT REFERENCES dim_customer,
    store_key      INT REFERENCES dim_store,
    -- Measures:
    quantity_sold  INT,
    unit_price     DECIMAL(10,2),
    discount_amt   DECIMAL(10,2),
    sales_amount   DECIMAL(10,2),
    cost_amount    DECIMAL(10,2),
    profit         DECIMAL(10,2)
);
```

**Types of Facts:**

| Type | Description | Example |
|---|---|---|
| **Additive** | Can be summed across ALL dimensions | `sales_amount`, `quantity` |
| **Semi-Additive** | Can be summed across SOME dimensions | `account_balance` (not across time) |
| **Non-Additive** | Cannot be summed (ratios, percentages) | `profit_margin`, `price` |

**Types of Fact Tables:**

| Type | Description |
|---|---|
| **Transaction Fact** | One row per transaction event (most common) |
| **Periodic Snapshot** | One row per period (end-of-day balance, weekly inventory) |
| **Accumulating Snapshot** | One row per process instance, updated as it moves through stages (order → shipped → delivered) |

---

### 9.2 Dimension Tables

**Dimension tables** contain the **descriptive context** — the "who, what, where, when, why" of each fact.

**Characteristics:**
- Contains descriptive text attributes
- Relatively smaller (thousands to millions of rows)
- Queried for filtering, grouping, and labeling
- Usually wide tables (many columns)

```sql
CREATE TABLE dim_product (
    product_key    INT PRIMARY KEY,          -- surrogate key
    product_id     VARCHAR(20),              -- natural/source key
    product_name   VARCHAR(200),
    category       VARCHAR(100),
    subcategory    VARCHAR(100),
    brand          VARCHAR(100),
    unit_cost      DECIMAL(10,2),
    launch_date    DATE,
    is_active      BOOLEAN,
    -- SCD columns (covered in Section 11):
    effective_date DATE,
    expiry_date    DATE,
    is_current     BOOLEAN
);
```

**Common Dimension Types:**

| Dimension | Description |
|---|---|
| **Date/Time** | Calendar attributes — year, quarter, month, day, week, fiscal periods |
| **Product** | Product name, category, brand, SKU |
| **Customer** | Demographics, segment, region, join date |
| **Geography** | Country, state, city, postal code |
| **Employee** | Name, role, department, hire date |
| **Channel** | Store, online, mobile, wholesale |

---

### 9.3 Surrogate Keys vs Natural Keys

| | Surrogate Key | Natural Key |
|---|---|---|
| **What it is** | System-generated integer (1, 2, 3…) | Business identifier (CUST-001, EMP-12345) |
| **Why use surrogate** | Handles SCDs cleanly; smaller joins; independent of source | N/A |
| **Example** | `product_key = 4421` | `product_id = 'PRD-SKU-8821'` |

**Always use surrogate keys in dimension tables** — natural keys change over time and break SCD tracking.

---

## 10. Star and Snowflake Schema

### 10.1 Star Schema

A **Star Schema** has one central fact table connected directly to multiple denormalized dimension tables. The diagram looks like a star.

```
                    dim_date
                       │
                       │
dim_customer ── fact_sales ── dim_product
                       │
                       │
                    dim_store
```

**Full Example:**

```
dim_date          fact_sales         dim_product
──────────        ──────────         ───────────
date_key ◄───── date_key            product_key ◄──────┐
year             product_key ──────▶ product_name       │
quarter          customer_key        category           │
month            store_key           brand              │
day              quantity_sold       unit_cost          │
day_of_week      sales_amount                           │
is_holiday       profit              dim_customer       │
                                     ───────────        │
                 ─────────────────▶  customer_key       │
                                     name               │
                                     city               │
                                     segment            │
```

**Advantages:**
- Simple to understand and query
- Fewer JOINs (fact directly to dimensions)
- Better query performance
- Easy for BI tools to navigate

**Disadvantages:**
- Data redundancy in dimension tables (city repeated per customer)
- Harder to maintain (update city name = update many rows)

---

### 10.2 Snowflake Schema

A **Snowflake Schema** normalizes the dimension tables — dimensions reference other sub-dimension tables instead of storing all attributes directly.

```
                              dim_date
                                 │
                                 │
dim_city ── dim_customer ── fact_sales ── dim_product ── dim_category
                                 │
                                 │
                              dim_store ── dim_region
```

**Example — Normalized Dimension:**

```sql
-- Star: denormalized customer
dim_customer(customer_key, name, city_name, state_name, country_name)

-- Snowflake: normalized
dim_customer(customer_key, name, city_key)
dim_city(city_key, city_name, state_key)
dim_state(state_key, state_name, country_key)
dim_country(country_key, country_name)
```

**Advantages:**
- Less redundancy in dimensions
- Easier to update (change a city name in one row)
- More storage efficient

**Disadvantages:**
- More JOINs → slower queries
- More complex for analysts to understand
- BI tools may struggle with deep hierarchies

---

### 10.3 Star vs Snowflake Comparison

| Feature | Star Schema | Snowflake Schema |
|---|---|---|
| **Structure** | Flat, denormalized dimensions | Normalized, hierarchical dimensions |
| **JOINs** | Fewer (fact → dimension) | More (fact → dim → sub-dim) |
| **Query speed** | Faster | Slower |
| **Storage** | More (redundancy) | Less |
| **Maintenance** | Harder (redundant updates) | Easier |
| **Complexity** | Simpler | More complex |
| **Best for** | Most DW use cases, BI tools | Strict normalization requirements |

> **Recommendation:** Start with Star Schema. Normalize only when storage is a real constraint or hierarchy maintenance becomes painful.

---

### 10.4 Galaxy Schema (Fact Constellation)

When **multiple fact tables share the same dimension tables**, the result is called a **Galaxy Schema**.

```
dim_date ──── fact_sales ──── dim_product
    │                              │
    └──── fact_inventory ──────────┘
                │
           dim_warehouse
```

Common in enterprises where multiple business processes share date, product, and customer dimensions.

---

## 11. Slowly Changing Dimensions (SCDs)

### 11.1 The Problem

Dimension data changes over time. For example:
- A customer moves cities
- An employee changes departments
- A product changes its category

**The question:** When a dimension attribute changes, should we **overwrite** the old value, **track the history**, or **both**?

The answer depends on the business need — and that's what the **SCD types** define.

---

### 11.2 SCD Type 0 — Retain Original

The dimension value **never changes** once loaded. The original value is always kept.

**Use case:** Date of birth, original join date, contract number  
**Implementation:** No special handling. Write once, never update.

---

### 11.3 SCD Type 1 — Overwrite

The old value is **overwritten** with the new value. **No history** is maintained.

```
Before: customer_key=1, name="Priya Kumar", city="Kochi"
Change: Priya moves to Bangalore

After: customer_key=1, name="Priya Kumar", city="Bangalore"

History of "Kochi" is LOST ❌
```

```sql
UPDATE dim_customer
SET city = 'Bangalore'
WHERE customer_key = 1;
```

**Use case:** Correcting errors, attributes where history doesn't matter (typo in name)  
**Pros:** Simple  
**Cons:** No history; historical reports may be affected

---

### 11.4 SCD Type 2 — Add New Row (Most Common)

A **new row** is inserted for the changed record. The old row is retained with an expiry date. History is fully preserved.

```
Before:
key | name        | city        | eff_date   | exp_date   | is_current
────┼─────────────┼─────────────┼────────────┼────────────┼───────────
1   | Priya Kumar | Kochi       | 2020-01-01 | 9999-12-31 | TRUE

After Priya moves to Bangalore:
key | name        | city        | eff_date   | exp_date   | is_current
────┼─────────────┼─────────────┼────────────┼────────────┼───────────
1   | Priya Kumar | Kochi       | 2020-01-01 | 2026-06-01 | FALSE  ← old row
2   | Priya Kumar | Bangalore   | 2026-06-02 | 9999-12-31 | TRUE   ← new row
```

```sql
-- Step 1: Expire old row
UPDATE dim_customer
SET exp_date = CURRENT_DATE - 1, is_current = FALSE
WHERE customer_key = 1 AND is_current = TRUE;

-- Step 2: Insert new row
INSERT INTO dim_customer
  (name, city, eff_date, exp_date, is_current)
VALUES
  ('Priya Kumar', 'Bangalore', CURRENT_DATE, '9999-12-31', TRUE);
```

**Use case:** Tracking address, department, role, salary band history  
**Pros:** Full historical reporting; most powerful SCD type  
**Cons:** Table grows larger; queries need to filter `is_current = TRUE` for current state

---

### 11.5 SCD Type 3 — Add New Column

Instead of a new row, a **new column** is added to track the previous value.

```
customer_key | name        | current_city | previous_city | city_change_date
─────────────┼─────────────┼──────────────┼───────────────┼──────────────────
1            | Priya Kumar | Bangalore    | Kochi         | 2026-06-02
```

```sql
ALTER TABLE dim_customer ADD COLUMN previous_city VARCHAR(100);
ALTER TABLE dim_customer ADD COLUMN city_change_date DATE;

UPDATE dim_customer
SET previous_city = current_city,
    current_city  = 'Bangalore',
    city_change_date = CURRENT_DATE
WHERE customer_key = 1;
```

**Use case:** When only one previous value matters (e.g., "current vs previous department")  
**Pros:** Simple; all in one row; easy to compare current vs previous  
**Cons:** Only tracks ONE previous value; poor for tracking more than 2 versions

---

### 11.6 SCD Types 4, 6 (Advanced — Brief)

| Type | Description |
|---|---|
| **Type 4 (Mini-Dimension)** | Frequently changing attributes moved to a separate "mini-dimension" (e.g., age band, income bracket separated from customer dim) |
| **Type 6 (Hybrid 1+2+3)** | Combines Type 1 (overwrite current), Type 2 (new row for history), and Type 3 (previous column); most complex but most flexible |

---

### 11.7 SCD Summary

| Type | History | Row Count | Complexity | When to Use |
|---|---|---|---|---|
| **Type 0** | None (immutable) | Same | Trivial | Immutable attributes |
| **Type 1** | None (overwrite) | Same | Low | Error corrections |
| **Type 2** | Full | Grows | Medium | Most tracking needs |
| **Type 3** | One version back | Same | Low-Medium | Simple "current vs previous" |
| **Type 4** | In mini-dimension | New table | High | Rapidly changing attributes |
| **Type 6** | Full + current + previous | Grows | High | Full flexibility needed |

---

## 12. Indexing & Clustering

### 12.1 What is an Index?

An **index** is a data structure that allows the database to find rows quickly **without scanning the entire table** (avoiding a full table scan).

Think of it like the index at the back of a textbook — instead of reading all 500 pages to find "normalization", you look it up in the index and jump to page 47.

```
Without index: Scan all 10 million rows → slow O(n)
With index:    B-tree lookup → fast O(log n)
```

### 12.2 How B-Tree Indexes Work

The most common index type. A **balanced tree** where:
- Each leaf node holds a key + pointer to the actual row (ROWID / page offset)
- Sorted order allows binary search
- Insert/delete automatically rebalances

```
                  [40 | 70]
                /     |     \
         [10|20]   [50|60]   [80|90]
           /\         /\         /\
         rows       rows       rows
```

```sql
-- Create an index
CREATE INDEX idx_customer_city ON customers(city);

-- Composite index
CREATE INDEX idx_orders_date_status ON orders(order_date, status);

-- Unique index
CREATE UNIQUE INDEX idx_email ON customers(email);
```

### 12.3 Index Types

| Type | Description | Best For |
|---|---|---|
| **B-Tree** | Default; sorted tree; good for equality and range | Most queries; columns with high cardinality |
| **Hash** | Hash table; O(1) exact lookup; no range support | Exact equality only (`=`) |
| **Bitmap** | One bit per row per value; very compact | Low-cardinality columns (gender, status, boolean) |
| **Full-Text** | Inverted index for text search | `LIKE '%keyword%'`, search engines |
| **Partial** | Index only rows matching a condition | `WHERE status = 'active'` — index only active rows |
| **Covering** | Index includes all columns needed by a query | Avoid table lookup entirely (index-only scan) |
| **Composite** | Index on multiple columns | Multi-column WHERE and ORDER BY |

### 12.4 When Indexes Help and When They Don't

**Indexes HELP when:**
- Querying a small fraction of rows (`WHERE customer_id = 1001`)
- Joining tables on indexed columns (`ON a.id = b.id`)
- Sorting large tables (`ORDER BY date_key`)
- Range queries (`WHERE order_date BETWEEN ... AND ...`)

**Indexes DON'T HELP (or hurt) when:**
- Query returns most rows (>10–30% of table) → full scan is faster
- Heavy write workloads — every INSERT/UPDATE/DELETE must update indexes
- Functions applied to indexed columns (`WHERE YEAR(order_date) = 2026` — won't use index on `order_date`)
- Too many indexes on one table — increases write overhead and storage

---

### 12.5 Clustering

**Clustering** determines the **physical order in which rows are stored** on disk. When rows related to a query are stored close together, disk I/O is minimized.

#### Clustered Index

The table data itself is sorted and stored in the order of the index. There can be **only one** clustered index per table (the data can only be physically sorted one way).

```sql
-- In SQL Server / MySQL InnoDB, the PRIMARY KEY is the clustered index
CREATE TABLE orders (
    order_id    INT PRIMARY KEY,   -- ← data stored in order of order_id
    order_date  DATE,
    customer_id INT
);
```

```
Clustered (physically sorted by order_id):
order_id=1 | data...
order_id=2 | data...
order_id=3 | data...
...
```

#### Non-Clustered (Secondary) Index

The index contains the key + a **pointer (bookmark)** back to the actual row. Multiple allowed per table.

```
Non-Clustered idx on city:
Bangalore → [rowid: 15, 67, 203, ...]  → go fetch actual rows
Kochi     → [rowid: 4, 89, ...]
```

#### Heap vs Clustered Table

| | Heap (no clustered index) | Clustered Table |
|---|---|---|
| **Row order** | Random (insertion order) | Sorted by clustered key |
| **Range scans** | Slow | Fast |
| **Updates that change key** | N/A | Expensive (row must move) |

---

### 12.6 Columnar Storage (Data Warehouse)

In analytics databases (BigQuery, Redshift, Snowflake), data is stored **column by column**, not row by row.

```
Row Store:
Row 1: [2026-01-01, 'Laptop', 75000, 'Kochi']
Row 2: [2026-01-02, 'Mouse',    500, 'Kochi']

Column Store:
dates:    [2026-01-01, 2026-01-02]
products: ['Laptop', 'Mouse']
amounts:  [75000, 500]
cities:   ['Kochi', 'Kochi']
```

**Why columnar is great for OLAP:**
- Queries typically touch only 2–3 columns out of 50 → only those columns are read from disk
- Same-type column data compresses extremely well (run-length encoding, dictionary encoding)
- Aggregations (`SUM`, `AVG`, `COUNT`) work directly on columns without loading rows

---

## 13. Sharding

### 13.1 What is Sharding?

**Sharding** is the process of **horizontally partitioning** a database across multiple independent database instances (called **shards**), each holding a **subset of the data**.

Each shard is a full, independent database. Together they form one logical database.

```
BEFORE SHARDING (single DB, overloaded):
┌────────────────────────┐
│    users table         │
│    100 million rows    │  ← too slow, one machine
└────────────────────────┘

AFTER SHARDING (3 shards):
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Shard 1    │  │  Shard 2    │  │  Shard 3    │
│  users      │  │  users      │  │  users      │
│  id: 1–33M  │  │  id: 33M–66M│  │  id: 66M–100M│
└─────────────┘  └─────────────┘  └─────────────┘
```

### 13.2 Sharding Strategies

#### Range-Based Sharding
Rows are assigned to shards based on a **range of the shard key**.

```
Shard Key: user_id
Shard 1: user_id 1        – 10,000,000
Shard 2: user_id 10000001 – 20,000,000
Shard 3: user_id 20000001 – 30,000,000
```

**Pros:** Simple; range queries stay on one shard  
**Cons:** Hotspots (if new users all go to Shard 3); uneven distribution

---

#### Hash-Based Sharding
A **hash function** maps the shard key to a shard number.

```
shard_number = hash(user_id) % number_of_shards

hash("user_1001") % 3 = 2  → Shard 2
hash("user_1002") % 3 = 0  → Shard 0
hash("user_1003") % 3 = 1  → Shard 1
```

**Pros:** Even distribution; no hotspots  
**Cons:** Range queries hit all shards; resharding is expensive (all keys need rehashing)

**Consistent Hashing** solves the resharding problem by using a ring structure where only a fraction of keys move when adding/removing shards.

---

#### Directory-Based Sharding
A **lookup table** maps each key (or range) to a shard.

```
Lookup Table:
user_id range    → Shard
────────────────────────
1 – 5M           → Shard A
5M – 15M         → Shard B (hot; bigger range for this shard)
15M – 20M        → Shard C
```

**Pros:** Flexible; can handle hot spots by reassigning ranges  
**Cons:** Lookup table becomes a bottleneck; single point of failure

---

#### Geographic / Tenant Sharding
Data is partitioned by region or customer (common in multi-tenant SaaS).

```
Shard India:   All rows where region = 'IN'
Shard Europe:  All rows where region = 'EU'
Shard US:      All rows where region = 'US'
```

---

### 13.3 Challenges of Sharding

| Challenge | Description |
|---|---|
| **Cross-shard queries** | JOINs across shards are expensive — you must query all shards and merge results |
| **Resharding** | Adding/removing shards requires moving data; downtime risk |
| **Transactions** | Distributed transactions across shards are complex (2-phase commit) |
| **Hotspots** | If shard key is poorly chosen, one shard gets all the traffic |
| **Operational complexity** | Managing N databases instead of 1 is harder |

### 13.4 Sharding vs Partitioning

| | Partitioning | Sharding |
|---|---|---|
| **Where** | Single database instance | Multiple database instances |
| **Goal** | Manage large tables within one DB | Distribute load across machines |
| **JOINs** | Transparent — DB handles it | Must be application-managed |
| **Example** | PostgreSQL table partitioning by month | MongoDB sharded cluster |

---

## 14. Query Optimization

### 14.1 What is Query Optimization?

**Query optimization** is the process by which a database engine (or a data engineer) selects the most **efficient execution plan** for a SQL query — minimizing CPU, memory, and I/O usage.

### 14.2 The Query Execution Pipeline

```
SQL Query Text
      │
      ▼
  Parser   ← Checks syntax, builds parse tree
      │
      ▼
  Semantic Analyzer  ← Validates tables, columns exist; resolves types
      │
      ▼
  Query Rewriter  ← Expands views, applies transformations
      │
      ▼
  Query Optimizer  ← Generates candidate plans, estimates costs, picks best
      │
      ▼
  Execution Engine  ← Runs the chosen plan
      │
      ▼
  Results
```

### 14.3 Understanding EXPLAIN / EXPLAIN ANALYZE

The most important tool for query optimization.

```sql
-- See the query plan (PostgreSQL)
EXPLAIN SELECT * FROM orders WHERE customer_id = 1001;

-- See actual execution times
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 1001;
```

**Key things to look for in an EXPLAIN output:**

| Term | What It Means |
|---|---|
| **Seq Scan** | Full table scan — reads every row. Slow on large tables. |
| **Index Scan** | Uses an index to find rows. Fast. |
| **Index Only Scan** | Answers from index alone, no table access. Fastest. |
| **Bitmap Index Scan** | Collects matching row IDs from index, then fetches rows. |
| **Nested Loop Join** | For each row in outer table, scan inner table. Good for small sets. |
| **Hash Join** | Build hash table from smaller table, probe with larger. Good for large equal joins. |
| **Merge Join** | Sort both sides, merge. Good when inputs are already sorted. |
| **cost=X..Y** | Startup cost..total cost (in DB-internal units) |
| **rows=N** | Estimated number of rows |
| **actual time** | (EXPLAIN ANALYZE only) real time taken |

---

### 14.4 Query Optimization Techniques

#### 1. Use Indexes Appropriately

```sql
-- ❌ Won't use index on order_date because of function:
SELECT * FROM orders WHERE YEAR(order_date) = 2026;

-- ✅ Range query — will use index:
SELECT * FROM orders WHERE order_date BETWEEN '2026-01-01' AND '2026-12-31';
```

---

#### 2. Select Only What You Need

```sql
-- ❌ Fetches all columns, many may be unused:
SELECT * FROM orders WHERE customer_id = 1001;

-- ✅ Fetch only needed columns:
SELECT order_id, order_date, total_amount FROM orders WHERE customer_id = 1001;
```

---

#### 3. Filter Early (Push Predicates Down)

```sql
-- ❌ Joins all rows, then filters:
SELECT o.order_id, c.name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date > '2026-01-01' AND c.region = 'South';

-- ✅ Filter before joining (CTEs or subqueries for clarity, optimizer does this automatically in good DBs):
WITH recent_orders AS (
    SELECT * FROM orders WHERE order_date > '2026-01-01'
),
south_customers AS (
    SELECT * FROM customers WHERE region = 'South'
)
SELECT r.order_id, s.name
FROM recent_orders r
JOIN south_customers s ON r.customer_id = s.customer_id;
```

---

#### 4. Avoid N+1 Query Problem

```sql
-- ❌ N+1: one query to get orders, then N queries for each customer name
for order in orders:
    customer = SELECT name FROM customers WHERE id = order.customer_id

-- ✅ One JOIN:
SELECT o.order_id, c.name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

---

#### 5. Use Covering Indexes

```sql
-- Query:
SELECT order_date, total_amount FROM orders WHERE customer_id = 1001;

-- Regular index on customer_id: index lookup → fetch row from table
-- Covering index on (customer_id, order_date, total_amount): index only scan!

CREATE INDEX idx_covering ON orders(customer_id, order_date, total_amount);
```

---

#### 6. Partition Pruning

If a table is partitioned (e.g., by month), queries that filter on the partition key only scan relevant partitions.

```sql
-- Table partitioned by order_date (monthly)
-- Query:
SELECT * FROM orders WHERE order_date BETWEEN '2026-06-01' AND '2026-06-30';
-- → Only June 2026 partition is scanned ✓
```

---

#### 7. Avoid SELECT DISTINCT When Possible

`DISTINCT` forces a sort or hash dedup of the entire result set — expensive.

```sql
-- ❌ Often unnecessary if JOINs are correct:
SELECT DISTINCT c.customer_id FROM customers c JOIN orders o ON c.customer_id = o.customer_id;

-- ✅ Use EXISTS instead:
SELECT customer_id FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);
```

---

#### 8. Statistics & Vacuuming

The query optimizer relies on **statistics** (row counts, value distributions, NULL percentages) to estimate costs and choose plans.

```sql
-- PostgreSQL: update statistics manually
ANALYZE orders;

-- PostgreSQL: reclaim dead rows from UPDATEs/DELETEs
VACUUM ANALYZE orders;
```

---

### 14.5 Query Optimization Checklist

```
□ Do the WHERE columns have indexes?
□ Are you selecting only the columns you need (no SELECT *)?
□ Are functions applied to indexed columns in WHERE clauses?
□ Are JOINs on indexed columns?
□ Is the smaller table on the build side of a Hash Join?
□ Is the table partitioned, and is the partition key in the filter?
□ Have you run EXPLAIN ANALYZE to see the actual plan?
□ Are statistics up to date (last ANALYZE run)?
□ Are there any full table scans that should be index scans?
□ Are there any N+1 query patterns?
□ Can expensive aggregations be pre-computed (materialized views)?
```

---

### 14.6 Materialized Views

A **materialized view** stores the result of a query physically on disk. Useful for expensive aggregations that are queried frequently.

```sql
-- Create a materialized view of monthly sales
CREATE MATERIALIZED VIEW mv_monthly_sales AS
SELECT
    DATE_TRUNC('month', order_date) AS month,
    product_id,
    SUM(sales_amount) AS total_sales,
    COUNT(*) AS order_count
FROM fact_sales
GROUP BY DATE_TRUNC('month', order_date), product_id;

-- Query it instantly (no aggregation at query time):
SELECT * FROM mv_monthly_sales WHERE month = '2026-06-01';

-- Refresh when underlying data changes:
REFRESH MATERIALIZED VIEW mv_monthly_sales;
```

---

##  Quick Revision Cheatsheet

| Topic | Key Takeaway |
|---|---|
| **DBMS** | Software layer between user and data; provides ACID, concurrency, security |
| **RDBMS** | Tables + SQL + strong ACID; best for structured, relational data |
| **NoSQL** | 4 types: KV, Document, Column-Family, Graph; scale-out, flexible schema |
| **ACID** | Atomicity (all-or-nothing), Consistency (valid state), Isolation (concurrent safety), Durability (survives crash) |
| **ER Model** | Entities + Attributes + Relationships; blueprint before SQL; M:N → junction table |
| **1NF** | Atomic values, single data type per column, unique rows |
| **2NF** | 1NF + no partial dependencies on composite PK |
| **3NF** | 2NF + no transitive dependencies between non-key attributes |
| **BCNF** | Every determinant is a superkey |
| **Denormalization** | Intentional redundancy for read performance (OLAP use case) |
| **Data Warehouse** | Subject-oriented, integrated, non-volatile, time-variant; for analytics |
| **OLTP** | Operational transactions; normalized; millisecond responses; small rows per query |
| **OLAP** | Analytical queries; denormalized; seconds/minutes; millions of rows per query |
| **ETL/ELT** | Extract → Transform → Load (or E→L→T in cloud); pipeline to populate DW |
| **Fact Table** | Measurable events; FK to dims; additive/semi-additive/non-additive measures |
| **Dimension Table** | Descriptive context (who, what, where, when); surrogate keys |
| **Star Schema** | Fact + flat dimensions; fewer JOINs; fastest for BI tools |
| **Snowflake Schema** | Fact + normalized dimensions; less redundancy; more JOINs |
| **SCD Type 1** | Overwrite old value; no history |
| **SCD Type 2** | New row per change; full history; most common |
| **SCD Type 3** | New column for previous value; one version back |
| **B-Tree Index** | Default; balanced tree; equality + range lookups |
| **Clustered Index** | Data physically sorted by this key; one per table |
| **Sharding** | Horizontal partition across multiple DB instances |
| **Range Sharding** | By key range; good for locality; hotspot risk |
| **Hash Sharding** | By hash; even distribution; no range queries |
| **EXPLAIN** | Show query execution plan; look for Seq Scans to optimize |
| **Materialized View** | Pre-computed query result stored on disk; refresh manually |

---

##  Recommended Further Reading

- **Books:**
  - *The Data Warehouse Toolkit* — Ralph Kimball & Margy Ross (Dimensional Modelling bible)
  - *Database System Concepts* — Silberschatz, Korth, Sudarshan (DBMS fundamentals)
  - *Designing Data-Intensive Applications* — Martin Kleppmann (Modern distributed data)
  - *Fundamentals of Data Engineering* — Joe Reis & Matt Housley

- **Online:**
  - [Use The Index, Luke](https://use-the-index-luke.com) — SQL indexing deep dive
  - [PostgreSQL Documentation](https://www.postgresql.org/docs/) — EXPLAIN, indexes, partitioning
  - [Kimball Group](https://www.kimballgroup.com) — Data Warehousing articles

---

*© AkumenbyQ — For educational use. Last updated June 2026.*