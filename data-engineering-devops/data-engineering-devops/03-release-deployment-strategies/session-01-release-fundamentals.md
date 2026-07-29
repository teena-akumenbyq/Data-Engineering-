# Session 01 — Release & Deployment Fundamentals

## Release vs Deployment
- **Deployment** = putting code/artifacts onto an environment.
- **Release** = making that functionality available to users.
- They can be decoupled (deploy dark, release later via a flag).

## Environment Promotion Path
```
dev  →  staging (UAT)  →  prod
```
| Environment | Purpose |
|-------------|---------|
| dev | Fast iteration, integration tests |
| staging | Prod-like, UAT, full data tests |
| prod | Live, gated by approval |

## Artifacts & Immutability
Build **once**, promote the **same artifact** through each environment (wheel, container image, dbt package). Never rebuild per environment — it breaks reproducibility.

## Approval Gates
- Manual approval before `prod`.
- Automated gates: tests pass, data quality checks pass, security scan clean.

## Rollback Basics
Always have a way back:
- Redeploy previous artifact/tag.
- `terraform apply` a previous state.
- Restore previous dbt/ADF version.
- Database changes: use reversible migrations.

## Configuration Management
- Same code, different config per environment.
- Config from variable groups / secrets / Key Vault — never hardcoded.

## Data-Engineering-Specific Concerns
- **Schema migrations** must be backward-compatible during rollout.
- **Backfills** should be idempotent and re-runnable.
- **Stateful pipelines**: consider in-flight data during deploys.

## Lab
1. Draw your dev→staging→prod path for a dbt project.
2. List the gates required before prod.
3. Define the rollback step for a failed prod deploy.
