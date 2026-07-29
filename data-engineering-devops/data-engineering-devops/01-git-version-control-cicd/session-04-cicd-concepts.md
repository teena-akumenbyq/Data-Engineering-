# Session 04 — CI/CD Concepts

## Definitions
- **CI (Continuous Integration):** every commit is automatically built and tested.
- **CD (Continuous Delivery):** every passing build is automatically prepared for release (deploy is a manual button).
- **CD (Continuous Deployment):** every passing build is automatically deployed to production.

## Why CI/CD for Data Engineering
- Catch broken dbt models or PySpark jobs before they hit prod.
- Enforce linting, unit tests, and schema checks on every PR.
- Deploy ADF pipelines, Databricks jobs, and Terraform consistently.
- Remove manual, error-prone deployments.

## Anatomy of a Pipeline
```
Trigger → Build → Test → Package → Deploy (dev → staging → prod)
```

| Stage | Data Engineering Example |
|-------|--------------------------|
| Trigger | push / PR to main |
| Build | install deps, compile dbt |
| Test | pytest, dbt test, great_expectations |
| Package | build wheel / container image |
| Deploy | terraform apply, deploy ADF, publish Databricks job |

## Key Concepts
- **Pipeline as code**: YAML lives in the repo (`.github/workflows`, `azure-pipelines.yml`).
- **Runners / Agents**: machines that execute jobs.
- **Artifacts**: build outputs passed between stages.
- **Secrets**: stored in the platform, never in code.
- **Environments**: dev / staging / prod with approval gates.

## Quality Gates for Data Pipelines
- Lint: `flake8`, `sqlfluff`
- Unit tests: `pytest`
- Data tests: `dbt test`, Great Expectations
- IaC validation: `terraform validate`, `terraform plan`

## Typical Trigger Rules
```yaml
# run on PRs to main, and on push to main
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
```

## Lab
1. Sketch the CI/CD stages for a dbt project on a whiteboard.
2. List which tests run at PR time vs at deploy time.
3. Identify where secrets are needed and where they should live.
