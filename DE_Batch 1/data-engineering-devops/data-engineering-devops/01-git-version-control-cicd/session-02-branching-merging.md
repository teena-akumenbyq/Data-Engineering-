# Session 02 — Branching & Merging

## Why Branch?
A branch is an isolated line of development. Data teams use branches to build a new pipeline or model without breaking `main`, then merge when tested.

## Branch Commands
```bash
git branch                   # list branches
git branch feature/new-etl   # create branch
git switch feature/new-etl   # move to branch (modern)
git switch -c feature/new-etl # create + switch
git branch -d feature/new-etl # delete merged branch
```
> `git checkout` still works but `git switch` / `git restore` are the modern, clearer commands.

## Merging
```bash
git switch main
git merge feature/new-etl    # bring feature into main
```

### Fast-forward vs Three-way
- **Fast-forward**: main hasn't moved — pointer just advances.
- **Three-way merge**: both branches changed — Git creates a merge commit.

## Handling Merge Conflicts
When two branches change the same lines:
```
<<<<<<< HEAD
current branch code
=======
incoming branch code
>>>>>>> feature/new-etl
```
Edit the file, remove the markers, keep the correct code, then:
```bash
git add conflicted_file.py
git commit
```

## Rebase (linear history)
```bash
git switch feature/new-etl
git rebase main              # replay your commits on top of main
```
> **Golden rule:** never rebase a branch that others have already pulled.

## Common Branching Strategy for Data Teams
```
main        → production-ready
develop     → integration
feature/*   → individual pipelines/models
hotfix/*    → urgent prod fixes
```

## Lab
1. From `main`, create `feature/add-transform`.
2. Add a transformation function, commit.
3. Switch to `main`, edit the same file to force a conflict.
4. Merge, resolve the conflict, and commit.
