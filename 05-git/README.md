# 🔀 Git — Complete In-Depth Guide (Beginner to Expert)

> **"Git is the most widely used version control system in the world. Mastering Git is essential for every developer — it's not optional."**

---

## 📑 Table of Contents

1. [Introduction & Overview](#1-introduction--overview)
2. [Installation & Configuration](#2-installation--configuration)
3. [Git Architecture & Internals](#3-git-architecture--internals)
4. [Basic Operations](#4-basic-operations)
5. [Branching & Merging](#5-branching--merging)
6. [Rebasing](#6-rebasing)
7. [Remote Repositories](#7-remote-repositories)
8. [Git Stash](#8-git-stash)
9. [Git Log & History](#9-git-log--history)
10. [Undoing Changes](#10-undoing-changes)
11. [Git Tags](#11-git-tags)
12. [Git Hooks](#12-git-hooks)
13. [Git Workflows](#13-git-workflows)
14. [Advanced Git Operations](#14-advanced-git-operations)
15. [Git with Spring Boot Projects](#15-git-with-spring-boot-projects)
16. [Best Practices](#16-best-practices)
17. [Common Mistakes & Pitfalls](#17-common-mistakes--pitfalls)
18. [Interview Questions & Answers (50+)](#18-interview-questions--answers-50)

---

## 1. Introduction & Overview

### What is Git?

Git is a **distributed version control system (DVCS)** created by Linus Torvalds in 2005 for Linux kernel development. It tracks changes in source code, enables collaboration among multiple developers, and maintains a complete history of every change.

### Why Git?

- **Distributed** — Every developer has a full copy of the repository
- **Fast** — Nearly all operations are local
- **Branching** — Lightweight branches make feature development easy
- **Data integrity** — SHA-1 hashing ensures data cannot be corrupted silently
- **Staging area** — Allows partial commits and precise change management
- **Open source** — Free, widely adopted, massive community

### Git vs Other VCS

| Feature | Git | SVN | Mercurial |
|---------|-----|-----|-----------|
| Type | Distributed | Centralized | Distributed |
| Speed | Very fast | Slow (network ops) | Fast |
| Branching | Lightweight | Heavy (copies) | Lightweight |
| Offline work | Full capability | Limited | Full capability |
| Learning curve | Steeper | Easier | Moderate |
| Market share | ~95% | ~3% | ~1% |

---

## 2. Installation & Configuration

### Installation

```bash
# Windows
winget install Git.Git
# Or download from https://git-scm.com/download/win

# macOS
brew install git
# Or: xcode-select --install

# Linux (Ubuntu/Debian)
sudo apt install git

# Linux (RHEL/CentOS)
sudo yum install git

# Verify
git --version
# git version 2.44.0
```

### Configuration

```bash
# User identity (REQUIRED — used in every commit)
git config --global user.name "Dilip"
git config --global user.email "dilip@example.com"

# Default branch name
git config --global init.defaultBranch main

# Default editor
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"           # Vim

# Line endings
git config --global core.autocrlf true   # Windows (converts LF→CRLF on checkout)
git config --global core.autocrlf input  # macOS/Linux (converts CRLF→LF on commit)

# Colorized output
git config --global color.ui auto

# Default merge strategy
git config --global pull.rebase true  # Rebase instead of merge on pull

# Aliases (shortcuts)
git config --global alias.st "status"
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.ci "commit"
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD"
git config --global alias.amend "commit --amend --no-edit"

# View all configuration
git config --list
git config --global --list

# Configuration levels:
# --system:  /etc/gitconfig           (all users)
# --global:  ~/.gitconfig             (current user)
# --local:   .git/config              (current repo, highest priority)
```

### SSH Key Setup (for GitHub/GitLab)

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "dilip@example.com"

# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Copy public key
cat ~/.ssh/id_ed25519.pub
# Paste in GitHub → Settings → SSH and GPG keys → New SSH key

# Test connection
ssh -T git@github.com
# Hi Dilip! You've successfully authenticated
```

---

## 3. Git Architecture & Internals

### The Three Trees

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Working     │     │  Staging     │     │  Repository  │
│  Directory   │────▶│  Area        │────▶│  (.git)      │
│  (Workspace) │ add │  (Index)     │ commit             │
└──────────────┘     └──────────────┘     └──────────────┘

git add:    Working Directory → Staging Area
git commit: Staging Area → Repository
git checkout/restore: Repository → Working Directory
```

### Git Object Model

```
Git stores FOUR types of objects (all SHA-1 hashed):

1. BLOB   — File content (raw data)
2. TREE   — Directory listing (pointers to blobs and other trees)
3. COMMIT — Snapshot (pointer to tree + metadata: author, message, parent)
4. TAG    — Named reference to a commit (annotated tags)

Commit → Tree → Blobs
  │        └──→ Trees (subdirectories)
  └──→ Parent Commit(s)
```

```bash
# Inspect git objects
git cat-file -t <sha1>     # Type of object
git cat-file -p <sha1>     # Content of object
git cat-file -s <sha1>     # Size of object

# Example:
git cat-file -p HEAD
# tree 4a3e5b...
# parent 8d2f1a...
# author Dilip <dilip@example.com> 1234567890 +0530
# committer Dilip <dilip@example.com> 1234567890 +0530
#
# Initial commit
```

### HEAD, Branches, and References

```
HEAD — Points to current branch (or commit in detached HEAD state)
Branch — A movable pointer to a commit
Tag — A fixed pointer to a commit

.git/
├── HEAD            → ref: refs/heads/main
├── refs/
│   ├── heads/      → Branch pointers
│   │   ├── main    → abc123 (commit SHA)
│   │   └── feature → def456
│   ├── tags/       → Tag pointers
│   │   └── v1.0    → 789abc
│   └── remotes/    → Remote tracking branches
│       └── origin/
│           ├── main → abc123
│           └── feature → def456
├── objects/        → All git objects (blobs, trees, commits)
├── config          → Repository configuration
├── index           → Staging area
└── hooks/          → Git hooks
```

---

## 4. Basic Operations

### Repository Initialization

```bash
# Create new repository
git init
git init my-project          # Create new directory with git

# Clone existing repository
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git          # SSH
git clone https://github.com/user/repo.git my-folder  # Custom directory
git clone --depth 1 https://github.com/user/repo.git   # Shallow clone
git clone --branch develop https://github.com/user/repo.git  # Specific branch
```

### File Status Lifecycle

```
Untracked → Staged → Committed → Modified → Staged → Committed
              ↑                      │
              └──────────────────────┘
```

### Daily Workflow Commands

```bash
# Check status
git status
git status -s               # Short format
# M  = Modified (staged)
# M  = Modified (not staged)
# MM = Modified, staged, then modified again
# A  = New file (staged)
# ?? = Untracked
# D  = Deleted

# Stage changes
git add file.txt             # Stage specific file
git add *.java               # Stage by pattern
git add src/                 # Stage entire directory
git add .                    # Stage everything (current dir + subdirs)
git add -A                   # Stage everything (entire working tree)
git add -p                   # Interactive staging (choose hunks)

# Unstage changes
git restore --staged file.txt    # Unstage (keep changes)
git reset HEAD file.txt          # Same (older syntax)

# Discard changes
git restore file.txt             # Discard working directory changes
git checkout -- file.txt         # Same (older syntax)

# Commit
git commit -m "Add user authentication"
git commit -am "Fix typo in README"  # Stage tracked files + commit
git commit --amend -m "New message"  # Modify last commit
git commit --amend --no-edit         # Add staged changes to last commit
git commit --allow-empty -m "Trigger CI"  # Empty commit

# View changes
git diff                     # Working directory vs staging area
git diff --staged            # Staging area vs last commit
git diff HEAD                # Working directory vs last commit
git diff branch1..branch2    # Between two branches
git diff --stat              # Summary of changes
git diff --name-only         # Only changed file names

# Remove files
git rm file.txt              # Remove from working dir AND stage removal
git rm --cached file.txt     # Remove from git tracking only (keep file)
git rm -r folder/            # Remove directory recursively

# Rename/Move
git mv old-name.txt new-name.txt

# Show file at specific commit
git show HEAD:src/main/java/App.java
git show abc1234:README.md
```

---

## 5. Branching & Merging

### Branch Operations

```bash
# List branches
git branch                   # Local branches
git branch -r                # Remote branches
git branch -a                # All branches (local + remote)
git branch -v                # Branches with last commit

# Create branch
git branch feature/login     # Create branch (don't switch)
git checkout -b feature/login  # Create and switch (old)
git switch -c feature/login    # Create and switch (new, preferred)

# Switch branch
git checkout main            # Old syntax
git switch main              # New syntax (preferred)

# Rename branch
git branch -m old-name new-name
git branch -m new-name       # Rename current branch

# Delete branch
git branch -d feature/login  # Delete (only if merged)
git branch -D feature/login  # Force delete (even if unmerged)

# Delete remote branch
git push origin --delete feature/login
git push origin :feature/login
```

### Merging

```bash
# Merge feature branch into main
git switch main
git merge feature/login

# Merge types:
# 1. Fast-forward merge (linear history)
#    main: A → B → C
#    feature:         D → E
#    After merge: A → B → C → D → E (main moves forward)

# 2. Three-way merge (creates merge commit)
#    main: A → B → C → F
#    feature:     D → E
#    After merge: A → B → C → F → G (merge commit)
#                      └─ D → E ─┘

# Force no fast-forward (always create merge commit)
git merge --no-ff feature/login

# Merge with commit message
git merge feature/login -m "Merge feature/login into main"

# Abort merge (during conflict)
git merge --abort

# Continue merge (after resolving conflicts)
git add resolved-file.txt
git merge --continue
# or
git commit
```

### Resolving Merge Conflicts

```bash
# When conflict occurs, Git marks the file:
<<<<<<< HEAD
Current branch content
=======
Incoming branch content
>>>>>>> feature/login

# Steps to resolve:
# 1. Open conflicted files
# 2. Edit to keep desired content (remove markers)
# 3. Stage resolved files: git add <file>
# 4. Complete merge: git commit

# Use merge tool
git mergetool                # Opens configured merge tool
git config --global merge.tool vscode
```

---

## 6. Rebasing

### What is Rebasing?

Rebasing moves your branch's commits to a new base commit, creating a **linear history**.

```bash
# Before rebase:
# main:    A → B → C
# feature:     D → E

# Rebase feature onto main:
git switch feature
git rebase main

# After rebase:
# main:    A → B → C
# feature:           D' → E'
# (D' and E' are NEW commits with different SHA-1 hashes)

# Then fast-forward merge:
git switch main
git merge feature
# Result: A → B → C → D' → E' (clean linear history)
```

### Interactive Rebase

```bash
# Rewrite last N commits
git rebase -i HEAD~3

# Opens editor with:
pick abc1234 First commit message
pick def5678 Second commit message
pick 789abcd Third commit message

# Commands:
# pick   = keep commit as-is
# reword = keep commit, change message
# edit   = keep commit, pause for amending
# squash = merge into previous commit (keep message)
# fixup  = merge into previous commit (discard message)
# drop   = remove commit entirely
# reorder = change commit order by reordering lines

# Example: Squash last 3 commits into one
pick abc1234 Add user model
squash def5678 Add user repository
squash 789abcd Add user service
# Result: One commit with combined messages
```

### Merge vs Rebase

| Feature | Merge | Rebase |
|---------|-------|--------|
| History | Preserves all commits + merge commit | Linear, clean history |
| Safety | Non-destructive | Rewrites history |
| Shared branches | ✅ Safe | ❌ Dangerous on shared branches |
| Conflict resolution | Once | Per commit |
| Use when | Merging feature → main | Updating feature branch with main |

**Golden Rule:** Never rebase commits that have been pushed and shared with others.

---

## 7. Remote Repositories

```bash
# List remotes
git remote                   # Names only
git remote -v                # Names + URLs

# Add remote
git remote add origin https://github.com/user/repo.git
git remote add upstream https://github.com/original/repo.git

# Change remote URL
git remote set-url origin git@github.com:user/repo.git

# Remove remote
git remote remove upstream

# Fetch (download changes, don't merge)
git fetch origin             # Fetch all branches from origin
git fetch --all              # Fetch from all remotes
git fetch --prune            # Remove deleted remote branches

# Pull (fetch + merge)
git pull                     # Pull current branch
git pull origin main         # Pull specific branch
git pull --rebase            # Pull with rebase instead of merge

# Push
git push                     # Push current branch
git push origin main         # Push specific branch
git push -u origin main      # Set upstream and push
git push --force             # Force push (DANGEROUS!)
git push --force-with-lease  # Safer force push (fails if remote changed)
git push --tags              # Push all tags
git push origin --delete feature/old  # Delete remote branch

# Track remote branch
git branch --set-upstream-to=origin/main main
git checkout --track origin/feature     # Create local tracking branch
```

---

## 8. Git Stash

```bash
# Stash current changes (save for later)
git stash                    # Stash tracked modified files
git stash -u                 # Include untracked files
git stash -a                 # Include ALL files (even ignored)
git stash save "WIP: login feature"  # With message

# List stashes
git stash list
# stash@{0}: On feature: WIP: login feature
# stash@{1}: On main: Quick fix

# Apply stash
git stash pop                # Apply latest + remove from stash
git stash apply              # Apply latest but keep in stash
git stash apply stash@{1}   # Apply specific stash
git stash pop stash@{2}     # Pop specific stash

# Show stash contents
git stash show               # Summary
git stash show -p            # Full diff

# Delete stash
git stash drop stash@{0}    # Drop specific stash
git stash clear              # Drop ALL stashes

# Create branch from stash
git stash branch new-branch stash@{0}
```

---

## 9. Git Log & History

```bash
# Basic log
git log                      # Full log
git log --oneline            # One line per commit
git log -n 10                # Last 10 commits
git log --graph              # ASCII graph
git log --all                # All branches
git log --decorate           # Show branch/tag names

# Pretty formats
git log --oneline --graph --all --decorate
git log --pretty=format:"%h %an %ar %s"
# %h = short hash, %an = author name, %ar = relative date, %s = subject

# Filter log
git log --author="Dilip"
git log --after="2024-01-01"
git log --before="2024-12-31"
git log --grep="fix"         # Search commit messages
git log -S "functionName"    # Search for code changes (pickaxe)
git log -- path/to/file      # Changes to specific file
git log --no-merges          # Exclude merge commits
git log main..feature        # Commits in feature but not in main

# Blame (who changed each line)
git blame file.txt
git blame -L 10,20 file.txt  # Lines 10-20 only
git blame --since=3.weeks    # Recent changes only

# Show specific commit
git show abc1234
git show HEAD
git show HEAD~3              # 3 commits ago
git show HEAD^               # Parent commit

# Shortlog (summary by author)
git shortlog -sn             # Sorted by commit count
```

---

## 10. Undoing Changes

```bash
# Undo working directory changes
git restore file.txt                  # Discard changes (Git 2.23+)
git checkout -- file.txt              # Same (older syntax)
git restore .                         # Discard all changes

# Undo staging
git restore --staged file.txt        # Unstage (keep changes)
git reset HEAD file.txt               # Same (older syntax)

# Undo last commit
git reset --soft HEAD~1               # Undo commit, keep changes staged
git reset --mixed HEAD~1              # Undo commit, keep changes unstaged (DEFAULT)
git reset --hard HEAD~1               # Undo commit, DISCARD all changes
# WARNING: --hard permanently deletes uncommitted changes!

# Revert a commit (creates a new commit that undoes changes)
git revert abc1234                    # Revert specific commit
git revert HEAD                       # Revert last commit
git revert HEAD~3..HEAD               # Revert last 3 commits
git revert -n abc1234                 # Revert without auto-commit

# Amend last commit
git commit --amend -m "New message"   # Change message
git add forgotten-file.txt
git commit --amend --no-edit          # Add file to last commit

# Recover deleted commits (within ~30 days)
git reflog                            # Show all HEAD movements
git reset --hard abc1234              # Restore to specific reflog entry
git cherry-pick abc1234               # Apply specific commit

# Clean untracked files
git clean -n                          # Dry run (show what would be deleted)
git clean -f                          # Delete untracked files
git clean -fd                         # Delete untracked files and directories
git clean -fX                         # Delete only ignored files
```

### Reset vs Revert vs Restore

| Command | Purpose | History | Safe for shared? |
|---------|---------|---------|-----------------|
| `restore` | Discard working directory changes | No change | N/A |
| `reset --soft` | Undo commit, keep staged | Rewrites | ❌ No |
| `reset --mixed` | Undo commit, keep unstaged | Rewrites | ❌ No |
| `reset --hard` | Undo commit, discard changes | Rewrites | ❌ No |
| `revert` | Create new "undo" commit | Preserves | ✅ Yes |

---

## 11. Git Tags

```bash
# Lightweight tag (just a pointer)
git tag v1.0.0

# Annotated tag (full object with message, author, date)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag a specific commit
git tag -a v1.0.0 abc1234 -m "Release 1.0.0"

# List tags
git tag
git tag -l "v1.*"            # Filter

# Show tag details
git show v1.0.0

# Push tags
git push origin v1.0.0       # Push specific tag
git push origin --tags        # Push all tags

# Delete tags
git tag -d v1.0.0             # Delete local
git push origin --delete v1.0.0  # Delete remote

# Checkout a tag (detached HEAD)
git checkout v1.0.0
git switch --detach v1.0.0
```

### Semantic Versioning (SemVer)

```
MAJOR.MINOR.PATCH
  │     │     │
  │     │     └── Bug fixes (backward compatible)
  │     └──────── New features (backward compatible)
  └────────────── Breaking changes

Examples:
v1.0.0 → v1.0.1 (bug fix)
v1.0.0 → v1.1.0 (new feature)
v1.0.0 → v2.0.0 (breaking change)
v1.0.0-SNAPSHOT   (development)
v1.0.0-RC1        (release candidate)
```

---

## 12. Git Hooks

Git hooks are scripts that run automatically at certain points in the Git workflow.

```bash
# Location: .git/hooks/ (local, not shared)
# To share hooks: Use a hooks directory + git config core.hooksPath

# Client-side hooks:
pre-commit        # Before commit — run linters, formatters
prepare-commit-msg # Before commit message editor opens
commit-msg        # After commit message — validate format
post-commit       # After commit — notifications
pre-push          # Before push — run tests
pre-rebase        # Before rebase

# Server-side hooks:
pre-receive       # Before accepting push
update            # Like pre-receive but per branch
post-receive      # After accepting push — deploy, notify
```

### Example: Pre-commit Hook

```bash
#!/bin/sh
# .git/hooks/pre-commit

echo "Running pre-commit checks..."

# Run tests
./mvnw test -q
if [ $? -ne 0 ]; then
    echo "Tests failed! Commit aborted."
    exit 1
fi

# Check for TODOs
if git diff --cached | grep -i "TODO"; then
    echo "WARNING: Commit contains TODO comments"
    exit 1
fi

echo "Pre-commit checks passed!"
exit 0
```

### Sharing Hooks with the Team

```bash
# Create a hooks directory in the project
mkdir .githooks

# Set Git to use it
git config core.hooksPath .githooks

# Or add to .gitconfig (project level)
echo "[core]\n\thooksPath = .githooks" >> .git/config
```

---

## 13. Git Workflows

### 1. Feature Branch Workflow

```
main:     A → B → C → D → (merge) → H → (merge) → K
                    ↑                   ↑
feature1:       E → F → G           
feature2:                    I → J
```

```bash
# 1. Create feature branch
git switch -c feature/user-auth

# 2. Make changes and commit
git add .
git commit -m "Add login endpoint"

# 3. Push feature branch
git push -u origin feature/user-auth

# 4. Create Pull Request on GitHub

# 5. After review, merge into main
git switch main
git pull
git merge feature/user-auth
git push

# 6. Delete feature branch
git branch -d feature/user-auth
git push origin --delete feature/user-auth
```

### 2. Gitflow Workflow

```
main:       ─────────────── v1.0 ──────────── v1.1 ──
develop:    ── A → B → C ───────── D → E → F ───────
                     ↑                   ↑
feature:        x → y → z          p → q
hotfix:                      ── h ──
```

Branches:
- `main` — Production-ready code (tagged releases)
- `develop` — Integration branch for features
- `feature/*` — New features (from develop)
- `release/*` — Release preparation (from develop)
- `hotfix/*` — Urgent fixes (from main)

### 3. Trunk-Based Development

```
main:  A → B → C → D → E → F → G → H
       ↑       ↑       ↑       ↑
       short-lived feature branches (< 1 day)
```

- Everyone commits to main (or short-lived branches)
- Feature flags instead of long-lived branches
- Continuous Integration and Deployment
- Preferred by teams doing CI/CD

---

## 14. Advanced Git Operations

### Cherry-Pick

```bash
# Apply a specific commit from another branch
git cherry-pick abc1234
git cherry-pick abc1234 def5678  # Multiple commits
git cherry-pick abc1234..def5678  # Range of commits
git cherry-pick --no-commit abc1234  # Apply changes but don't commit
```

### Bisect (Find Bug-Introducing Commit)

```bash
# Binary search through commits to find the one that introduced a bug
git bisect start
git bisect bad                  # Current commit is bad
git bisect good v1.0.0          # v1.0.0 was good

# Git checks out a commit in the middle
# Test it, then mark as good or bad
git bisect good                 # or: git bisect bad

# Repeat until Git finds the exact commit
# Git reports: "abc1234 is the first bad commit"

git bisect reset                # Return to original state

# Automate with a test script
git bisect run ./test-script.sh
```

### Submodules

```bash
# Add a submodule
git submodule add https://github.com/user/lib.git libs/external-lib

# Clone with submodules
git clone --recursive https://github.com/user/repo.git

# Initialize submodules after clone
git submodule init
git submodule update

# Update submodules to latest
git submodule update --remote

# Remove submodule
git submodule deinit libs/external-lib
git rm libs/external-lib
rm -rf .git/modules/libs/external-lib
```

### Worktree (Multiple Working Directories)

```bash
# Create a new working tree for a branch
git worktree add ../my-project-hotfix hotfix/critical-bug

# List worktrees
git worktree list

# Remove worktree
git worktree remove ../my-project-hotfix
```

---

## 15. Git with Spring Boot Projects

### .gitignore for Spring Boot

```gitignore
# Compiled class files
*.class

# Package files
*.jar
*.war
*.ear

# Build directories
target/
build/
out/
bin/

# IDE files
.idea/
*.iml
.vscode/
.settings/
.project
.classpath
*.swp

# OS files
.DS_Store
Thumbs.db

# Environment files
.env
.env.local
*.env

# Application configuration with secrets
application-local.properties
application-local.yml

# Logs
*.log
logs/

# Maven
.mvn/timing.properties
.mvn/wrapper/maven-wrapper.jar

# Gradle
.gradle/
gradle-app.setting
!gradle-wrapper.jar
!gradle-wrapper.properties

# Spring Boot
spring-shell.log

# Lombok
lombok.config

# Coverage reports
coverage/
*.exec
```

### Commit Message Convention

```
type(scope): subject

body (optional)

footer (optional)

Types:
- feat:     New feature
- fix:      Bug fix
- docs:     Documentation
- style:    Formatting (no code change)
- refactor: Code restructuring
- test:     Adding tests
- chore:    Build, CI, dependencies
- perf:     Performance improvement

Examples:
feat(auth): add JWT token refresh endpoint
fix(user): resolve null pointer in getUserById
docs(readme): update API documentation
refactor(order): extract payment processing to service
test(user): add unit tests for UserService
chore(deps): upgrade Spring Boot to 3.3.0
```

---

## 16. Best Practices

1. **Commit often, push frequently** — Small, focused commits
2. **Write meaningful commit messages** — Use conventional commits
3. **Use feature branches** — Never commit directly to main
4. **Pull before push** — Avoid unnecessary merge conflicts
5. **Use `.gitignore`** — Never commit build artifacts, secrets, IDE files
6. **Don't commit secrets** — Use environment variables or secret managers
7. **Use `--force-with-lease` instead of `--force`** — Safer force push
8. **Squash WIP commits before merging** — Clean history
9. **Use Git hooks** — Automate quality checks
10. **Tag releases** — Use semantic versioning (v1.2.3)
11. **Keep branches short-lived** — Merge within 1-3 days
12. **Review code via Pull Requests** — Quality gate
13. **Use `git stash` wisely** — Don't hoard stashes
14. **Learn `git reflog`** — Your safety net for undoing mistakes
15. **Sign commits** — `git commit -S` for verified commits

---

## 17. Common Mistakes & Pitfalls

```bash
# ❌ MISTAKE 1: Committing to wrong branch
# ✅ FIX: 
git stash
git switch correct-branch
git stash pop
# Or: git cherry-pick if already committed

# ❌ MISTAKE 2: Force pushing to shared branches
git push --force  # DESTROYS others' work!
# ✅ FIX: Use git push --force-with-lease (fails if remote changed)

# ❌ MISTAKE 3: Committing secrets/passwords
# ✅ FIX: Use .gitignore, pre-commit hooks, git-secrets tool
# If already committed: BFG Repo Cleaner or git filter-branch

# ❌ MISTAKE 4: Giant commits
# ✅ FIX: Commit often, use git add -p for partial staging

# ❌ MISTAKE 5: Rebasing shared/public branches
# ✅ FIX: Only rebase LOCAL branches. Use merge for shared branches.

# ❌ MISTAKE 6: Not pulling before pushing
git push  # REJECTED! Remote has new commits
# ✅ FIX: git pull --rebase then git push

# ❌ MISTAKE 7: Losing work with git reset --hard
# ✅ FIX: git reflog → find the commit → git reset --hard <sha>
# Reflog keeps references for ~30 days

# ❌ MISTAKE 8: Ignoring merge conflicts
# ✅ FIX: Always resolve conflicts carefully, test after resolving

# ❌ MISTAKE 9: Not using .gitignore from the start
# ✅ FIX: Add .gitignore in the very first commit

# ❌ MISTAKE 10: Committing large binary files
# ✅ FIX: Use Git LFS for large files, or don't commit them
```

---

## 18. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is Git?**
**A:** Git is a distributed version control system that tracks changes in source code. Every developer has a full copy of the repository, enabling offline work, fast operations, and robust branching.

**Q2: What is the difference between Git and GitHub?**
**A:** Git is the version control TOOL (runs locally). GitHub is a PLATFORM/service that hosts Git repositories online and adds collaboration features (Pull Requests, Issues, Actions, etc.). Alternatives: GitLab, Bitbucket.

**Q3: What is a Git repository?**
**A:** A directory containing your project files plus a hidden `.git` folder that stores the complete history, branches, tags, and configuration. It's the entire database of your project's history.

**Q4: What is the difference between `git pull` and `git fetch`?**
**A:** `git fetch` downloads new data from remote without modifying your working directory. `git pull` = `git fetch` + `git merge` (or rebase). Fetch is safer because you can review changes before integrating.

**Q5: What is a branch in Git?**
**A:** A branch is a lightweight movable pointer to a commit. It allows parallel development. Creating a branch is instant (just creates a pointer). The default branch is typically `main` or `master`.

**Q6: What is `git clone`?**
**A:** Creates a complete copy of a remote repository including all history, branches, and tags. It automatically sets up `origin` as the remote.

**Q7: What is the staging area (index)?**
**A:** An intermediate area between the working directory and the repository. `git add` moves changes to staging. `git commit` saves staged changes to the repository. It allows you to create precise, selective commits.

**Q8: What is `HEAD` in Git?**
**A:** HEAD is a pointer to the current branch's latest commit. It represents "where you are now." When you commit, HEAD moves forward. Detached HEAD means HEAD points to a commit, not a branch.

---

### Intermediate Level

**Q9: What is the difference between merge and rebase?**
**A:** Merge creates a new merge commit combining two branches (preserves history). Rebase replays your commits on top of another branch (creates linear history but rewrites commits). Use merge for shared branches, rebase for local feature branches.

**Q10: How do you resolve a merge conflict?**
**A:** 1) Open conflicted files (marked with `<<<<<<<`, `=======`, `>>>>>>>`). 2) Edit to keep desired code. 3) Remove conflict markers. 4) `git add` resolved files. 5) `git commit` to complete the merge.

**Q11: What is `git stash`?**
**A:** Temporarily saves uncommitted changes so you can switch branches or work on something else. `git stash` saves, `git stash pop` restores. Changes are stored in a stack.

**Q12: What is the difference between `git reset` and `git revert`?**
**A:** `reset` moves HEAD backward, rewriting history (dangerous for shared branches). `revert` creates a NEW commit that undoes a previous commit (safe for shared branches).

**Q13: What is a Pull Request (PR)?**
**A:** A request to merge your branch into another branch, typically via GitHub/GitLab. It enables code review, discussion, and CI checks before merging. It's a collaboration and quality control mechanism.

**Q14: What is `git cherry-pick`?**
**A:** Applies a specific commit from one branch to another without merging the entire branch. Useful for applying bug fixes from one branch to another.

**Q15: What is `.gitignore`?**
**A:** A file that specifies patterns of files/directories Git should ignore (not track). Common entries: build output, IDE files, environment files, logs. Patterns support wildcards and negation.

---

### Advanced Level

**Q16: What is `git reflog`?**
**A:** A log of all HEAD movements (commits, resets, checkouts, merges). It's your safety net — you can recover "lost" commits within ~30 days. `git reflog` → find commit → `git reset --hard <sha>`.

**Q17: What is `git bisect`?**
**A:** A binary search tool to find the commit that introduced a bug. You mark one commit as good and one as bad, and Git checks out commits in between for you to test. Can be automated with `git bisect run <script>`.

**Q18: Explain the three types of `git reset`.**
**A:** `--soft`: Moves HEAD, keeps staging and working dir. `--mixed` (default): Moves HEAD, resets staging, keeps working dir. `--hard`: Moves HEAD, resets staging AND working dir (DESTROYS changes).

**Q19: What is `git rebase -i` (interactive rebase)?**
**A:** Allows rewriting history interactively: squash commits, reorder, edit messages, drop commits. Useful for cleaning up before merging a feature branch. Commands: pick, squash, fixup, reword, edit, drop.

**Q20: What are Git submodules?**
**A:** A way to include one Git repository inside another. The parent tracks a specific commit of the submodule. Useful for shared libraries. Complex to manage — alternatives: Git subtree, dependency managers.

---

### Rapid-Fire (Q21–Q50)

**Q21: What is `git tag`?** A named reference to a specific commit. Used for marking releases (v1.0.0). Annotated tags store metadata; lightweight tags are just pointers.

**Q22: Difference between `git add .` and `git add -A`?** `.` adds from current directory down. `-A` adds from entire working tree.

**Q23: What is a detached HEAD?** HEAD points to a commit instead of a branch. Commits made in this state can be lost unless you create a branch.

**Q24: What is `git clean`?** Removes untracked files. `-n` for dry run, `-f` to force, `-d` for directories, `-x` for ignored files too.

**Q25: What is `--force-with-lease`?** A safer force push that fails if the remote branch has been updated by someone else since your last fetch.

**Q26: What is `git blame`?** Shows who last modified each line of a file and when. Useful for understanding code history.

**Q27: What is `git worktree`?** Creates additional working directories for the same repo. Work on multiple branches simultaneously without stashing.

**Q28: What is a shallow clone?** `git clone --depth 1` — only downloads the latest commit, not full history. Faster for CI/CD.

**Q29: What is the difference between `origin` and `upstream`?** `origin` is your fork. `upstream` is the original repository you forked from.

**Q30: What is `git diff --staged`?** Shows changes that are staged (in the index) compared to the last commit.

**Q31: What is `git log --oneline --graph`?** Compact log with ASCII art showing the branch/merge structure.

**Q32: What is a fast-forward merge?** When the branch being merged has no diverging commits. HEAD simply moves forward. No merge commit is created.

**Q33: What is `git commit --amend`?** Modifies the last commit (message or content). Rewrites the commit SHA.

**Q34: What is `git remote -v`?** Shows remote repository names and their URLs (fetch and push).

**Q35: What is `git config`?** Manages Git configuration at system, global, or local level. Stores user info, aliases, preferences.

**Q36: What is `.git` directory?** Hidden directory containing the entire Git database: objects, refs, config, hooks, HEAD, index.

**Q37: What is `git rm --cached`?** Removes a file from Git tracking but keeps it in the working directory.

**Q38: What is `git push -u`?** Sets the upstream tracking branch. After this, `git push` and `git pull` work without specifying the remote/branch.

**Q39: What is `git log -S "text"`?** Pickaxe search — finds commits where the specified string was added or removed.

**Q40: What is `git shortlog`?** Summarizes git log output by author. `-s` for count only, `-n` for sorted.

**Q41: What is a merge commit?** A commit with two or more parents, created when merging branches that have diverged.

**Q42: What is the three-way merge?** Compares two branches against their common ancestor to determine changes. Creates a merge commit.

**Q43: What is `git checkout -b`?** Creates a new branch and switches to it. Modern equivalent: `git switch -c`.

**Q44: How do you undo `git add`?** `git restore --staged <file>` or `git reset HEAD <file>`.

**Q45: What is `git diff branch1..branch2`?** Shows differences between two branches.

**Q46: What is `git stash pop` vs `git stash apply`?** `pop` applies and removes from stash stack. `apply` applies but keeps in stash.

**Q47: What is `git rebase --onto`?** Rebases a branch onto a specific commit: `git rebase --onto newBase oldBase branch`.

**Q48: What is Conventional Commits?** A commit message convention: `type(scope): subject`. Types: feat, fix, docs, refactor, test, chore.

**Q49: What is `pre-commit` hook?** A script that runs before every commit. Used for linting, formatting, running tests. Exit code 1 aborts the commit.

**Q50: What is `git filter-branch`?** Rewrites history by running a filter on each commit. Used for removing sensitive data. Modern alternative: `git filter-repo`.

---

## 📚 References & Further Reading

- [Git Official Documentation](https://git-scm.com/doc)
- [Pro Git Book (Free)](https://git-scm.com/book/en/v2)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Learn Git Branching (Interactive)](https://learngitbranching.js.org/)

---

> **Previous Topic:** [← 04 - JUnit](../04-junit/README.md)  
> **Next Topic:** [06 - DSA →](../06-dsa/README.md)
