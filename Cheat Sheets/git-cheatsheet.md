# Git Cheatsheet

Quick reference for essential and intermediate Git commands, with data engineering examples.

---

## Common Workflows

### Feature Branch Workflow
```bash
# Start new feature from main
git switch main
git pull origin main
git switch -c feature/new-loans-model

# Work, stage, commit
git add models/loans.sql
git commit -m "Add loans model with balance calculations"

# Keep up to date with main
git fetch origin
git rebase origin/main

# Push and open PR
git push -u origin feature/new-loans-model
```

### Fix a Mistake Before Pushing
```bash
# Forgot to add a file to last commit
git add forgotten_file.sql
git commit --amend --no-edit          # Add to last commit, keep message

# Wrong commit message
git commit --amend -m "Better message"

# Undo last commit but keep changes
git reset HEAD~1
```

### Squash Commits Before Merging
```bash
# Interactive rebase to squash last 3 commits into one
git rebase -i HEAD~3
# In editor: change "pick" to "squash" (or "s") on commits 2 and 3
# Edit the combined commit message, save
git push --force-with-lease
```

### Check What You're About to Push
```bash
git log origin/main..HEAD            # Commits not yet on remote
git diff origin/main..HEAD           # All changes not yet on remote
git diff origin/main..HEAD --stat    # Summary of files changed
```

### Find a Lost Commit
```bash
git reflog                            # Full history of HEAD movements
git checkout <commit-hash>            # Inspect the lost commit
git branch recovered-work <hash>     # Restore it as a branch
```

### Untrack a File Already Committed
```bash
# 1. Add the file to .gitignore (if not already there)
echo "secrets.env" >> .gitignore

# 2. Remove it from Git's index (stops tracking it, keeps local file)
git rm --cached secrets.env

# For a whole directory:
git rm --cached -r target/

# 3. Commit the change
git commit -m "Stop tracking secrets.env"

# Git will no longer track changes to it, even though it still exists locally
```

---

## Setup & Configuration

| Command | Description | Example | Use Case |
|---------|-------------|---------|----------|
| `git config --global user.name` | Set global username for all repos | `git config --global user.name "Your Name"` | First-time setup |
| `git config --global user.email` | Set global email for all repos | `git config --global user.email "you@example.com"` | First-time setup |
| `git config --global core.editor` | Set default text editor | `git config --global core.editor "vim"` | Set commit editor |
| `git config --global init.defaultBranch` | Set default branch name | `git config --global init.defaultBranch main` | Use `main` instead of `master` |
| `git config --list` | Show all current config settings | `git config --list` | Verify config |
| `git config --global alias.<name>` | Create a shortcut alias | `git config --global alias.st status` | `git st` runs `git status` |

---

## Repository Basics

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `git init` | Create a new empty repository | N/A | `git init` | Start tracking a project |
| `git clone` | Download a repository from remote | `--depth` (shallow), `--branch` (specific branch) | `git clone --depth 1 repo-url` | Download project (latest only) |
| `git status` | Show changed, staged, and untracked files | `-s` (short format), `-b` (branch info) | `git status -s` | Quick status check |
| `git add` | Stage file changes for next commit | `-A` (all), `-p` (patch/interactive), `.` (current dir) | `git add -A` | Stage all changes |
| `git add -p` | Interactively choose hunks to stage | N/A | `git add -p model.sql` | Stage only some changes in a file |
| `git commit` | Save staged changes with a message | `-m` (message), `-a` (all tracked), `--amend` (modify last) | `git commit -m "Add loans model"` | Commit staged changes |
| `git commit -am` | Stage all tracked files and commit in one step | N/A | `git commit -am "Fix null handling"` | Shortcut: add & commit |
| `git log` | Show commit history | `--oneline`, `--graph`, `--all`, `-n` (limit) | `git log --oneline --graph` | View branching history |
| `git show` | Show details of a specific commit | N/A | `git show abc1234` | Inspect a commit |

---

## Branching & Merging

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `git branch` | List all local branches | `-a` (all incl. remote), `-v` (verbose), `-d` (delete) | `git branch -a` | See all branches |
| `git branch <name>` | Create a new branch | N/A | `git branch feature/add-loans-model` | Create without switching |
| `git checkout -b` | Create and switch to new branch | N/A | `git checkout -b feature/add-loans-model` | Create & switch (classic) |
| `git switch -c` | Create and switch to new branch (modern) | N/A | `git switch -c feature/add-loans-model` | Create & switch (preferred) |
| `git switch` | Switch to an existing branch | N/A | `git switch main` | Switch branches |
| `git merge` | Merge another branch into current branch | `--no-ff` (always merge commit), `--squash` | `git merge feature/loans` | Merge feature into main |
| `git rebase` | Reapply commits on top of another branch | `-i` (interactive), `--onto` | `git rebase main` | Keep linear history |
| `git rebase -i` | Interactive rebase — squash, reorder, edit commits | `HEAD~n` (last n commits) | `git rebase -i HEAD~3` | Clean up last 3 commits |
| `git cherry-pick` | Apply a specific commit to current branch | `-n` (no commit), `-x` (note source) | `git cherry-pick abc1234` | Port a single fix |
| `git branch -d` | Delete a merged branch | `-D` (force delete unmerged) | `git branch -d old-feature` | Clean up after merge |

---

