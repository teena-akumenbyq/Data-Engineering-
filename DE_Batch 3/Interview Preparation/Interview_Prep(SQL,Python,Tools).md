# Data Engineering Interview Preparation
## SQL · Python · Core Concepts · Tools
### For Beginners

---

# Section 1: SQL

---

**Q1. What is SQL and why is it important for data engineering?**

SQL (Structured Query Language) is the standard language for querying and manipulating relational databases. Data engineers use SQL to extract data from source systems, transform data inside warehouses, write pipeline logic, and validate data quality. It is the single most tested skill in data engineering interviews.

---

**Q2. What are the different types of JOINs?**

| JOIN Type | What it returns |
|---|---|
| `INNER JOIN` | Only rows that match in both tables |
| `LEFT JOIN` | All rows from the left table; NULLs for non-matching right rows |
| `RIGHT JOIN` | All rows from the right table; NULLs for non-matching left rows |
| `FULL OUTER JOIN` | All rows from both tables; NULLs where no match |
| `CROSS JOIN` | Every combination of rows from both tables (cartesian product) |
| `SELF JOIN` | A table joined with itself (used for hierarchical data) |

```sql
-- Example: LEFT JOIN
SELECT c.customer_name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
-- Customers with no orders will appear with NULL order_id
```

---

**Q3. What is the difference between WHERE and HAVING?**

- `WHERE` filters individual rows **before** grouping
- `HAVING` filters groups **after** `GROUP BY`

```sql
-- WHERE: filter before grouping
SELECT department, COUNT(*) AS headcount
FROM employees
WHERE status = 'active'         -- filters rows first
GROUP BY department
HAVING COUNT(*) > 10;           -- then filters groups
```

> **Rule of thumb:** If you are filtering on an aggregation (SUM, COUNT, AVG), use `HAVING`. Everything else uses `WHERE`.

---

**Q4. What are window functions? Give an example.**

Window functions perform calculations across a set of rows related to the current row — without collapsing rows the way `GROUP BY` does. They are heavily used in data engineering for rankings, running totals, and time-series analysis.

```sql
-- ROW_NUMBER: rank each order per customer by date
SELECT
    customer_id,
    order_id,
    order_date,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS row_num
FROM orders;

-- SUM with running total
SELECT
    order_date,
    revenue,
    SUM(revenue) OVER (ORDER BY order_date) AS running_total
FROM daily_sales;
```

Common window functions:

| Function | Purpose |
|---|---|
| `ROW_NUMBER()` | Unique sequential number per partition |
| `RANK()` | Rank with gaps for ties |
| `DENSE_RANK()` | Rank without gaps for ties |
| `LAG(col, n)` | Value from n rows before |
| `LEAD(col, n)` | Value from n rows ahead |
| `SUM() OVER(...)` | Running or partitioned total |
| `NTILE(n)` | Divide rows into n buckets |

---

**Q5. What is a CTE (Common Table Expression)?**

A CTE is a temporary named result set defined with `WITH`. It makes complex queries more readable and reusable within a single statement. CTEs do not persist after the query ends.

```sql
WITH monthly_revenue AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY 1
)
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue
FROM monthly_revenue;
```

> **Tip:** Use CTEs instead of nested subqueries for readability. You can define multiple CTEs in one `WITH` block.

---

**Q6. What is the difference between DELETE, TRUNCATE, and DROP?**

| Command | What it does | Rollback? | Removes structure? |
|---|---|---|---|
| `DELETE` | Removes specific rows (use with `WHERE`) | Yes | No |
| `TRUNCATE` | Removes all rows quickly, resets identity | Usually no | No |
| `DROP` | Removes entire table including structure | No | Yes |

```sql
DELETE FROM orders WHERE order_date < '2020-01-01';  -- specific rows
TRUNCATE TABLE staging_orders;                        -- all rows, fast
DROP TABLE temp_data;                                 -- remove entire table
```

---

**Q7. What is the difference between UNION and UNION ALL?**

- `UNION` combines results from two queries and **removes duplicates** (slower)
- `UNION ALL` combines results and **keeps all rows** including duplicates (faster)

```sql
-- UNION ALL is preferred in pipelines unless deduplication is explicitly needed
SELECT customer_id FROM source_a
UNION ALL
SELECT customer_id FROM source_b;
```

> **Data engineering tip:** Always use `UNION ALL` in pipelines unless you specifically need deduplication. `UNION` adds a costly sort/dedup step.

