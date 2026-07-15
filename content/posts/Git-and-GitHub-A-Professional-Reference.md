+++
title = 'Git & GitHub: A Professional Reference'
date = 2026-04-19T16:16:00-04:00
draft = false
description = 'A practical, workflow-oriented Git & GitHub reference — the staging-area mental model, core concepts, everyday commands, branching, rebasing, .gitignore, handling leaked secrets, and resolving merge conflicts.'
+++

<!--more-->


### Mental Model First

Git has **three local layers** and **one remote**:

```
Working Tree   →   Index (Staging Area)   →   Local Repository   →   Remote (GitHub)
  (your files)        (what will commit)         (commit history)       (origin)

   edit files          git add                    git commit             git push
   ←─────────────────────────────────────────────────────────────────────────────
                       git restore --staged       git reset              git fetch/pull
```

The **Index** (also called "staging area") is the thing most beginners skip over. It is a snapshot of what your *next commit* will look like. You explicitly build it with `git add` before committing.

------

### Core Concepts

**Repository:** A directory that git tracks. Contains a hidden `.git/` folder with the entire history.

**Commit:** A permanent, immutable snapshot of the entire project at one point in time. Every commit has:

- A SHA hash (e.g., `daa978d`) — its unique ID
- A parent commit (or two, for merges)
- Author, timestamp, message
- A pointer to the project tree at that moment

**Branch:** A lightweight movable pointer to a commit. Creating a branch costs nothing — it's just a 41-byte file. The current branch advances automatically when you commit.

**HEAD:** A pointer to the commit you are currently "on." Usually `HEAD → branch → commit`. After a `git checkout <sha>` it points directly at a commit ("detached HEAD").

**Remote:** A named URL to another copy of the repository. `origin` is the conventional name for GitHub.

**Tracking branch:** A local reference to the last known state of a remote branch. Written as `origin/main`. Updated only by `git fetch`.

------

### Daily Workflow

```bash
# Start of day — bring your local repo up to date
git fetch origin          # download new commits from GitHub (does NOT touch your files)
git status                # see what's changed

# Work
# ... edit files ...

git diff                  # see unstaged changes (working tree vs. index)
git diff --staged         # see staged changes (index vs. last commit)

git add path/to/file      # stage a specific file
git add -p                # stage interactively — choose which hunks to include

git commit -m "message"   # commit everything staged

# Review before pushing
git log --oneline -10     # last 10 commits
git log --oneline main..HEAD  # commits on this branch not yet in main

git push origin feature/my-branch   # push to GitHub
```

------

### Branching

```bash
# Create and switch to a new branch from current HEAD
git checkout -b feature/tier4

# Create from a specific branch (recommended — always branch off main)
git checkout main
git checkout -b feature/tier4

# List all branches (local)
git branch

# List all branches (local + remote)
git branch -a

# Switch to an existing branch
git checkout main
# Modern equivalent (Git 2.23+):
git switch main

# Delete a local branch (safe — refuses if unmerged work)
git branch -d feature/old

# Delete a local branch forcibly (you lose unmerged commits)
git branch -D feature/old

# Delete a remote branch
git push origin --delete feature/old
```

**Branch naming conventions** (what professionals use):

```
feature/short-description     new functionality
fix/bug-description           bug fixes
review/phase-3-tier2          code review work
chore/update-gitignore        maintenance, no code change
docs/update-readme            documentation only
test/add-filter-tests         test-only changes
```

------

### Merging vs. Rebasing

These are the two ways to integrate changes from one branch into another. Understanding the difference is critical.

#### Merge

```bash
git checkout main
git merge feature/tier4
```

Creates a **merge commit** with two parents. The history shows exactly when branches diverged and rejoined. History is truthful but can be noisy with many small feature branches.

```
main:     A --- B --- C ------- M   ← merge commit
                       \       /
feature:                D --- E
```

**Use merge when:** the branch has meaningful parallel history (e.g., a long-lived feature or a PR you want to preserve as a unit of work).

#### Rebase

```bash
git checkout feature/tier4
git rebase main
```