## Remote Repositories

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `git remote -v` | Show all remotes and URLs | N/A | `git remote -v` | Verify remote config |
| `git remote add` | Add a new remote connection | N/A | `git remote add origin git@github.com:user/repo.git` | Connect to remote |
| `git remote rename` | Rename a remote | N/A | `git remote rename origin upstream` | Rename remote |
| `git remote remove` | Remove a remote | N/A | `git remote remove old-remote` | Remove connection |
| `git fetch` | Download remote changes without merging | `--all` (all remotes), `--prune` (remove stale) | `git fetch --all --prune` | Sync remote tracking branches |
| `git pull` | Fetch and merge remote changes | `--rebase` (rebase instead), `--ff-only` | `git pull --rebase origin main` | Update with rebase |
| `git push` | Upload local commits to remote | `-u` (set upstream), `--force-with-lease`, `--tags` | `git push -u origin feature-branch` | Push and set upstream |
| `git push --force-with-lease` | Force push (safe — fails if remote has new commits) | N/A | `git push --force-with-lease` | Safe force push after rebase |

---

## Comparing Changes

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `git diff` | Show unstaged changes | `--stat` (summary), `--name-only`, `--color` | `git diff` | What changed but not staged |
| `git diff --staged` | Show staged changes vs last commit | N/A | `git diff --staged` | Review before committing |
| `git diff <branch1> <branch2>` | Compare two branches | `--name-status`, `--stat` | `git diff main feature` | What differs between branches |
| `git diff <commit1> <commit2>` | Compare two commits | N/A | `git diff HEAD~1 HEAD` | Changes in last commit |
| `git diff <file>` | Show changes for a specific file | N/A | `git diff models/loans.sql` | File-specific diff |

---

## Undo & Reset

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `git restore <file>` | Discard unstaged changes in a file | `--staged` (unstage), `--source` (from commit) | `git restore models/loans.sql` | Undo file changes |
| `git restore --staged <file>` | Unstage a file (keep changes in working dir) | N/A | `git restore --staged models/loans.sql` | Unstage without losing work |
| `git revert <commit>` | Create a new commit that undoes a commit | `--no-commit` (stage only) | `git revert abc1234` | Safe undo — preserves history |
| `git reset HEAD~1` | Undo last commit — keep changes staged | N/A | `git reset HEAD~1` | Soft undo last commit |
| `git reset --soft HEAD~1` | Undo last commit — keep changes staged | N/A | `git reset --soft HEAD~1` | Recommit with edits |
| `git reset --mixed HEAD~1` | Undo last commit — keep changes unstaged | N/A | `git reset --mixed HEAD~1` | Default reset |
| `git reset --hard HEAD~1` | Undo last commit — discard all changes | N/A | `git reset --hard HEAD~1` | Completely remove last commit |
| `git clean -fd` | Remove all untracked files and directories | `-n` (dry run first) | `git clean -nfd` then `git clean -fd` | Clean untracked files |

---

## Stashing

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `git stash` | Save uncommitted changes and revert working dir | `push -m` (with message), `-u` (include untracked) | `git stash push -m "WIP: loans refactor"` | Switch branches without committing |
| `git stash list` | Show all saved stashes | N/A | `git stash list` | View all stashes |
| `git stash pop` | Reapply most recent stash and remove it | N/A | `git stash pop` | Restore last stash |
| `git stash apply` | Reapply a stash without removing it | `stash@{n}` (specific stash) | `git stash apply stash@{2}` | Apply specific stash |
| `git stash drop` | Delete a specific stash | `stash@{n}` (specific stash) | `git stash drop stash@{0}` | Remove a stash |
| `git stash clear` | Delete all stashes | N/A | `git stash clear` | Clean out all stashes |

---

## Tags

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `git tag` | List all tags | `-l` (list with pattern) | `git tag -l "v1.*"` | View all version tags |
| `git tag <name>` | Create a lightweight tag | N/A | `git tag v1.0` | Mark a release |
| `git tag -a <name>` | Create an annotated tag with message | `-m` (message) | `git tag -a v1.0 -m "First release"` | Proper release tag |
| `git push --tags` | Push all tags to remote | N/A | `git push --tags` | Publish release tags |
| `git tag -d <name>` | Delete a local tag | N/A | `git tag -d v1.0` | Remove local tag |

---

## History & Search

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `git log --oneline` | Condensed one-line history | `--graph`, `--all`, `-n` (limit) | `git log --oneline --graph --all` | Visual branch history |
| `git log --author` | Filter commits by author | N/A | `git log --author="Alice"` | See someone's commits |
| `git log --since / --until` | Filter commits by date | N/A | `git log --since="2024-01-01"` | Commits in a date range |
| `git log -S` | Find commits that added/removed a string | N/A | `git log -S "balance_usd"` | Find when code was introduced |
| `git log -- <file>` | Show history for a specific file | N/A | `git log -- models/loans.sql` | File change history |
| `git blame <file>` | Show who last changed each line | `-L` (line range) | `git blame models/loans.sql` | Find who wrote a line |
| `git grep` | Search for a pattern across tracked files | `-n` (line numbers), `-i` (ignore case) | `git grep "balance_usd"` | Search codebase |
| `git bisect` | Binary search to find commit that introduced a bug | `start`, `good`, `bad` | `git bisect start; git bisect bad; git bisect good v1.0` | Debug regression |

---

## .gitignore

> To stop tracking a file already committed: `git rm --cached <file>` then commit. See **Untrack a File Already Committed** workflow below.

```bash
# Ignore a specific file
secrets.env

# Ignore all .env files
*.env

# Ignore a directory
target/
__pycache__/
.venv/

# Ignore all .log files in any subdirectory
**/*.log

# Exception: track this specific file even though *.log is ignored
!important.log

# Ignore OS files
.DS_Store
Thumbs.db

# Common dbt ignores
target/
dbt_packages/
logs/
profiles.yml
```

