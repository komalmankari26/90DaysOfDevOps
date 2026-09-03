# Day 40 – My First GitHub Actions Workflow

## 📌 Objective

Today I created my first GitHub Actions workflow and learned how GitHub can automatically run tasks whenever I push code to a repository.

The goal was to understand the basic structure of a CI/CD workflow and see it actually run on a GitHub-hosted Ubuntu runner.

---

# 1. Repository Setup

Repository name:

```text
github-actions-practice
```

Repository visibility:

```text
Public
```

Repository was cloned locally using:

```bash
git clone https://github.com/YOUR-USERNAME/github-actions-practice.git
```

Entered the repository:

```bash
cd github-actions-practice
```

Opened it in VS Code:

```bash
code .
```

---

# 2. GitHub Actions Folder Structure

Created the following folder structure:

```text
github-actions-practice/
└── .github/
    └── workflows/
        └── hello.yml
```

GitHub Actions workflow files are stored inside:

```text
.github/workflows/
```

Workflow files use the `.yml` or `.yaml` extension.

---

# 3. My First Workflow

File:

```text
.github/workflows/hello.yml
```

Initial workflow:

```yaml
name: Hello GitHub Actions

on:
  push:

jobs:
  greet:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Say hello
        run: echo "Hello from GitHub Actions!"
```

---

# 4. Understanding the Workflow

## `name:`

```yaml
name: Hello GitHub Actions
```

This gives the workflow a readable name.

The name appears in the GitHub Actions interface.

---

## `on:`

```yaml
on:
  push:
```

This tells GitHub **when the workflow should run**.

In this workflow, every push to the repository triggers the workflow.

Simple idea:

```text
git push
   ↓
GitHub receives the push
   ↓
GitHub Actions starts
```

---

## `jobs:`

```yaml
jobs:
```

This section defines the work that GitHub Actions needs to perform.

A workflow can contain one or multiple jobs.

Example:

```text
Workflow
   │
   ├── Job 1
   ├── Job 2
   └── Job 3
```

My workflow has one job:

```yaml
greet:
```

---

## `greet:`

```yaml
greet:
```

This is the ID of my job.

I named the job `greet` because its purpose is to greet the user and demonstrate GitHub Actions.

---

## `runs-on:`

```yaml
runs-on: ubuntu-latest
```

This specifies the operating system/environment where the job runs.

GitHub provides a hosted Ubuntu runner for this job.

Conceptually:

```text
GitHub
   ↓
Creates temporary Ubuntu runner
   ↓
Runs my job
   ↓
Runner finishes
```

---

## `steps:`

```yaml
steps:
```

Steps are the individual tasks performed by a job.

My workflow contains multiple steps.

For example:

```text
greet
 │
 ├── Checkout code
 ├── Say hello
 ├── Show date and time
 ├── Show branch name
 ├── List repository files
 └── Show operating system
```

---

## `uses:`

```yaml
uses: actions/checkout@v4
```

`uses:` allows us to use an existing GitHub Action.

Here I use:

```text
actions/checkout@v4
```

This checks out the repository code onto the GitHub Actions runner.

Instead of writing all the checkout logic ourselves, we use an existing action.

---

## `run:`

```yaml
run: echo "Hello from GitHub Actions!"
```

`run:` executes a shell command on the runner.

The command:

```bash
echo "Hello from GitHub Actions!"
```

prints:

```text
Hello from GitHub Actions!
```

Other commands I used:

```bash
date
```

```bash
ls -la
```

```bash
uname -a
```

---

## `name:` on a step

Example:

```yaml
- name: Say hello
```

This gives an individual step a readable name.

It makes the Actions interface easier to understand.

---

# 5. Extended Workflow

I updated my workflow to include more practical steps.

```yaml
name: Hello GitHub Actions

on:
  push:

jobs:
  greet:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Say hello
        run: echo "Hello from GitHub Actions!"

      - name: Show date and time
        run: date

      - name: Show branch name
        run: echo "Branch: ${{ github.ref_name }}"

      - name: List repository files
        run: ls -la

      - name: Show operating system
        run: uname -a
```

---

# 6. GitHub Branch Variable

I used:

```yaml
${{ github.ref_name }}
```

This is a GitHub-provided context value.

It gives the name of the branch that triggered the workflow.