Replays your commits *on top of* the current tip of `main`, as if you had branched off today. Creates **new commits** with new SHAs. The history is linear and clean.

```
Before:               After:
main:  A-B-C           main:  A-B-C
           \                       \
feature: D-E           feature:    D'-E'   (new commits, same changes)
```

**Use rebase when:** you want a clean, linear history; before opening a PR to make the diff easy to read; to integrate upstream changes into your local branch before pushing.

**Golden rule: never rebase commits that are already on a shared remote branch** (i.e., that others might have based work on). Rebase rewrites SHAs — anyone else with the old SHAs will have a divergent history.

------

### The PR (Pull Request) Lifecycle

```
1. Branch off main
   git checkout main && git pull origin main
   git checkout -b feature/my-feature

2. Develop + commit
   (edit, git add, git commit — repeat)

3. Push branch to GitHub
   git push -u origin feature/my-feature
   # -u sets the upstream tracking relationship (only needed first time)

4. Open PR on GitHub
   GitHub UI: "Compare & pull request"
   Or: gh pr create (if gh CLI is installed)

5. Review cycle
   More commits pushed to the same branch appear in the PR automatically

6. Merge (on GitHub)
   Three options:
   a) "Create a merge commit" — preserves all commits, adds merge commit
   b) "Squash and merge" — flattens all commits into one commit on main
   c) "Rebase and merge" — replays commits linearly onto main (no merge commit)

7. Clean up
   git checkout main
   git pull origin main            # bring local main up to date
   git branch -d feature/my-feature   # delete local branch (already merged)
```

------

### Staying In Sync With `main`

While you work on a feature branch, `main` advances. There are two clean ways to stay current:

**Option A — Merge main into your branch (safe, preserves history):**

```bash
git checkout feature/tier4
git fetch origin
git merge origin/main
```

**Option B — Rebase onto main (cleaner history, rewrites your commits):**

```bash
git checkout feature/tier4
git fetch origin
git rebase origin/main
# If conflicts:
#   edit file to resolve
#   git add file
#   git rebase --continue
# To abort and return to pre-rebase state:
#   git rebase --abort
```

For solo projects like this tracker, **rebase** keeps the log readable. For team projects, ask what convention the team uses.

------

### Undoing Things

This is the most important section — know your options before you need them.

| Situation                               | Command                                        | Effect                                                      |
| --------------------------------------- | ---------------------------------------------- | ----------------------------------------------------------- |
| Unstage a file (keep edits)             | `git restore --staged file`                    | Removes from index, keeps disk changes                      |
| Discard edits in working tree           | `git restore file`                             | **Destructive** — loses unsaved changes                     |
| Amend the last commit message           | `git commit --amend`                           | Rewrites the last commit — **do not use if already pushed** |
| Add forgotten file to last commit       | `git add file && git commit --amend --no-edit` | Same warning                                                |
| Undo last commit, keep changes staged   | `git reset --soft HEAD~1`                      | Moves branch back 1 commit, index unchanged                 |
| Undo last commit, keep changes unstaged | `git reset HEAD~1`                             | Moves branch back 1 commit, index reset                     |
| Undo last commit, discard all changes   | `git reset --hard HEAD~1`                      | **Destructive** — changes are gone                          |
| Undo a commit that's already pushed     | `git revert <sha>`                             | Creates a new "undo" commit — safe for shared history       |
| Recover a deleted commit                | `git reflog` then `git checkout <sha>`         | Git keeps orphaned commits for ~30 days                     |

**Rule of thumb:**

- `reset` rewrites history — only safe for commits that are **local only**
- `revert` creates a new commit that undoes a previous one — safe for **shared/pushed** commits
- `reflog` is your safety net — it tracks every `HEAD` movement, even after `reset --hard`

------

### Viewing History

```bash
git log                          # full log
git log --oneline                # one line per commit
git log --oneline --graph        # ASCII branch graph
git log --oneline main..HEAD     # commits on current branch not in main
git log --oneline -p             # commits with full diffs
git log --follow path/to/file    # history of a single file (through renames)

git show <sha>                   # show a specific commit's diff
git show HEAD                    # show the last commit
git diff main..feature/tier4     # diff between two branches
git diff HEAD~3                  # diff against 3 commits ago
```

