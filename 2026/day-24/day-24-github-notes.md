# Day 24 – Advanced Git: Merge, Rebase, Stash & Cherry Pick

## Objective
Learn advanced Git operations including Merge, Rebase, Squash Merge, Stash, and Cherry Pick by performing hands-on exercises.

---

# Task 1 – Git Merge

## Fast-Forward Merge
A fast-forward merge happens when the target branch has no new commits after the feature branch was created. Git simply moves the branch pointer forward without creating a merge commit.

### Example
- Created `feature-login` branch.
- Added two commits.
- Merged into `main`.
- Result: **Fast-forward merge**.

## Merge Commit
A merge commit is created when both the feature branch and the target branch contain new commits. Git combines both histories into a new merge commit.

## Merge Conflict
A merge conflict occurs when Git cannot automatically merge changes because the same line of the same file was modified in different branches.

### Conflict Resolution Steps
1. Open the conflicted file.
2. Remove conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).
3. Keep the required content.
4. Save the file.
5. Run:
   ```bash
   git add <file>
   git commit
   ```

---

# Task 2 – Git Rebase

## What is Rebase?

Git Rebase moves the commits of a feature branch on top of the latest commit of another branch, creating a clean and linear history.

### Rebase Workflow

```bash
git checkout feature-dashboard
git rebase main
```

## Merge vs Rebase

| Merge | Rebase |
|--------|---------|
| Preserves complete history | Creates linear history |
| Creates merge commits | Rewrites commit history |
| Good for team collaboration | Good for cleaning feature branches |

## Why should you not rebase shared commits?

Rebasing changes commit history. If commits have already been pushed and shared with others, rebasing can create conflicts and confusion for collaborators.

## When to use Rebase?

- Before merging a feature branch.
- To keep Git history clean.
- For local feature branches.

---

# Task 3 – Squash Merge

## What is Squash Merge?

Squash merge combines multiple commits into a single commit before merging into the target branch.

### Command

```bash
git merge --squash feature-profile
```

### Advantages

- Clean Git history.
- One commit per feature.
- Easy to review.

### Trade-Off

- Individual commit history is lost.

## Squash Merge vs Regular Merge

| Squash Merge | Regular Merge |
|---------------|---------------|
| One commit | All commits preserved |
| Cleaner history | Complete history available |
| Easier review | Better for tracking development |

---

# Task 4 – Git Stash

## What is Git Stash?

Git Stash temporarily saves uncommitted changes without creating a commit.

### Common Commands

```bash
git stash
git stash list
git stash pop
git stash apply
git stash push -m "message"
```

## Difference Between Pop and Apply

| git stash pop | git stash apply |
|----------------|-----------------|
| Restores changes and removes the stash | Restores changes but keeps the stash |

## Real-World Use Cases

- Urgent bug fixes.
- Switching branches without committing unfinished work.
- Temporary work backup.

---

# Task 5 – Cherry Pick

## What is Cherry Pick?

Cherry-pick copies a specific commit from one branch and applies it to another branch.

### Command

```bash
git cherry-pick <commit-id>
```

## Real-World Use Cases

- Apply urgent bug fixes.
- Copy one feature without merging the entire branch.
- Backport fixes to release branches.

## Possible Issues

- Merge conflicts.
- Duplicate commits.
- Confusing history if overused.

---

# Git Commands Practiced

```bash
git checkout -b <branch-name>
git merge <branch-name>
git merge --squash <branch-name>
git rebase main
git stash
git stash list
git stash pop
git stash apply
git stash push -m "message"
git cherry-pick <commit-id>
git cherry-pick --continue
git cherry-pick --abort
git log --oneline --graph --all
git status
git add .
git commit -m "message"
git push origin main
```

---

# Key Learnings

- Understood the difference between Fast-Forward Merge and Merge Commit.
- Learned how to resolve Merge Conflicts.
- Practiced Git Rebase to create a linear commit history.
- Used Squash Merge to combine multiple commits into one.
- Used Git Stash to save and restore unfinished work.
- Applied a single commit using Git Cherry Pick.
- Visualized commit history using:

```bash
git log --oneline --graph --all
```

---

# Conclusion

Day 24 provided practical experience with advanced Git workflows used in real software development and DevOps projects. These concepts help maintain a clean repository history, simplify collaboration, and improve version control practices.
