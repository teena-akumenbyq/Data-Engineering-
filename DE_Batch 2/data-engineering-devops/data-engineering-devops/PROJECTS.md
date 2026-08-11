# Simple Projects — DevOps for Data Engineering

Small, buildable projects. Each maps to a topic and can be done in a session or two.
Difficulty: ⭐ easy · ⭐⭐ medium.

---

## Topic 01 — Git, Version Control & CI/CD

### Project 1.1 — Version a Mini ETL Script ⭐
**Goal:** Practice the full Git loop on a real data script.
- Write `extract.py` that reads a CSV and prints row count.
- `git init`, add a `.gitignore` (ignore `__pycache__/`, `.env`, `*.csv`).
- Make 3 meaningful commits as you add: extract → transform → load functions.
- Tag it `v1.0.0`.
**You learn:** init, add, commit, .gitignore, tags.

### Project 1.2 — Branch-and-Merge a New Feature ⭐
**Goal:** Simulate team development.
- From `main`, branch `feature/add-null-check`.
- Add data-validation code, commit.
- Edit the same lines on `main` to force a conflict, then merge and resolve.
**You learn:** branching, merging, conflict resolution.

### Project 1.3 — GitHub Repo with a Test Workflow ⭐⭐
**Goal:** First taste of CI.
- Push the ETL script to GitHub.
- Add a `pytest` test for the transform function.
- Add `.github/workflows/ci.yml` that runs `pytest` on every PR.
- Open a PR with a broken test → watch CI fail → fix → watch it pass.
**You learn:** remotes, PRs, CI feedback loop.

---

## Topic 02 — Azure DevOps / GitHub Actions

### Project 2.1 — dbt CI on Pull Requests ⭐⭐
**Goal:** Real data-pipeline CI.
- Small dbt project with 2 models + a `not_null` / `unique` test.
- GitHub Actions workflow: `dbt deps` → `dbt compile` → `dbt test` on PRs.
- Store the warehouse password as a repo secret.
**You learn:** secrets, dbt in CI, quality gates.

### Project 2.2 — Scheduled Data Job ⭐
**Goal:** Automate a recurring pipeline.
- A Python script that fetches from a public API and writes JSON.
- GitHub Actions with a `cron` schedule (daily).
- Upload the output as a workflow artifact.
**You learn:** scheduled triggers, artifacts.

### Project 2.3 — Two-Stage Azure Pipeline ⭐⭐
**Goal:** Build → Deploy with an approval.
- `azure-pipelines.yml` with **Build** (test) and **DeployDev** stages.
- Add a variable group for config.
- Add a manual approval on a `prod` environment.
**You learn:** stages, environments, approvals.

### Project 2.4 — Package a Shared Python Lib ⭐⭐
**Goal:** Reusable code for pipelines.
- Build a small utility package (e.g. logging + config helpers).
- Publish to GitHub Packages or Azure Artifacts.
- Consume it from another repo's pipeline.
**You learn:** artifact feeds, versioning.

---

## Topic 03 — Release & Deployment Strategies

### Project 3.1 — Blue-Green Tables ⭐⭐
**Goal:** Zero-downtime data swap.
- Two tables `sales_blue` and `sales_green` + a view `sales` pointing to one.
- Load new data into the idle table, then repoint the view.
- Practice rolling back by repointing.
**You learn:** blue-green pattern for data.

### Project 3.2 — Canary dbt Model ⭐⭐
**Goal:** Validate before cutover.
- Run a new version of a model as `model_v2` alongside `model_v1`.
- Diff row counts and key aggregates.
- Promote v2 only if the diff is within tolerance.
**You learn:** canary / shadow runs, comparison checks.

### Project 3.3 — Gated Prod Deploy with Rollback ⭐⭐
**Goal:** Safe promotion path.
- Pipeline: dev → staging → prod, prod behind manual approval.
- Add a post-deploy data-quality check (row count / freshness) that fails the run.
- Script a one-command rollback to the previous artifact/tag.
**You learn:** promotion, gates, rollback automation.

---

## Capstone (ties all three) ⭐⭐
**Mini Data Platform CI/CD**
1. Repo with a small ingestion script + dbt models (Topic 1).
2. CI on PRs: lint, pytest, dbt test (Topic 2).
3. CD: deploy to dev automatically, prod behind approval, with a blue-green table swap and rollback (Topic 3).

Build it once end-to-end and you've exercised every concept in the course.

---

## Suggested Order
1.1 → 1.2 → 1.3 → 2.1 → 2.2 → 3.1 → then pick 2.3 / 3.3 → Capstone.