------

### Stashing

Stash temporarily shelves uncommitted changes so you can switch context.

```bash
git stash                        # stash all tracked changes
git stash push -m "wip: tier4"   # stash with a name
git stash list                   # see all stashes
git stash pop                    # restore most recent stash and drop it
git stash apply stash@{1}        # restore a specific stash, keep it in list
git stash drop stash@{1}         # delete a specific stash
git stash branch feature/rescue  # create a branch from a stash
```

**When to use:** you're mid-edit and need to quickly switch to `main` to check something or fix a bug. Stash your work, do the fix, come back and `stash pop`.

------

### Commit Message Convention

Professional projects follow the **Conventional Commits** format (this project already uses it):

```
<type>(<optional scope>): <short summary>

<optional body>

<optional footer>
```

**Types:**

| Type       | When to use                                       |
| ---------- | ------------------------------------------------- |
| `feat`     | New user-facing feature                           |
| `fix`      | Bug fix                                           |
| `test`     | Adding or fixing tests only                       |
| `refactor` | Code restructure, no behavior change              |
| `chore`    | Tooling, config, dependencies, no code change     |
| `docs`     | Documentation only                                |
| `review`   | Code review findings applied (project convention) |
| `perf`     | Performance improvement                           |
| `ci`       | CI/CD changes                                     |

**Rules:**

- Summary line ≤ 72 characters
- Imperative mood: "add filter bar" not "added filter bar"
- Body explains *why*, not *what* (the diff shows what)
- Reference issues/PRs in footer: `Closes #12`

------

### `.gitignore` Rules

```gitignore
# Exact file
secrets.json

# All files with extension
*.log
*.pyc

# Directory (trailing slash)
.venv/
__pycache__/

# Directory anywhere in tree
**/__pycache__/

# Negate a rule (include despite earlier exclusion)
!important.log
```

**Critical:** `.gitignore` only ignores **untracked** files. If a file was already committed, adding it to `.gitignore` does nothing — you must `git rm --cached file` to untrack it (as in your current situation).

------

### Common Mistakes and Fixes

**Committed to `main` instead of a feature branch:**

```bash
git branch feature/oops          # create branch pointing at current commit
git reset --hard HEAD~1          # move main back one commit (only if not pushed)
git checkout feature/oops        # switch to the branch with your work
```

**Pushed a commit with a secret/credential:**

```bash
# 1. Remove the file and commit
git rm --cached secrets.json
git commit -m "remove secrets"
git push

# 2. Rotate the credential immediately (the secret is now in git history — assume it's compromised)
# 3. Optionally scrub history with git filter-repo (advanced, destructive — ask first)
```

**Merge conflict:**

```bash
git merge origin/main
# CONFLICT: both modified config.py
# Open config.py — look for <<<<<<< / ======= / >>>>>>> markers
# Edit to the correct final state, removing all markers
git add config.py
git merge --continue        # or: git commit
# To abort and go back:
git merge --abort
```

**Accidentally deleted a branch:**

```bash
git reflog                  # find the SHA of the last commit on that branch
git checkout -b recovered-branch <sha>
```

------

### This Project's Workflow (Applied)

```
main               ← production-ready; every commit is reviewed
  └─ feature/phase-3-tier4   ← active feature branch
```

**Your typical session:**

```bash
# Start
git checkout feature/phase-3-tier4
git fetch origin && git status      # check if anything new on remote

# Work → add → commit (repeat)
git add pages/1_Opportunities.py
git commit -m "feat(opportunities): T4-A row selection with session_state"

# End of session — push
git push origin feature/phase-3-tier4

# When the tier is done — open a PR on GitHub
# After PR is merged:
git checkout main
git pull origin main
git branch -d feature/phase-3-tier4
```

------

That covers ~95% of what you'll use day-to-day. The key things to internalize first: the three layers (working tree → index → commit), the difference between `reset` (rewrites history, local only) and `revert` (safe for shared history), and the merge-vs-rebase trade-off.