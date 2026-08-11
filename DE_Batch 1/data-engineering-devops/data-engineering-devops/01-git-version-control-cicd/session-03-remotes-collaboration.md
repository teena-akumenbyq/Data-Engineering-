# Session 03 — Remotes & Collaboration

## Remotes
A remote is a hosted copy of your repo (GitHub, Azure Repos, GitLab).
```bash
git remote -v                        # list remotes
git remote add origin <url>          # link a remote
git push -u origin main              # first push, sets upstream
git push                             # subsequent pushes
git pull                             # fetch + merge
git fetch                            # download without merging
```

## Clone → Work → Push Loop
```bash
git clone <url>
git switch -c feature/my-work
# ... edit, add, commit ...
git push -u origin feature/my-work
```

## Pull Requests / Merge Requests
The core collaboration unit:
1. Push your feature branch.
2. Open a PR against `main`.
3. Teammates review + comment.
4. CI runs automatically (tests, linting).
5. Approve and merge (squash / merge commit / rebase).

## Keeping a Fork/Branch Updated
```bash
git fetch origin
git switch main
git pull origin main
git switch feature/my-work
git rebase main          # or: git merge main
```

## Undoing Things (safely)
```bash
git restore file.py               # discard working changes
git restore --staged file.py      # unstage
git revert <commit>               # new commit that undoes a commit (safe, shared)
git reset --soft HEAD~1           # undo commit, keep changes staged
git reset --hard HEAD~1           # DANGER: discard commit + changes
```

## Tagging Releases
```bash
git tag v1.0.0
git tag -a v1.0.0 -m "First prod release"
git push origin v1.0.0
```

## Lab
1. Push a feature branch to a remote.
2. Open a PR and self-review the diff.
3. Practice `git revert` on a bad commit.
4. Tag a release `v0.1.0` and push the tag.
