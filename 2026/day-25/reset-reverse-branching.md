# Day 25 – Git Reset vs Revert & Branching Strategies

**Date:** 06-Aug-2026

## Objective

Learn how to safely undo changes in Git using **git reset** and **git revert**, understand the differences between them, and explore common branching strategies used in software development.

---

# Task 1 – Git Reset (Hands-On)

## Commits Created

- Commit A
- Commit B
- Commit C

---

## 1. git reset --soft HEAD~1

### Command

```bash
git reset --soft HEAD~1
```

### Observation

- The latest commit was removed from Git history.
- All changes remained **staged**.
- No changes were lost.
- Ready to commit again without adding files.

---

## 2. git reset --mixed HEAD~1

### Command

```bash
git reset --mixed HEAD~1
```

or

```bash
git reset HEAD~1
```

### Observation

- The latest commit was removed.
- Changes remained in the working directory.
- Changes became **unstaged**.
- Files needed to be added again using `git add`.

---

## 3. git reset --hard HEAD~1

### Command

```bash
git reset --hard HEAD~1
```

### Observation

- The latest commit was removed.
- Changes in the working directory were deleted.
- Working directory became clean.
- This operation permanently removed uncommitted changes.

---

## Difference between --soft, --mixed and --hard

| Reset Type | Commit History | Staging Area | Working Directory |
|------------|----------------|--------------|-------------------|
| **--soft** | Moves HEAD back | Keeps staged | Keeps changes |
| **--mixed** | Moves HEAD back | Unstages changes | Keeps changes |
| **--hard** | Moves HEAD back | Removes staged changes | Deletes changes |

---

## Which reset is destructive?

**git reset --hard**

Reason:

It permanently deletes all uncommitted changes from the staging area and working directory.

---

## When should each be used?

### git reset --soft

- Fix previous commit message
- Combine commits
- Recommit without losing staged changes

### git reset --mixed

- Modify files before committing again
- Remove files from staging area

### git reset --hard

- Discard unwanted local work
- Restore repository to a previous clean state

---

## Should git reset be used after pushing commits?

Generally **No**.

Using git reset on pushed commits rewrites Git history and may cause conflicts for other developers working on the same branch.

---

# Task 2 – Git Revert (Hands-On)

## Commits Created

- Commit X
- Commit Y
- Commit Z

---

## Revert Commit Y

### Command

```bash
git revert <commit-id>
```

### Observation

- Git created a new commit.
- Commit Y remained in Git history.
- The changes introduced by Commit Y were undone.
- No commit history was removed.

---

## Is Commit Y still present?

**Yes**

Commit Y remains in Git history.

Git simply creates another commit that reverses its changes.

---

## Difference between git revert and git reset

### git reset

- Removes commits from history.
- Can rewrite Git history.
- Used mainly for local changes.

### git revert

- Creates a new commit to undo previous changes.
- Does not rewrite history.
- Safe for shared repositories.

---

## Why is git revert safer?

- Preserves Git history.
- Other developers do not lose commits.
- Avoids conflicts caused by history rewriting.
- Recommended for public/shared branches.

---

## When to use revert vs reset?

### Use git reset

- Before pushing commits.
- While working locally.
- To clean up commit history.

### Use git revert

- After commits have been pushed.
- In shared repositories.
- When safely undoing changes without affecting others.

---

# Task 3 – Reset vs Revert Comparison

| Feature | git reset | git revert |
|----------|-----------|------------|
| What it does | Moves HEAD to an earlier commit | Creates a new commit that undoes previous changes |
| Removes commit history | Yes | No |
| Safe for shared branches | No | Yes |
| Rewrites history | Yes | No |
| Creates new commit | No | Yes |
| Best used for | Local changes | Shared repositories |

---

# Task 4 – Branching Strategies

## 1. GitFlow

### How it works

GitFlow uses multiple long-lived branches.

- main
- develop
- feature
- release
- hotfix

### Flow

```
main
 │
develop
 ├── feature/login
 ├── feature/payment
 │
release
 │
main
 │
hotfix
```

### Used In

- Enterprise applications
- Large software teams
- Scheduled release cycles

### Pros

- Highly organized
- Stable releases
- Supports parallel development

### Cons

- Complex workflow
- Many branches to manage
- Slower delivery

---

## 2. GitHub Flow

### How it works

Developers create a feature branch from **main**, make changes, open a Pull Request, review code, and merge back into **main**.

### Flow

```
main
 │
feature branch
 │
Pull Request
 │
main
```

### Used In

- GitHub projects
- Web applications
- Continuous Deployment

### Pros

- Very simple
- Easy collaboration
- Fast development

### Cons

- Requires good testing
- Less suitable for scheduled releases

---

## 3. Trunk-Based Development

### How it works

Developers work on very short-lived branches or directly on the main branch and merge changes frequently.

### Flow

```
main
 │
small branch
 │
merge quickly
 │
main
```

### Used In

- Google
- Facebook
- Netflix
- Large CI/CD environments

### Pros

- Fast delivery
- Continuous Integration
- Small merge conflicts

### Cons

- Requires automated testing
- Requires disciplined development

---

# Answers

## Which strategy is best for a startup shipping fast?

**GitHub Flow** or **Trunk-Based Development**

Reason:

Simple workflow, faster releases, continuous deployment.

---

## Which strategy is best for a large team with scheduled releases?

**GitFlow**

Reason:

Supports multiple teams, release branches, and hotfixes.

---

## Which branching strategy does Kubernetes use?

Kubernetes primarily follows a **Trunk-Based Development** approach with short-lived feature branches and Pull Requests merged into the main development branch after code review and automated testing.

---

# Git Commands Learned (Days 22–25)

## Repository Setup

```bash
git init
git clone
git config --global user.name
git config --global user.email
```

---

## Basic Workflow

```bash
git status
git add
git commit
git log
git diff
```

---

## Branching

```bash
git branch
git checkout
git switch
git merge
git rebase
```

---

## Remote Repository

```bash
git remote add origin
git push
git pull
git fetch
git clone
```

---

## Temporary Work

```bash
git stash
git stash pop
git cherry-pick
```

---

## Undo Changes

```bash
git reset --soft
git reset --mixed
git reset --hard
git revert
git reflog
```

---

# Key Takeaways

- git reset rewrites history and is mainly used for local commits.
- git revert safely undoes changes by creating a new commit.
- git reflog helps recover commits even after a hard reset.
- GitHub Flow is ideal for fast-moving teams.
- GitFlow is suitable for large projects with planned releases.
- Trunk-Based Development supports rapid Continuous Integration and Continuous Deployment (CI/CD).
- Always be cautious while using `git reset --hard` because it permanently removes uncommitted changes.

---

# Conclusion

Today I learned how to safely undo mistakes in Git using **git reset** and **git revert**, understood the differences between them, explored recovery using **git reflog**, and studied the most popular branching strategies used by software engineering teams. These concepts are essential for effective version control and collaborative software development.
