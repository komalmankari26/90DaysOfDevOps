# Day 41 – Triggers & Matrix Builds

## Task

The pipeline originally runs on push. In Day 41, I learned different ways to trigger GitHub Actions workflows and how to run the same job across multiple environments using matrix builds.

---

## Expected Output

* Pull Request workflow
* Scheduled workflow trigger
* Manual workflow trigger
* Matrix workflow
* Exclude and fail-fast configuration
* Notes file: `day-41-triggers.md`

---

# Task 1: Trigger on Pull Request

## Workflow File

`.github/workflows/pr-check.yml`

```yaml
name: PR Check

on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - synchronize

jobs:
  pr-check:
    runs-on: ubuntu-latest

    steps:
      - name: Print PR Branch
        run: echo "PR check running for branch: ${{ github.head_ref }}"
```

## What I Learned

* `pull_request` triggers a workflow when activity happens on a Pull Request.
* `branches: [main]` means the PR is targeting the `main` branch.
* `opened` runs the workflow when a PR is created.
* `synchronize` runs the workflow when new commits are pushed to the PR branch.
* `${{ github.head_ref }}` gives the name of the source branch of the Pull Request.

## Verification

I created a new branch, pushed a commit, and opened a Pull Request against `main`.

The workflow appeared and ran on the Pull Request page.

---

# Task 2: Scheduled Trigger

## Cron Configuration

A scheduled workflow can be configured using the `schedule` trigger.

Example:

```yaml
on:
  schedule:
    - cron: '0 0 * * *'
```

This runs the workflow every day at **00:00 UTC (midnight UTC)**.

## Cron Expression for Every Monday at 9 AM

```text
0 9 * * 1
```

### Cron Breakdown

| Field        | Value | Meaning                |
| ------------ | ----: | ---------------------- |
| Minute       |   `0` | At minute 0            |
| Hour         |   `9` | At 9 AM                |
| Day of Month |   `*` | Every day of the month |
| Month        |   `*` | Every month            |
| Day of Week  |   `1` | Monday                 |

**Answer:**

```text
0 9 * * 1
```

> Note: GitHub Actions cron schedules use UTC.

---

# Task 3: Manual Trigger

## Workflow File

`.github/workflows/manual.yml`

```yaml
name: Manual Workflow

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Enter environment name'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

jobs:
  manual:
    runs-on: ubuntu-latest

    steps:
      - name: Print Environment
        run: echo "Selected environment: ${{ inputs.environment }}"
```

## What I Learned

* `workflow_dispatch` allows a workflow to be started manually.
* Inputs can be provided when clicking **Run workflow**.
* The `environment` input accepts:

  * `staging`
  * `production`
* The selected value can be accessed using:

```text
${{ inputs.environment }}
```

## Verification

I went to:

**GitHub → Actions → Manual Workflow → Run workflow**

I selected an environment and ran the workflow.

The selected environment was printed in the workflow output.

---

# Task 4: Matrix Builds

## Workflow File

`.github/workflows/matrix.yml`

```yaml
name: Matrix Build

on:
  push:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version:
          - '3.10'
          - '3.11'
          - '3.12'

    steps:
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Print Python Version
        run: python --version
```

## What I Learned

A matrix allows the same job to run multiple times with different values.

The Python versions used were:

* Python 3.10
* Python 3.11
* Python 3.12

Therefore, the workflow creates **3 jobs**.

These jobs can run in parallel.

---

# Matrix With Operating Systems

The matrix can be extended to include two operating systems:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
    python-version:
      - '3.10'
      - '3.11'
      - '3.12'
```

The job should then use:

```yaml
runs-on: ${{ matrix.os }}
```

## Total Jobs

There are:

**2 operating systems × 3 Python versions = 6 jobs**

So the matrix creates **6 total combinations/jobs**.

| Operating System | Python 3.10 | Python 3.11 | Python 3.12 |
| ---------------- | ----------- | ----------- | ----------- |
| Ubuntu           | ✓           | ✓           | ✓           |
| Windows          | ✓           | ✓           | ✓           |

---

# Task 5: Exclude & Fail-Fast

## Excluding a Combination

One specific combination can be removed using `exclude`.

Example:

```yaml
strategy:
  fail-fast: false

  matrix:
    os:
      - ubuntu-latest
      - windows-latest

    python-version:
      - '3.10'
      - '3.11'
      - '3.12'

    exclude:
      - os: windows-latest
        python-version: '3.10'
