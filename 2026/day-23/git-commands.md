# Day 23 – Git Branching & Working with GitHub

## Task 1: Understanding Branches

### 1. What is a branch in Git?
A branch is an independent line of development in Git. It allows developers to work on new features, bug fixes, or experiments without affecting the main project.

### 2. Why do we use branches instead of committing everything to `main`?
- Keeps the main branch stable.
- Allows multiple developers to work simultaneously.
- Makes testing easier.
- Prevents unfinished work from affecting the production code.
- Simplifies code reviews before merging.

### 3. What is `HEAD` in Git?
`HEAD` is a pointer that refers to the current branch and the latest commit you are working on.

### 4. What happens to your files when you switch branches?
Git updates your working directory to match the selected branch. Files unique to that branch appear, and files not present in that branch disappear.

---

# Task 2 – Branching Commands

## Commands Used

### List all branches
```bash
git branch
```

### Create a new branch
```bash
git branch feature-1
```

### Switch to a branch
```bash
git switch feature-1
```

### Create and switch to a new branch
```bash
git switch -c feature-2
```

### Switch back to main
```bash
git switch main
```

### View commit history
```bash
git log --oneline
```

### Delete a branch
```bash
git branch -d feature-2
```

### Difference between `git switch` and `git checkout`

| git switch | git checkout |
|------------|--------------|
| Used only for switching branches. | Can switch branches and restore files. |
| Easier and safer for beginners. | Older command with multiple purposes. |

### Branch Verification
After creating a commit on `feature-1`, I switched back to `main` and verified that the commit was not available on the `main` branch.

---

# Task 3 – Push to GitHub

## Commands

### Add GitHub remote
```bash
git remote add origin git@github.com:komalmankari26/devops-git-practice.git
```

### Verify remote
```bash
git remote -v
```

### Push main branch
```bash
git push -u origin main
```

### Push feature branch
```bash
git switch feature-1
git push -u origin feature-1
```

### Difference between Origin and Upstream

**Origin**
- Refers to your own GitHub repository.
- It is the default remote created after cloning or adding a remote.

**Upstream**
- Refers to the original repository from which a project was forked.
- Used to keep your fork updated with the original project.

---

# Task 4 – Pull from GitHub

### Difference between `git fetch` and `git pull`

| git fetch | git pull |
|------------|-----------|
| Downloads new commits from the remote repository. | Downloads new commits and merges them into the current branch. |
| Does not modify the working directory. | Updates the local branch immediately. |
| Safer for reviewing changes first. | Faster when you want to update directly. |

---

# Task 5 – Clone vs Fork

## Difference between Clone and Fork

| Clone | Fork |
|--------|------|
| Creates a local copy of a repository. | Creates a copy of a repository under your GitHub account. |
| Git command. | GitHub feature. |
| Used when you have direct access to the repository. | Used when contributing to someone else's project. |

## When would you clone?
- To work on your own repository.
- To contribute to a repository you have write access to.

## When would you fork?
- To contribute to open-source projects.
- When you do not have write access to the original repository.

## How do you keep your fork in sync with the original repository?

```bash
git remote add upstream <original-repository-url>
git fetch upstream
git merge upstream/main
git push origin main
```

---

# Summary

Today I learned:
- Git branching
- Creating and switching branches
- Working with `git switch`
- Difference between `git switch` and `git checkout`
- Pushing branches to GitHub
- Understanding `origin` and `upstream`
- Difference between `git fetch` and `git pull`
- Difference between cloning and forking repositories
  <img width="1920" height="1080" alt="Screenshot (330)" src="https://github.com/user-attachments/assets/aeed2986-3525-4490-81cb-b27c4e3c846c" />