For example:

```text
Branch: main
```

This is useful because workflows can behave differently depending on the branch.

---

# 7. Useful Linux Commands

### Show date and time

```bash
date
```

### List files

```bash
ls -la
```

The `-a` option includes hidden files.

This is useful for seeing folders such as:

```text
.github
```

### Show operating system/kernel information

```bash
uname -a
```

This helps us inspect the environment where the GitHub Actions job is running.

---

# 8. Git Commands Used

### Check repository status

```bash
git status
```

### Stage the workflow

```bash
git add .github/workflows/hello.yml
```

### Commit changes

```bash
git commit -m "Add first GitHub Actions workflow"
```

### Push to GitHub

```bash
git push origin main
```

Because my workflow is triggered by:

```yaml
on:
  push:
```

every push can create a new GitHub Actions run.

---

# 9. Intentionally Breaking the Pipeline

To understand failures, I added:

```yaml
- name: Intentional failure
  run: exit 1
```

`exit 1` tells the shell that the command failed.

The workflow then looked conceptually like:

```text
Checkout code          ✅
Say hello              ✅
Show date              ✅
Show branch            ✅
List files             ✅
Show operating system  ✅
Intentional failure    ❌
                         ↓
                    Job fails
```

The GitHub Actions run becomes red when the job fails.

---

# 10. How I Read a Failed Pipeline

When a workflow fails:

1. Open the repository on GitHub.
2. Click **Actions**.
3. Open the failed workflow run.
4. Click the failed job.
5. Find the step with the red ❌.
6. Open that step.
7. Read the command and error message.
8. Fix the problem.
9. Commit and push again.

The important thing is to find the **first failed step** and read its logs.

---

# 11. Fixing the Failed Pipeline

I removed:

```yaml
- name: Intentional failure
  run: exit 1
```

Then committed and pushed the fix:

```bash
git add .github/workflows/hello.yml
```

```bash
git commit -m "Fix failed workflow"
```

```bash
git push origin main
```

The next workflow run should become green.

---

# 12. My CI/CD Flow

What I learned today:

```text
Developer
   │
   │ git push
   ▼
GitHub Repository
   │
   ▼
GitHub Actions
   │
   ▼
Ubuntu Runner
   │
   ├── Checkout code
   │
   ├── Run commands
   │
   ├── Test/build/etc.
   │
   ▼
Success or Failure
```

In this exercise:

```text
git push
   ↓
Workflow triggered
   ↓
Runner started
   ↓
Repository checked out
   ↓
Commands executed
   ↓
Pipeline result
   ↓
Green ✅ / Red ❌
```

---

# 13. Key Concepts I Learned

### GitHub Actions

GitHub's automation platform for running workflows directly from a GitHub repository.

### Workflow

A YAML file that defines automated tasks.

Example:

```text
.github/workflows/hello.yml
```

### Job

A collection of steps that run together on a runner.

### Runner

The machine/environment that executes the workflow.

### Step

An individual task inside a job.

### Action

A reusable piece of automation.

Example:

```yaml
uses: actions/checkout@v4
```

### Shell command

A command executed using:

```yaml
run:
```

---

# 14. Final Workflow

My final working workflow:

```yaml
name: Hello GitHub Actions

on:
  push:

jobs:
  greet:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Say hello
        run: echo "Hello from GitHub Actions!"

      - name: Show date and time
        run: date

      - name: Show branch name
        run: echo "Branch: ${{ github.ref_name }}"

      - name: List repository files
        run: ls -la

      - name: Show operating system
        run: uname -a
```

---

# 15. Screenshot

Add my screenshot of the successful green GitHub Actions run below this section.

```text
[INSERT SCREENSHOT HERE]
```

The screenshot should show the successful workflow with the green checkmark.

---

# 16. Final Learning

Before Day 40, CI/CD was mostly a concept for me.

Today I actually created a workflow, pushed it to GitHub, watched GitHub create a runner, executed commands on that runner, intentionally caused a failure, read the logs, fixed the workflow, and got a green pipeline.

The basic idea I learned is:

```text
Code Change
     ↓
   Push
     ↓
GitHub Actions
     ↓
   Runner
     ↓
   Steps
     ↓
Success / Failure
```

This is the foundation for building more advanced CI/CD pipelines.