---

**Q8. How do you find and remove duplicate records?**

```sql
-- Step 1: Find duplicates
SELECT email, COUNT(*) AS cnt
FROM customers
GROUP BY email
HAVING COUNT(*) > 1;

-- Step 2: Keep only the latest record per email using ROW_NUMBER
WITH deduped AS (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY created_at DESC) AS rn
    FROM customers
)
SELECT * FROM deduped WHERE rn = 1;
```

---

**Q9. What are indexes and why do they matter?**

An index is a data structure that speeds up data retrieval on a column. Without an index, the database scans every row (full table scan). With an index, it jumps directly to matching rows.

- **Use indexes on:** columns used in `WHERE`, `JOIN`, or `ORDER BY`
- **Avoid over-indexing:** indexes slow down `INSERT`/`UPDATE`/`DELETE` because the index must be updated too

```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);
```

---

**Q10. What is the difference between OLTP and OLAP databases?**

| | OLTP | OLAP |
|---|---|---|
| Full form | Online Transaction Processing | Online Analytical Processing |
| Optimised for | Fast reads/writes of individual rows | Complex aggregations over many rows |
| Schema | Highly normalised | Denormalised (star/snowflake schema) |
| Examples | MySQL, PostgreSQL, Azure SQL | Snowflake, BigQuery, Azure Synapse |
| Used by | Applications (checkout, login) | Analysts, data engineers, dashboards |

---

**Q11. What is a subquery? How is it different from a CTE?**

A subquery is a query nested inside another query. Both subqueries and CTEs achieve similar results, but CTEs are more readable and can be referenced multiple times in the same query.

```sql
-- Subquery (harder to read for complex logic)
SELECT * FROM orders
WHERE customer_id IN (SELECT customer_id FROM customers WHERE country = 'India');

-- CTE (same result, more readable)
WITH indian_customers AS (
    SELECT customer_id FROM customers WHERE country = 'India'
)
SELECT * FROM orders WHERE customer_id IN (SELECT customer_id FROM indian_customers);
```

---

**Q12. What is a MERGE statement?**

`MERGE` (also called upsert) allows you to `INSERT`, `UPDATE`, or `DELETE` in a single statement based on whether a matching row exists. It is essential for incremental pipeline loads.

```sql
MERGE INTO target_table AS target
USING source_table AS source
ON target.id = source.id
WHEN MATCHED THEN
    UPDATE SET target.value = source.value, target.updated_at = GETDATE()
WHEN NOT MATCHED BY TARGET THEN
    INSERT (id, value, updated_at) VALUES (source.id, source.value, GETDATE())
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

---

# Section 2: Python

---

**Q13. Why is Python used in data engineering?**

Python is the dominant language in data engineering because of its rich ecosystem of libraries, readability, and broad support across cloud platforms. It is used to write ETL scripts, interact with APIs, automate file processing, and build data pipelines.

---

**Q14. What is Pandas? What are the key operations every data engineer should know?**

Pandas is a Python library for data manipulation and analysis. It provides `DataFrame` (a table) and `Series` (a column) as its core structures.

```python
import pandas as pd

# Read data
df = pd.read_csv('sales.csv')
df = pd.read_parquet('sales.parquet')

# Inspect
df.head()           # first 5 rows
df.info()           # column types and null counts
df.describe()       # summary statistics
df.shape            # (rows, columns)

# Select columns
df[['name', 'revenue']]

# Filter rows
df[df['revenue'] > 1000]
df.query("country == 'India' and revenue > 500")

# Group and aggregate
df.groupby('department')['revenue'].sum()

# Handle nulls
df.isnull().sum()                   # count nulls per column
df.fillna(0)                        # replace nulls with 0
df.dropna(subset=['customer_id'])   # drop rows with null customer_id

# Rename and cast
df.rename(columns={'rev': 'revenue'})
df['revenue'] = df['revenue'].astype(float)

# Sort
df.sort_values('revenue', ascending=False)

# Save
df.to_parquet('output.parquet', index=False)
df.to_csv('output.csv', index=False)
```

---

**Q15. How do you read a CSV file and connect to a database in Python?**

```python
import pandas as pd
from sqlalchemy import create_engine

# Read CSV
df = pd.read_csv('data.csv')

# Connect to PostgreSQL and read
engine = create_engine('postgresql://user:password@host:5432/dbname')
df = pd.read_sql("SELECT * FROM orders WHERE status = 'complete'", engine)