```

This excludes:

```text
Windows + Python 3.10
```

## Number of Jobs After Exclusion

Originally:

```text
2 operating systems × 3 Python versions = 6 jobs
```

After excluding one combination:

```text
6 - 1 = 5 jobs
```

Therefore, **5 jobs** run.

---

# Fail-Fast

The matrix can use:

```yaml
fail-fast: false
```

## `fail-fast: true`

`fail-fast` is `true` by default.

If one matrix job fails, GitHub Actions cancels the other currently running matrix jobs.

Example:

```text
Job 1 → Success
Job 2 → Failure
Job 3 → Cancelled
Job 4 → Cancelled
```

## `fail-fast: false`

When `fail-fast` is `false`, a failure in one matrix job does not automatically cancel the other matrix jobs.

Example:

```text
Job 1 → Success
Job 2 → Failure
Job 3 → Success
Job 4 → Success
```

The remaining jobs continue running.

## Main Difference

| Setting            | Behavior                                           |
| ------------------ | -------------------------------------------------- |
| `fail-fast: true`  | Cancels in-progress matrix jobs when one job fails |
| `fail-fast: false` | Allows the remaining matrix jobs to continue       |

---

# Important GitHub Actions Concepts Learned

## 1. Pull Request Trigger

```yaml
on:
  pull_request:
```

Used to automatically run workflows when Pull Request events occur.

## 2. Schedule Trigger

```yaml
on:
  schedule:
    - cron: '0 0 * * *'
```

Used to run workflows automatically according to a schedule.

## 3. Manual Trigger

```yaml
on:
  workflow_dispatch:
```

Used to manually start a workflow from the Actions tab.

## 4. Matrix Strategy

```yaml
strategy:
  matrix:
```

Used to run the same job with multiple configurations.

## 5. Exclude

```yaml
exclude:
```

Used to remove specific combinations from a matrix.

## 6. Fail-Fast

```yaml
fail-fast: false
```

Controls whether other matrix jobs are cancelled when one job fails.

---

# Workflow Files Created

The following workflow files were created/used during Day 41:

```text
.github/
└── workflows/
    ├── pr-check.yml
    ├── manual.yml
    └── matrix.yml
```

The scheduled trigger was also added to a workflow as required by the task.

---

# Verification Checklist

* [x] Pull Request trigger tested
* [x] Pull Request workflow appeared on the PR page
* [x] Scheduled trigger configured
* [x] Midnight UTC cron configured
* [x] Monday 9 AM cron expression documented
* [x] Manual workflow created
* [x] Manual workflow triggered from Actions
* [x] Environment input tested
* [x] Matrix build created
* [x] Python 3.10 tested
* [x] Python 3.11 tested
* [x] Python 3.12 tested
* [x] Multiple operating systems added to matrix
* [x] Matrix combinations understood
* [x] One matrix combination excluded
* [x] `fail-fast: false` tested
* [x] Difference between `fail-fast: true` and `false` understood

---

# Screenshots

Add screenshots of the completed workflow runs below.

## Task 1 – Pull Request Workflow

*Add PR workflow screenshot here.*

## Task 2 – Scheduled Workflow

*Add scheduled workflow screenshot here.*

## Task 3 – Manual Workflow

*Add manual workflow screenshot here.*

## Task 4 – Matrix Build

*Add matrix jobs screenshot here.*

## Task 5 – Exclude & Fail-Fast

*Add exclude/fail-fast screenshot here.*

---

# Submission

The notes file should be placed at:

```text
2026/day-41/day-41-triggers.md
```

Then commit and push it to the fork.

Example Git commands:

```bash
git add 2026/day-41/day-41-triggers.md
git commit -m "Add Day 41 triggers and matrix build notes"
git push
```

---

# Day 41 Summary

Day 41 covered different ways of triggering GitHub Actions workflows:

**Pull Request → Schedule → Manual → Matrix → Exclude → Fail-Fast**

The main concept learned is that GitHub Actions can automatically run workflows based on different events and can use matrix strategies to test the same workflow across multiple configurations.

---

# Learn in Public

Share the matrix build screenshot to show the multiple jobs running in parallel.

`#90DaysOfDevOps`
`#DevOpsKaJosh`
`#TrainWithShubham`

**Day 41 completed – Triggers & Matrix Builds.** 🚀
