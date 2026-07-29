# Session 03 — GitHub Actions

## What is GitHub Actions?
CI/CD built into GitHub. Workflows are YAML files in `.github/workflows/`.

## Core Concepts
| Term | Meaning |
|------|---------|
| **Workflow** | The whole automated process (one YAML file) |
| **Event** | What triggers it (`push`, `pull_request`, `schedule`) |
| **Job** | A set of steps on one runner |
| **Step** | A single task or shell command |
| **Action** | A reusable unit (`actions/checkout@v4`) |
| **Runner** | The machine executing the job |

## Basic Workflow
```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest
```

## Data Pipeline Workflow (dbt example)
```yaml
name: dbt-ci
on:
  pull_request:
    branches: [ main ]
jobs:
  dbt-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install dbt-snowflake
      - run: dbt deps
      - run: dbt compile
      - run: dbt test
        env:
          DBT_PASSWORD: ${{ secrets.DBT_PASSWORD }}
```

## Secrets
- Repo → Settings → Secrets and variables → Actions.
- Reference as `${{ secrets.MY_SECRET }}`.
- Never echo secrets in logs.

## Matrix Builds
```yaml
strategy:
  matrix:
    python-version: ['3.10', '3.11', '3.12']
```

## Scheduled Runs (great for data jobs)
```yaml
on:
  schedule:
    - cron: '0 6 * * *'   # daily 06:00 UTC
```

## Deploy to Azure from GitHub Actions
```yaml
      - uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      - run: az deployment group create ...
```

## Lab
1. Add a `ci.yml` that runs `pytest` on every PR.
2. Add a scheduled workflow that runs daily.
3. Store one secret and consume it in a step.
