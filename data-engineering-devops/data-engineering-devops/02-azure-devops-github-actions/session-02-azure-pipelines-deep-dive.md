# Session 02 — Azure Pipelines Deep Dive

## Pipeline Hierarchy
```
Pipeline
 └── Stages       (e.g. Build, DeployDev, DeployProd)
      └── Jobs     (run on an agent)
           └── Steps (tasks / scripts)
```

## Multi-Stage YAML Example
```yaml
trigger:
  - main

variables:
  pythonVersion: '3.11'

stages:
  - stage: Build
    jobs:
      - job: BuildAndTest
        pool: { vmImage: ubuntu-latest }
        steps:
          - task: UsePythonVersion@0
            inputs: { versionSpec: '$(pythonVersion)' }
          - script: |
              pip install -r requirements.txt
              pytest
              dbt deps && dbt compile
            displayName: 'Test & Compile'

  - stage: DeployDev
    dependsOn: Build
    jobs:
      - deployment: DeployToDev
        environment: 'dev'
        pool: { vmImage: ubuntu-latest }
        strategy:
          runOnce:
            deploy:
              steps:
                - script: terraform apply -auto-approve
                  displayName: 'Deploy Dev'
```

## Variables & Variable Groups
```yaml
variables:
  - group: 'data-platform-secrets'   # from Library
  - name: environment
    value: 'dev'
```
Store secrets in **Library → Variable Groups**, optionally linked to Azure Key Vault.

## Environments & Approvals
- Define `dev`, `staging`, `prod` environments.
- Add **approval checks** on `prod` — a human must approve before deploy.

## Templates (reuse)
```yaml
# build-template.yml
steps:
  - script: pytest
    displayName: 'Run tests'
```
```yaml
# azure-pipelines.yml
steps:
  - template: build-template.yml
```

## Self-hosted vs Microsoft-hosted Agents
- **Microsoft-hosted**: fresh VM each run, no maintenance.
- **Self-hosted**: your VM — needed for private networks (e.g. Databricks in a VNet).

## Lab
1. Build a 2-stage pipeline: Build → DeployDev.
2. Add a variable group linked to Key Vault.
3. Add a manual approval on a `prod` environment.
