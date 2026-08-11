# Linked Services & Datasets in Azure Data Factory

These two are the **foundation blocks** of ADF. Before you can move or transform any data, you need to tell ADF **where the data lives** and **what that data looks like**. That's exactly what Linked Services and Datasets do.

---

## 1. Linked Services

### What is it?
A **Linked Service** is like a **connection string / address book entry**. It tells ADF **how to connect** to a data source or destination — including the server location, credentials, and authentication method.

Think of it like saving a contact's phone number in your phone. You're not talking to them yet — you're just storing *how to reach them*.

### Why is it used?
- To securely store connection details (server name, username, password, keys, tokens)
- To reuse the same connection across multiple pipelines/datasets without re-entering details
- To connect to different types of systems: databases, cloud storage, SaaS apps, APIs

### Examples of Linked Services
| Type | Example |
|---|---|
| Database | Azure SQL Database, MySQL, Oracle |
| Storage | Azure Blob Storage, Azure Data Lake, Amazon S3 |
| SaaS App | Salesforce, Dynamics 365 |
| File-based | SFTP, FTP, File System |
| Compute | Azure Databricks, HDInsight |

### Where is it created?
Under **Manage → Linked Services** (in ADF Studio).

### Key Point
👉 A Linked Service = **"Where is the data / how do I connect to it?"**

---

## 2. Datasets

### What is it?
A **Dataset** represents the **actual data structure** you want to work with — like a specific table, file, or folder — **within** a Linked Service.

Going back to the phone contact analogy: if the Linked Service is the contact's phone number, the Dataset is the **specific conversation/message thread** you have with them — pointing to exact content.

### Why is it used?
- To point to a specific table, file, container, or folder inside the connected source
- To define the data's schema/structure (column names, data types) — optional but helpful
- To be used as an **input** (source) or **output** (destination/sink) in a pipeline activity (like Copy Data)

### Examples of Datasets
| Linked Service | Dataset points to |
|---|---|
| Azure SQL Database | A specific table, e.g., `dbo.Customers` |
| Azure Blob Storage | A specific file, e.g., `sales_data.csv` |
| Data Lake | A specific folder path |
| Salesforce | A specific object, e.g., `Leads` |

### Where is it created?
Under **Author → Datasets** (in ADF Studio).

### Key Point
👉 A Dataset = **"What exact piece of data am I using from that connection?"**

---

## How They Work Together (Simple Flow)

```
Linked Service (connection to the server/storage)
        ↓
Dataset (specific table/file inside that connection)
        ↓
Activity in Pipeline (e.g., Copy Data — uses dataset as source/sink)
```

### Real-Life Example
Imagine you want to copy data from an Excel file in Google Drive to a SQL table.

1. **Linked Service 1** → Connects ADF to your Google Drive account (login/credentials)
2. **Dataset 1** → Points to the exact Excel file, e.g., `Students_2024.xlsx`
3. **Linked Service 2** → Connects ADF to your SQL Server (login/credentials)
4. **Dataset 2** → Points to the exact table, e.g., `dbo.Students`
5. **Pipeline (Copy Activity)** → Uses Dataset 1 as source and Dataset 2 as sink to move the data

---

## Quick Summary

| Concept | Analogy | Purpose |
|---|---|---|
| **Linked Service** | Saving someone's phone number | Defines the **connection** to a data source/destination |
| **Dataset** | The specific message/file you send them | Defines the **exact data** (table/file/folder) within that connection |

**One Linked Service can have many Datasets** — e.g., one connection to a SQL Server, but different datasets for different tables in it.