# Write DataFrame to a database table
df.to_sql('staging_orders', engine, if_exists='replace', index=False)
# if_exists options: 'fail', 'replace', 'append'
```

> **Tip:** In production, never hardcode credentials. Use environment variables or a secrets manager (like Azure Key Vault).

---

**Q16. What are list comprehensions and why are they useful?**

List comprehensions provide a concise way to create lists by applying an expression to each item in an iterable — often replacing a `for` loop in a single line.

```python
# Standard for loop
squares = []
for x in range(10):
    squares.append(x ** 2)

# List comprehension (equivalent, more concise)
squares = [x ** 2 for x in range(10)]

# With condition
even_squares = [x ** 2 for x in range(10) if x % 2 == 0]

# Practical DE use: extract filenames from a list of paths
paths = ['/data/2024/jan.csv', '/data/2024/feb.csv']
filenames = [p.split('/')[-1] for p in paths]
# ['jan.csv', 'feb.csv']
```

---

**Q17. How do you work with JSON data in Python?**

```python
import json
import requests

# Parse JSON string
json_str = '{"name": "Alice", "age": 30}'
data = json.loads(json_str)
print(data['name'])  # Alice

# Write to JSON file
with open('output.json', 'w') as f:
    json.dump(data, f, indent=2)

# Read from JSON file
with open('output.json', 'r') as f:
    data = json.load(f)

# Fetch from a REST API
response = requests.get('https://api.example.com/orders')
orders = response.json()  # list of dicts

# Convert to DataFrame
df = pd.DataFrame(orders)

# Flatten nested JSON
df_flat = pd.json_normalize(orders, sep='_')
```

---

**Q18. What is error handling in Python and how do you use it in pipelines?**

Error handling prevents a pipeline from crashing silently and ensures failures are logged and handled gracefully.

```python
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def load_data(filepath):
    try:
        df = pd.read_csv(filepath)
        logging.info(f"Loaded {len(df)} rows from {filepath}")
        return df
    except FileNotFoundError:
        logging.error(f"File not found: {filepath}")
        raise
    except Exception as e:
        logging.error(f"Unexpected error loading {filepath}: {e}")
        raise
    finally:
        logging.info("Load attempt completed")
