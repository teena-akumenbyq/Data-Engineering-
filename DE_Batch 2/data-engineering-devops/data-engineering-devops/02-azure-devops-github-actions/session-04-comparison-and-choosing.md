# Session 04 — Azure DevOps vs GitHub Actions: Choosing

## Side-by-Side
| Feature | Azure DevOps Pipelines | GitHub Actions |
|---------|------------------------|----------------|
| Config file | `azure-pipelines.yml` | `.github/workflows/*.yml` |
| Repo hosting | Azure Repos (or GitHub) | GitHub |
| Reusability | Templates | Reusable workflows + Actions |
| Marketplace | Task marketplace | Huge Actions marketplace |
| Environments/Approvals | Built-in, mature | Environments + protection rules |
| Package feeds | Azure Artifacts | GitHub Packages |
| Work tracking | Azure Boards (strong) | GitHub Issues/Projects |
| Best for | Enterprise, Azure-heavy shops | GitHub-centric teams, OSS |

## Key Terminology Mapping
| Concept | Azure DevOps | GitHub Actions |
|---------|--------------|----------------|
| Pipeline | Pipeline | Workflow |
| Stage | Stage | Job (grouped) |
| Job | Job | Job |
| Step | Step / Task | Step |
| Agent | Agent | Runner |
| Secret store | Variable Group / Key Vault | Actions Secrets |
| Reusable block | Template | Reusable workflow / Action |

## When to Choose Azure DevOps
- Deep Azure integration and enterprise governance.
- Need mature Boards + Artifacts + approvals in one place.
- Organization already standardized on Azure DevOps.

## When to Choose GitHub Actions
- Code already lives on GitHub.
- Want the large community Actions marketplace.
- Open-source or GitHub-native workflows.

## Both Can Deploy to Azure
Both use a **service principal** (via `azure/login` or a service connection) to deploy ADF, Databricks, AKS, and Terraform.

## Practical Guidance for Data Teams
- Repo on GitHub → GitHub Actions is the path of least friction.
- Repo on Azure Repos + heavy Azure infra → Azure Pipelines.
- Either way: pipeline-as-code, environments, approvals, secrets in a vault.

## Lab
1. Take the dbt CI from Session 3 and rewrite it as an Azure Pipeline.
2. Note which concepts map 1:1 and which differ.
