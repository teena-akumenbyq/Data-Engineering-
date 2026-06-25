# Azure Data Engineering Interview Preparation
---

## 1. Azure Core Services

---

**Q1. What is Azure Data Factory (ADF)?**

Azure Data Factory is Azure's cloud-based ETL and data integration service. It allows you to create, schedule, and orchestrate data pipelines that move and transform data across different sources and destinations — without writing infrastructure code.

Key concepts:
- **Pipeline** — a logical grouping of activities
- **Activity** — a single step (e.g., copy data, run a stored procedure)
- **Dataset** — a reference to the data you want to use (input or output)
- **Linked Service** — connection string to a data source (like a database or storage account)
- **Integration Runtime** — the compute used to run the pipeline (Azure, self-hosted, or SSIS)

> **Note:** Think of ADF as the "Airflow of Azure" — it schedules and orchestrates data movement, but does not process large data itself (that's Spark/Databricks).

---

**Q2. What is Azure Blob Storage vs Azure Data Lake Storage Gen2 (ADLS Gen2)?**

| Feature | Azure Blob Storage | ADLS Gen2 |
|---|---|---|
| Use case | General object storage | Big data analytics |
| File system | Flat (no hierarchy) | Hierarchical namespace |
| Access control | Container-level | Fine-grained ACLs per folder/file |
| Performance | Standard | Optimised for analytics (Hadoop compatible) |
| Best for | Images, backups, logs | Raw data lakes, Spark/Databricks workloads |

ADLS Gen2 is built **on top of** Blob Storage — it adds a hierarchical namespace and fine-grained access control. For data engineering, you almost always use ADLS Gen2.

---

**Q3. What is Azure Synapse Analytics?**

Azure Synapse Analytics is an all-in-one analytics platform that combines:
- **Dedicated SQL Pool** (formerly SQL Data Warehouse) — for large-scale structured queries
- **Serverless SQL Pool** — query data in ADLS Gen2 directly using SQL without loading it first
- **Apache Spark Pools** — distributed big data processing
- **Synapse Pipelines** — built-in orchestration (similar to ADF)
- **Synapse Link** — real-time integration with Azure Cosmos DB

> **Note:** Synapse is Azure's answer to Snowflake + Databricks + Airflow in one unified workspace.

---

**Q4. What is Azure Databricks?**

Azure Databricks is a managed Apache Spark platform built on top of Azure. It is used for large-scale data transformation, machine learning, and streaming pipelines.

Key features:
- Collaborative notebooks (Python, SQL, Scala, R)
- Auto-scaling Spark clusters
- Delta Lake support (ACID transactions on data lake)
- Unity Catalog for data governance
- Native integration with ADLS Gen2, ADF, and Synapse

> **Note:** Use Databricks when you need to process large volumes of data that SQL alone cannot handle efficiently.

---

**Q5. What is Azure SQL Database vs Azure SQL Managed Instance?**

| Feature | Azure SQL Database | Azure SQL Managed Instance |
|---|---|---|
| Type | PaaS (fully managed) | PaaS with near-full SQL Server compatibility |
| SQL Server compatibility | ~95% | ~100% |
| Use case | New cloud-native apps | Migrating on-prem SQL Server to cloud |
| Cross-database queries | No | Yes |
| SQL Agent | No | Yes |

For data engineering, Azure SQL Database is commonly used as a staging or operational database feeding into Synapse or ADLS.

---

## 2. Azure Data Pipeline Concepts

---

**Q6. What is the Medallion Architecture in Azure?**

The Medallion Architecture is a data lake design pattern with three layers:

- **Bronze (Raw)** — raw, unprocessed data landed from source systems. Stored in ADLS Gen2. Never modified after ingestion.
- **Silver (Cleansed)** — data is cleaned, deduplicated, and filtered. Conformed to a standard schema.
- **Gold (Curated)** — business-ready aggregated data optimised for reporting and analytics (fact/dimension tables).

In Azure this is typically implemented with:
- ADF or Synapse Pipelines for ingestion (Bronze)
- Azure Databricks or Spark in Synapse for transformation (Silver, Gold)
- Azure Synapse Dedicated SQL Pool or Power BI for consumption (Gold)

---

**Q7. How does data flow in a typical Azure data pipeline?**

A common beginner-level Azure pipeline looks like this:

```
Source (SQL DB / REST API / CSV)
        ↓
  Azure Data Factory (ADF)
        ↓
  ADLS Gen2 - Bronze layer (raw files)
        ↓
  Azure Databricks / Synapse Spark
        ↓
  ADLS Gen2 - Silver layer (cleaned)
        ↓
  Azure Synapse / Delta Lake
        ↓
  Gold layer → Power BI / Reporting
```

---

**Q8. What is Delta Lake and why is it important?**

Delta Lake is an open-source storage layer that adds reliability to data lakes. It is the default table format in Azure Databricks.

Key features:
- **ACID transactions** — safe reads/writes even with concurrent operations
- **Schema enforcement** — rejects data that doesn't match the table schema
- **Time travel** — query previous versions of data using `VERSION AS OF` or `TIMESTAMP AS OF`
- **Upserts (MERGE)** — update existing rows or insert new ones in one statement
- **Compaction** — combines small files into larger ones for better read performance

> **Note:** Without Delta Lake, data lakes suffer from partial writes and no transaction guarantees. Delta Lake fixes that.

---

**Q9. What is the difference between Batch and Streaming in Azure?**

| | Batch | Streaming |
|---|---|---|
| When data is processed | At scheduled intervals | Continuously as data arrives |
| Latency | Minutes to hours | Milliseconds to seconds |
| Azure tools | ADF, Synapse, Databricks (scheduled) | Azure Event Hubs, Azure Stream Analytics, Databricks Structured Streaming |
| Use case | Daily sales report, monthly billing | Real-time fraud detection, live dashboards |

---

**Q10. What is Azure Event Hubs?**

Azure Event Hubs is a fully managed real-time data ingestion service (message queue). It can ingest millions of events per second from IoT devices, applications, or click streams.

Key concepts:
- **Producer** — sends events to Event Hubs
- **Consumer Group** — independent readers of the event stream
- **Partition** — unit of parallelism; events are distributed across partitions
- **Retention** — events are retained for 1–7 days by default

It is the Azure equivalent of Apache Kafka and can integrate with Databricks Structured Streaming or Azure Stream Analytics for downstream processing.

---

## 3. SQL on Azure

---

**Q11. How do you query data in ADLS Gen2 without loading it?**

Using **Synapse Serverless SQL Pool**, you can query files directly in ADLS Gen2 using `OPENROWSET`:

```sql
SELECT *
FROM OPENROWSET(
    BULK 'https://<storage>.dfs.core.windows.net/bronze/sales/*.parquet',
    FORMAT = 'PARQUET'
) AS sales_data;
```

This is useful for ad-hoc exploration of raw data without the cost of loading it into a dedicated SQL pool first.

---

**Q12. What file formats are commonly used in Azure data lakes?**

| Format | Description | Best for |
|---|---|---|
| **Parquet** | Columnar, compressed | Analytics, Spark, Synapse — preferred format |
| **Delta** | Parquet + transaction log | Lakehouse workloads in Databricks |
| **CSV** | Plain text, row-based | Simple ingestion, interoperability |
| **JSON** | Semi-structured | API responses, nested data |
| **Avro** | Row-based, schema embedded | Kafka/Event Hubs message serialisation |

> **Note:** Always convert CSV/JSON to Parquet after landing in Bronze. Parquet is 10x faster for analytical queries and uses far less storage.

---

**Q13. What is PolyBase in Azure Synapse?**

PolyBase is a technology that allows Azure Synapse Dedicated SQL Pool to query external data in ADLS Gen2 or Blob Storage using standard T-SQL — without copying the data into Synapse first. It is used to load data into Synapse via `CREATE EXTERNAL TABLE` or `COPY INTO`.

```sql
COPY INTO dbo.sales_fact
FROM 'https://<storage>.dfs.core.windows.net/gold/sales/'
WITH (FILE_TYPE = 'PARQUET');
```

---

## 4. Azure Security & Governance

---

**Q14. What is Azure Active Directory (Azure AD / Entra ID)?**

Azure Active Directory (now called Microsoft Entra ID) is Azure's identity and access management service. In data engineering:

- **Service Principals** — application identities used by pipelines and scripts to authenticate to Azure services (e.g., ADF accessing ADLS Gen2)
- **Managed Identity** — an automatically managed identity for Azure resources; no credentials to manage
- **RBAC (Role-Based Access Control)** — assign roles like Storage Blob Data Contributor to control access to resources

> **Note:** Always use Managed Identity or Service Principal for pipeline authentication — never hardcode credentials in code.

---

**Q15. What is Azure Key Vault?**

Azure Key Vault is a secure store for secrets, keys, and certificates. In data pipelines:
- Store database passwords, API keys, and connection strings in Key Vault
- ADF and Databricks can reference Key Vault secrets at runtime — the secret value is never exposed in your code or pipeline configuration
- Access is controlled via Azure AD and audit logs track every access

---

**Q16. What is Microsoft Purview?**

Microsoft Purview is Azure's data governance and cataloguing service. It provides:
- **Data discovery** — scan and catalogue data assets across Azure, on-prem, and multi-cloud
- **Data lineage** — visualise how data flows from source to destination through pipelines
- **Data classification** — automatically tag sensitive data (PII, financial data)
- **Business glossary** — define standard business terms across the organisation

---

## 5. Common Interview Questions

---

**Q17. Explain the difference between ADF and Azure Databricks. When would you use each?**

| | Azure Data Factory | Azure Databricks |
|---|---|---|
| Primary role | Orchestration & data movement | Data transformation & processing |
| Processing power | Low (uses connectors) | High (distributed Spark) |
| Code | Low-code / GUI | Python, SQL, Scala notebooks |
| Best for | Copying data, scheduling | Complex transformations, ML, large datasets |

**Use ADF** to move data between systems and trigger jobs on a schedule.  
**Use Databricks** when you need to transform or process large volumes of data that SQL cannot handle.  
In practice, ADF and Databricks are used **together** — ADF orchestrates the pipeline and triggers Databricks notebooks for heavy transformation.

---

**Q18. What happens when an ADF pipeline fails? How do you handle it?**

1. **Check the Monitor tab** in ADF — it shows a visual breakdown of which activity failed and the error message
2. **Common causes:** wrong connection string, source schema change, file not found, timeout
3. **Retry policies** — each activity in ADF can be configured with a retry count and interval
4. **Alerting** — configure Azure Monitor alerts or email notifications on pipeline failure
5. **Re-run** — ADF allows you to re-run a failed pipeline from the failed activity without re-running the entire pipeline

---

**Q19. What is partitioning in Azure data lakes and why does it matter?**

Partitioning is the practice of organising files in ADLS Gen2 into folder hierarchies based on a key (usually date):

```
/bronze/sales/year=2024/month=01/day=15/data.parquet
/bronze/sales/year=2024/month=01/day=16/data.parquet
```

Benefits:
- **Query performance** — Spark and Synapse only scan the relevant partitions (partition pruning)
- **Cost reduction** — fewer files scanned = less compute cost
- **Pipeline efficiency** — incremental loads only process the latest partition

---

**Q20. How would you load data incrementally in ADF?**

Incremental loading means only processing new or changed records since the last run — not the full dataset. Common approaches in ADF:

- **Watermark pattern** — store the last loaded timestamp in a control table. Each run queries only records where `modified_date > last_watermark`, then updates the watermark after success.
- **Change Data Capture (CDC)** — use Azure SQL's built-in CDC to capture row-level inserts, updates, and deletes. ADF reads from the CDC tables.
- **File date filter** — in ADLS Gen2, filter files by their `LastModified` metadata using ADF's wildcard or `modifiedDatetimeStart`/`modifiedDatetimeEnd` parameters.

---

**Q21. What is Unity Catalog in Azure Databricks?**

Unity Catalog is Databricks' centralised data governance layer. It provides:
- A single place to manage access to tables, files, and models across all Databricks workspaces
- Fine-grained permissions at the table, column, and row level
- Data lineage tracking
- Integration with Microsoft Purview

For beginners: think of it as the permissions and cataloguing system for your Databricks data assets.

---

**Q22. What is the COPY INTO command in Synapse vs INSERT INTO?**

`COPY INTO` is the recommended way to bulk-load data into Azure Synapse Dedicated SQL Pool from ADLS Gen2. It is faster, simpler, and more flexible than the older `INSERT INTO ... SELECT` from external tables.

```sql
-- Recommended: COPY INTO
COPY INTO dbo.fact_orders
FROM 'https://<storage>.dfs.core.windows.net/gold/orders/*.parquet'
WITH (FILE_TYPE = 'PARQUET', CREDENTIAL = (IDENTITY = 'Managed Identity'));

-- Older approach: INSERT INTO from external table
INSERT INTO dbo.fact_orders
SELECT * FROM dbo.ext_orders;
```

`COPY INTO` supports Parquet, CSV, ORC, and JSON and handles authentication natively via Managed Identity.

---

## 6. Quick Reference — Azure Services Map

| Category | Azure Service | Equivalent concept |
|---|---|---|
| Orchestration | Azure Data Factory | Apache Airflow |
| Object storage | ADLS Gen2 / Blob Storage | AWS S3 |
| Data warehouse | Azure Synapse Dedicated SQL Pool | Snowflake / Redshift |
| Big data processing | Azure Databricks | Spark on EMR |
| Streaming ingest | Azure Event Hubs | Apache Kafka |
| Stream processing | Azure Stream Analytics | Apache Flink |
| Secrets management | Azure Key Vault | AWS Secrets Manager |
| Data governance | Microsoft Purview | AWS Glue Data Catalog |
| NoSQL database | Azure Cosmos DB | DynamoDB / MongoDB |
| Relational database | Azure SQL Database | AWS RDS |

---