```

> **Pipeline rule:** Always log what was loaded, how many rows were processed, and any errors — with enough detail to debug without re-running.

---

**Q19. What are Python decorators? What is `*args` and `**kwargs`?**

**Decorators** wrap a function to add behaviour without modifying it. Common in frameworks like Airflow (`@dag`, `@task`).

```python
# Simple decorator example
def log_execution(func):
    def wrapper(*args, **kwargs):
        print(f"Running {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@log_execution
def extract_data():
    return pd.read_csv('data.csv')
```

**`*args`** collects extra positional arguments as a tuple.  
**`**kwargs`** collects extra keyword arguments as a dictionary.

```python
def pipeline_config(name, *args, **kwargs):
    print(f"Pipeline: {name}")
    print(f"Steps: {args}")
    print(f"Options: {kwargs}")

pipeline_config("daily_load", "extract", "transform", retries=3, timeout=60)
```

---

**Q20. How do you work with files and directories in Python?**

```python
import os
from pathlib import Path

# List files in a directory
files = os.listdir('/data/raw/')

# Using pathlib (modern and preferred)
data_dir = Path('/data/raw')
csv_files = list(data_dir.glob('*.csv'))
parquet_files = list(data_dir.rglob('*.parquet'))  # recursive

# Check if file exists
if Path('output.csv').exists():
    print("File found")

# Create directories
Path('/data/processed/2024').mkdir(parents=True, exist_ok=True)

# Read all CSV files in a folder into one DataFrame
import pandas as pd
df = pd.concat([pd.read_csv(f) for f in csv_files], ignore_index=True)
```

---

# Section 3: Core Concepts

---

**Q21. What is ETL vs ELT?**

| | ETL | ELT |
|---|---|---|
| Full form | Extract → Transform → Load | Extract → Load → Transform |
| Where transformation happens | Before loading (external compute) | Inside the data warehouse |
| Best for | Legacy systems, sensitive data masking | Cloud data warehouses (Snowflake, BigQuery, Synapse) |
| Tools | Informatica, SSIS, custom Python | dbt, Spark SQL inside warehouse |

> **Modern trend:** ELT has become the standard in cloud-native architectures because data warehouses like BigQuery and Synapse are powerful enough to run transformations at scale using SQL.

---

**Q22. What is a data warehouse vs a data lake vs a data lakehouse?**

| | Data Warehouse | Data Lake | Data Lakehouse |
|---|---|---|---|
| Data type | Structured only | Structured + unstructured | All types |
| Schema | Schema-on-write | Schema-on-read | Schema-on-read + enforcement |
| Storage cost | High | Low | Low |
| Query speed | Fast | Slower without indexing | Fast (with Delta/Iceberg) |
| Examples | Snowflake, Redshift, Synapse | S3, ADLS Gen2, GCS | Databricks, Delta Lake, Apache Iceberg |

A **data lakehouse** combines the low-cost storage of a data lake with the ACID transactions and query performance of a data warehouse.

---

**Q23. What is batch processing vs stream processing?**

| | Batch | Streaming |
|---|---|---|
| Data processed | Large accumulated chunks | Individual events as they arrive |
| Latency | Minutes to hours | Milliseconds to seconds |
| Complexity | Lower | Higher |
| Tools | Spark, ADF, SQL jobs | Kafka, Flink, Spark Structured Streaming, Azure Stream Analytics |
| Use case | Daily sales report, payroll | Fraud detection, live dashboards, IoT monitoring |

---

**Q24. What is data partitioning?**

Partitioning splits large datasets into smaller, manageable chunks based on a column value — typically date. This dramatically improves query performance because the engine only scans relevant partitions (partition pruning).

```
/data/sales/year=2024/month=01/day=01/data.parquet
/data/sales/year=2024/month=01/day=02/data.parquet
```

Benefits:
- Faster queries — only relevant partitions are read
- Easier incremental loading — process only today's partition
- Lower cost — scan less data, pay less in cloud warehouses

> **Beginner tip:** Always partition large tables by date. Partition by a higher-cardinality key (like user_id) only if you regularly filter by it.

---

**Q25. What is idempotency in data pipelines?**

A pipeline is **idempotent** if running it multiple times produces the same result as running it once. This is critical because pipelines fail and must be safely re-run.

**Non-idempotent (bad):**
```python
# Appends rows every run — creates duplicates on re-run
df.to_sql('orders', engine, if_exists='append')
```

**Idempotent (good):**
```python
# Overwrites the partition on every run — safe to re-run
df.to_sql('orders', engine, if_exists='replace')
# Or use MERGE/UPSERT in SQL to handle re-runs safely
```

> **Rule:** Design every pipeline step to be safely re-runnable without side effects.

---

**Q26. What is a DAG (Directed Acyclic Graph)?**

A DAG is a graph of tasks where edges represent dependencies and no task can depend on itself (no cycles). In data engineering, DAGs define the execution order of pipeline steps.

```
extract_data → validate_data → transform_data → load_to_warehouse → notify
```

- **Directed** — tasks have a defined order
- **Acyclic** — no circular dependencies (Task A cannot depend on Task B if B depends on A)

DAGs are the core abstraction in Apache Airflow and Azure Data Factory pipelines.

---

**Q27. What is data normalisation vs denormalisation?**

**Normalisation** eliminates redundancy by splitting data into related tables (used in OLTP systems).

```
orders table: order_id, customer_id, product_id, quantity
customers table: customer_id, name, email
products table: product_id, name, price
```

**Denormalisation** combines tables into fewer, wider tables for faster read performance (used in OLAP/data warehouses).

```
orders_flat table: order_id, customer_name, customer_email, product_name, price, quantity
```

> **Rule:** Normalise in operational databases; denormalise in data warehouses for analytical performance.

---

**Q28. What is a star schema?**

A star schema is the most common data warehouse design pattern. It has:
- One central **fact table** — contains measurable events (sales, clicks, transactions) with foreign keys
- Multiple **dimension tables** — contain descriptive attributes (who, what, where, when)

```
dim_customer ──┐
dim_product  ──┼──▶ fact_sales ◀──── dim_date
dim_store    ──┘
```

The fact table stores `customer_id`, `product_id`, `store_id`, `date_id`, `quantity`, `revenue`.  
Dimensions store full descriptive details like `customer_name`, `product_category`, `store_city`.

---

**Q29. What is the difference between SCD Type 1 and SCD Type 2?**

SCD stands for **Slowly Changing Dimension** — how to handle changes to dimension data over time.

| | SCD Type 1 | SCD Type 2 |
|---|---|---|
| Strategy | Overwrite the old value | Add a new row for each change |
| History preserved | No | Yes |
| Use case | Fixing errors, non-important changes | Tracking historical state (e.g. customer address) |

```sql
-- SCD Type 2 adds columns to track versions
-- customer_id | name  | city      | is_current | valid_from | valid_to
-- 1001        | Alice | Mumbai    | FALSE      | 2022-01-01 | 2023-06-01
-- 1001        | Alice | Bangalore | TRUE       | 2023-06-01 | NULL
```

---

**Q30. What is data lineage?**

Data lineage tracks the complete journey of data — from its origin through every transformation to its final destination. It answers: *"Where did this number come from?"*

Importance in data engineering:
- Debugging — trace where incorrect data was introduced
- Compliance — prove to auditors where sensitive data has flowed
- Impact analysis — understand which pipelines break if a source table changes

Tools: Microsoft Purview, Apache Atlas, OpenLineage, dbt's built-in lineage graph.

---

**Q31. What is data quality and how do you validate it in pipelines?**

Data quality ensures data is accurate, complete, consistent, and timely. Common checks added to pipelines:

| Check type | Example |
|---|---|
| Null check | `customer_id` must never be NULL |
| Uniqueness check | No duplicate `order_id` values |
| Range check | `age` must be between 0 and 120 |
| Referential integrity | Every `product_id` in orders must exist in products |
| Freshness check | Table must have been updated in the last 24 hours |
| Row count check | Today's row count should be within 20% of yesterday's |

```python
# Example: null check in Python
assert df['customer_id'].isnull().sum() == 0, "customer_id contains NULL values"

# Row count validation
assert len(df) > 0, "Empty dataset loaded — pipeline may have failed"
```

---

# Section 4: Tools

---

**Q32. What is Apache Spark and what makes it powerful?**

Apache Spark is an open-source distributed data processing engine. It processes large datasets across a cluster of machines in parallel, keeping intermediate data in memory — making it significantly faster than older disk-based systems.

Key concepts:
- **RDD (Resilient Distributed Dataset)** — low-level distributed data structure (rarely used directly)
- **DataFrame** — high-level, SQL-like API (most common for data engineering)
- **SparkSQL** — run SQL queries on Spark DataFrames
- **Transformation vs Action** — transformations (filter, select, join) are lazy; actions (count, write, show) trigger execution

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum as spark_sum

spark = SparkSession.builder.appName("SalesETL").getOrCreate()

# Read Parquet
df = spark.read.parquet("/data/sales/")

# Transform
result = (df
    .filter(col("status") == "complete")
    .groupBy("region")
    .agg(spark_sum("revenue").alias("total_revenue"))
    .orderBy("total_revenue", ascending=False)
)

# Write
result.write.mode("overwrite").parquet("/data/output/revenue_by_region/")
```

---

**Q33. What is Apache Airflow and how does it work?**

Apache Airflow is an open-source workflow orchestration platform. You write pipelines as Python DAGs, and Airflow schedules and monitors their execution.

Core components:
- **DAG** — the pipeline definition (Python file)
- **Task** — a single unit of work inside a DAG
- **Operator** — the type of task (PythonOperator, BashOperator, SQLOperator, etc.)
- **Scheduler** — triggers DAGs based on schedule or dependency
- **Executor** — runs the tasks (LocalExecutor, CeleryExecutor, KubernetesExecutor)
- **XCom** — mechanism to pass data between tasks

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def extract(): print("Extracting data...")
def transform(): print("Transforming data...")
def load(): print("Loading data...")

with DAG('daily_pipeline', start_date=datetime(2024, 1, 1), schedule_interval='@daily', catchup=False) as dag:
    t1 = PythonOperator(task_id='extract', python_callable=extract)
    t2 = PythonOperator(task_id='transform', python_callable=transform)
    t3 = PythonOperator(task_id='load', python_callable=load)
    t1 >> t2 >> t3  # Define execution order
```

---

**Q34. What is Apache Kafka?**

Apache Kafka is a distributed event streaming platform. Producers publish messages (events) to topics; consumers read from those topics independently and at their own pace.

Key concepts:
- **Topic** — a named stream of events (like a table for streaming data)
- **Partition** — topics are split into partitions for parallelism
- **Offset** — the position of a message within a partition (consumers track this)
- **Consumer Group** — multiple consumers sharing work; each partition is consumed by one member at a time
- **Retention** — Kafka stores messages for a configurable period (default 7 days)

```
Producer (App / IoT Device)
        ↓
   Kafka Topic (partitioned)
        ↓
   Consumer (Spark Streaming / Flink)
        ↓
   Data Lake / Database
```

> **Beginner tip:** Kafka is not a database — it is a message log. Use it to decouple data producers from consumers and buffer high-velocity event streams.

---

**Q35. What is dbt (data build tool)?**

dbt is an open-source transformation tool that brings software engineering best practices to SQL-based data transformations inside a data warehouse.

What dbt does:
- Runs SQL `SELECT` statements as models and materialises them as tables or views
- Manages dependencies between models automatically
- Runs data quality tests (`not_null`, `unique`, `accepted_values`, custom SQL)
- Generates documentation and a lineage graph
- Integrates with Git for version control

```sql
-- models/silver/orders_cleaned.sql
SELECT
    order_id,
    customer_id,
    UPPER(status) AS status,
    order_date::DATE AS order_date,
    amount
FROM {{ ref('bronze_orders') }}   -- reference another dbt model
WHERE order_id IS NOT NULL
```

```yaml
# schema.yml — define tests
models:
  - name: orders_cleaned
    columns:
      - name: order_id
        tests:
          - not_null
          - unique
      - name: status
        tests:
          - accepted_values:
              values: ['COMPLETE', 'PENDING', 'CANCELLED']
```

---

**Q36. What is Docker and why do data engineers use it?**

Docker is a platform for packaging applications and their dependencies into containers — isolated, portable environments that run the same way everywhere.

Why data engineers use it:
- Run Airflow, Kafka, Spark locally for development without complex setup
- Package pipeline code with its exact Python dependencies for reproducible runs
- Deploy pipelines to cloud environments with consistent behaviour

```dockerfile
# Example Dockerfile for a Python pipeline
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY pipeline.py .
CMD ["python", "pipeline.py"]
```

---

**Q37. What is Git and why is version control important in data engineering?**

Git is the standard version control system. In data engineering:
- Track changes to pipeline code, SQL models, and configuration files
- Collaborate with team members without overwriting each other's work
- Roll back to a working version when a deployment breaks something
- Implement CI/CD — automatically test and deploy pipelines when code is merged

```bash
git init                          # initialise a repository
git add pipeline.py               # stage changes
git commit -m "Add incremental load logic"   # save snapshot
git push origin main              # push to remote (GitHub/GitLab/Azure DevOps)

git checkout -b feature/new-etl   # create a feature branch
git merge feature/new-etl         # merge changes back to main
```

> **Best practice:** Never push database credentials, API keys, or secrets to Git. Use `.gitignore` and environment variables.

---

**Q38. What is the difference between Parquet and CSV? Which should you use?**

| | CSV | Parquet |
|---|---|---|
| Format | Row-based, plain text | Columnar, binary |
| Compression | None (or gzip) | Built-in (Snappy, Gzip, Zstd) |
| Schema | None (all strings) | Embedded schema with types |
| Query speed | Slow (scan all columns) | Fast (scan only needed columns) |
| File size | Large | 5–10x smaller than CSV |
| Human readable | Yes | No |

```python
# Always save to Parquet in data lake pipelines
df.to_parquet('output.parquet', compression='snappy', index=False)

# Read Parquet (much faster than CSV for analytics)
df = pd.read_parquet('output.parquet', columns=['order_id', 'revenue'])
```

> **Rule:** Use CSV only for source data ingestion or sharing with non-technical stakeholders. Use Parquet for everything inside your data lake and warehouse pipelines.

---

# Quick Reference — Key Concepts Cheat Sheet

| Concept | One-line definition |
|---|---|
| ETL | Extract → Transform → Load (transform before loading) |
| ELT | Extract → Load → Transform (transform inside the warehouse) |
| DAG | Directed Acyclic Graph — pipeline task dependency map |
| Idempotency | Safe to re-run a pipeline multiple times with the same result |
| Partitioning | Organise data files by a key (date) to speed up queries |
| Star schema | Fact table + dimension tables for warehouse data modelling |
| SCD Type 2 | Track historical changes by adding new rows with validity dates |
| Delta Lake | ACID transactions + time travel on top of a data lake |
| Window function | Calculation across related rows without collapsing them |
| Data lineage | Tracking where data came from and how it was transformed |

---

*Last updated: June 2026*
