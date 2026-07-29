# Session 01 — Azure DevOps Overview

## What is Azure DevOps?
A suite of services for the full software/data delivery lifecycle. Five core services:

| Service | Purpose | Data Engineering Use |
|---------|---------|----------------------|
| **Azure Repos** | Git repositories | Version dbt, PySpark, ADF, Terraform |
| **Azure Pipelines** | CI/CD | Automate deploy dev→staging→prod |
| **Azure Boards** | Work tracking (Agile) | Sprint planning, backlog |
| **Azure Artifacts** | Package feeds | Host Python wheels, shared libs |
| **Azure Test Plans** | Manual/exploratory testing | Less common for data teams |

> For data engineering, **Azure Repos + Azure Pipelines** are the non-negotiable core.

## Organizations and Projects
```
Organization
 └── Project
      ├── Repos
      ├── Pipelines
      ├── Boards
      └── Artifacts
```

## Azure Repos
- Full Git support + PRs with branch policies.
- Branch policies can require: min reviewers, linked work item, successful build, no active comments.

## Azure Pipelines (two flavors)
- **YAML pipelines** (recommended) — pipeline as code in `azure-pipelines.yml`.
- **Classic pipelines** — UI-based, legacy.

## Minimal YAML Pipeline
```yaml
trigger:
  - main

pool:
  vmImage: ubuntu-latest

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '3.11'
  - script: |
      pip install -r requirements.txt
      pytest
    displayName: 'Install & Test'
```

## Service Connections
Securely link a pipeline to Azure resources (subscription, ACR, Databricks) using a **service principal** — no credentials in code.

## Lab
1. Create a free Azure DevOps org and project.
2. Push a repo to Azure Repos.
3. Add a branch policy requiring one reviewer on `main`.
