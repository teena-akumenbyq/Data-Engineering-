# Azure Data Factory – Interface Overview

When you open ADF Studio, you'll see a left-side navigation menu with these main sections:

## 1. Home
The landing page. Shows quick-start options like "Ingest data," "Transform data," "Configure SSIS," and recent/pinned pipelines. It's basically a shortcut hub to jump into common tasks.

## 2. Author
This is where you **build everything** — pipelines, datasets, dataflows. Think of it as the "workshop" or "design" area.

Inside Author, you create:
- **Pipelines** – A workflow made of connected steps (activities) that move/process data
- **Datasets** – Represents the data structure (a table, file, folder) you're working with
- **Dataflows** – Drag-and-drop visual designer for transforming data (no code needed)
- **Power Query** – Excel-like data prep experience

### Activities (inside a Pipeline)
Activities are the individual "steps" or "tasks" in a pipeline. Main types:

| Activity Type | What it does |
|---|---|
| **Copy Data** | Moves data from a source to a destination |
| **Data Flow** | Runs a transformation (clean, join, filter data) |
| **Lookup** | Reads a value/config from a dataset to use later in pipeline |
| **Get Metadata** | Fetches info about a file/folder (size, existence, schema) |
| **ForEach** | Loops through a list of items and repeats an activity |
| **If Condition** | Branches logic (like an if-else statement) |
| **Web Activity** | Calls an external API/REST endpoint |
| **Stored Procedure** | Runs a SQL stored procedure |
| **Execute Pipeline** | Calls/triggers another pipeline |
| **Wait** | Pauses the pipeline for a set time |
| **Set Variable** | Stores a value in a variable during pipeline run |

## 3. Monitor
This is where you **track pipeline runs** — like a CCTV camera for your data flows.

- See which pipelines **succeeded, failed, or are running**
- Check **trigger runs** (scheduled runs)
- View detailed logs/errors for debugging
- See execution time, duration, and activity-level status

## 4. Manage
This is the **settings/admin area** — where you set up connections and control access.

Key things here:
- **Linked Services** – Connection strings to external sources (like SQL Server, Blob Storage, Salesforce)
- **Integration Runtimes** – The compute engine that actually runs your data movement (Azure-hosted, self-hosted, or SSIS)
- **Triggers** – Set schedules (e.g., "run every day at 6 AM") or event-based triggers (e.g., "run when a file lands in a folder")
- **Git Configuration** – Connect ADF to GitHub/Azure DevOps for version control
- **Access Control** – Manage who can view/edit the factory

## 5. Learning Center
A built-in **help/tutorial hub** with guided walkthroughs, videos, and documentation links to help you learn ADF features without leaving the portal.

---

## Quick Summary

**Home** = Dashboard → **Author** = Build → **Monitor** = Track → **Manage** = Configure → **Learning Center** = Learn
