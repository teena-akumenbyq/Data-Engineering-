# Session 03 — Implementing Releases & Monitoring

## Environments in Pipelines
Both Azure Pipelines and GitHub Actions support **environments** with protection rules (approvals, wait timers, allowed branches).

### Azure Pipelines — gated prod deploy
```yaml
- stage: DeployProd
  dependsOn: DeployStaging
  jobs:
    - deployment: Prod
      environment: 'prod'        # add approval check in UI
      strategy:
        runOnce:
          deploy:
            steps:
              - script: terraform apply -auto-approve
```

### GitHub Actions — environment approval
```yaml
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment: production      # add required reviewers in repo settings
    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh
```

## Blue-Green with Terraform (pattern)
- Maintain two resource sets or two slots.
- Use a pointer (App Service slot swap, view, or DNS) to cut over.
```bash
az webapp deployment slot swap --slot green --target-slot production
```

## Canary with Traffic Split (AKS example)
- Deploy v2 alongside v1.
- Use ingress / service mesh to send 5% traffic to v2.

## Post-Deploy Verification
- Smoke tests / health checks.
- Data quality checks (row counts, null ratios, freshness).
- Compare canary output vs baseline.

## Monitoring & Observability
| Layer | Tool |
|-------|------|
| Infra/app metrics | Azure Monitor, Application Insights |
| Logs | Log Analytics |
| Data quality | Great Expectations, dbt tests |
| Pipeline runs | ADF monitoring, Databricks jobs UI |
| Alerts | Azure Alerts → Teams/email/PagerDuty |

## Rollback Automation
- Keep last-known-good artifact tag.
- One-command redeploy of previous version.
- For blue-green: swap back to previous slot.

## Lab
1. Add an approval-gated prod stage to your pipeline.
2. Add a post-deploy data-quality check that fails the run on bad data.
3. Configure one alert on pipeline failure.
