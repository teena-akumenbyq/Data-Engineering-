# Session 01 — Git Fundamentals

## What is Version Control?
Version control tracks changes to code over time, lets multiple people collaborate, and lets you roll back to any previous state. Git is a **distributed** VCS — every clone is a full repository with complete history.

## Why Data Engineers Need Git
- Version-control data pipeline code (dbt models, PySpark jobs, ADF templates)
- Track Terraform / IaC changes
- Collaborate on shared notebooks and SQL
- Enable CI/CD (nothing deploys without a commit)

## Initial Setup (run once per machine)
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"
git config --list
```

## Starting a Repository
```bash
git init                 # initialize a new repo in current folder
git clone <url>          # copy an existing remote repo
git status               # show working tree state
```

## The Three Areas
| Area | Description | Command to move here |
|------|-------------|----------------------|
| Working Directory | Files you edit | (edit files) |
| Staging Area (Index) | Changes marked for next commit | `git add` |
| Repository | Committed history | `git commit` |

## Core Daily Commands
```bash
git add file.py          # stage one file
git add .                # stage everything
git commit -m "message"  # commit staged changes
git log --oneline        # compact history
git diff                 # unstaged changes
git diff --staged        # staged changes
```

## The .gitignore File
Keep secrets and junk out of version control:
```
__pycache__/
*.pyc
.env
.terraform/
*.tfstate
.databricks/
spark-warehouse/
```

## Lab
1. Create a folder `de-pipeline`, run `git init`.
2. Add a `pipeline.py` and a `.gitignore`.
3. Stage and commit with a meaningful message.
4. Run `git log --oneline` and confirm your commit.
