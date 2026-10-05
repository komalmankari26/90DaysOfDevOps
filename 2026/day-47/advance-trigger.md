# Day 47 – Advanced Triggers: PR Events, Cron Schedules & Event-Driven Pipelines

## Overview

This implementation covers advanced GitHub Actions event handling for:

* Pull Request lifecycle events
* PR validation gates
* Scheduled workflows with cron
* Path and branch filters
* `workflow_run` workflow chaining
* `repository_dispatch` external triggers
* Manual workflow execution with `workflow_dispatch`

---

# 1. Pull Request Lifecycle Events

## Workflow

File:

`.github/workflows/pr-lifecycle.yml`

```yaml
name: PR Lifecycle Events

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

jobs:
  pr-info:
    runs-on: ubuntu-latest

    steps:
      - name: Show PR Event Details
        run: |
          echo "Event type: ${{ github.event.action }}"
          echo "PR title: ${{ github.event.pull_request.title }}"
          echo "PR author: ${{ github.event.pull_request.user.login }}"
          echo "Source branch: ${{ github.event.pull_request.head.ref }}"
          echo "Target branch: ${{ github.event.pull_request.base.ref }}"

      - name: Check if PR was merged
        if: github.event.action == 'closed' && github.event.pull_request.merged == true
        run: |
          echo "PR was merged successfully!"
          echo "Merged PR: #${{ github.event.pull_request.number }}"
```

## Events demonstrated

| Event         | Meaning                                 |
| ------------- | --------------------------------------- |
| `opened`      | A new pull request is created           |
| `synchronize` | New commits are pushed to the PR branch |
| `reopened`    | A previously closed PR is reopened      |
| `closed`      | The PR is closed or merged              |

A `closed` event does not necessarily mean the PR was merged.

The merge condition is:

```yaml
if: github.event.action == 'closed' && github.event.pull_request.merged == true
```

---

# 2. PR Validation Workflow

## Workflow

File:

`.github/workflows/pr-checks.yml`

```yaml
name: PR Validation Checks

on:
  pull_request:
    branches:
      - main

jobs:
  file-size-check:
    name: Check File Sizes
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Check for files larger than 1 MB
        run: |
          echo "Checking files for size limit..."

          large_files=$(find . -type f \
            -not -path './.git/*' \
            -size +1M \
            -print)

          if [ -n "$large_files" ]; then
            echo "ERROR: Files larger than 1 MB found:"
            echo "$large_files"
            exit 1
          fi

          echo "All files are within the 1 MB limit."

  branch-name-check:
    name: Validate Branch Name
    runs-on: ubuntu-latest

    steps:
      - name: Check branch name
        env:
          BRANCH_NAME: ${{ github.head_ref }}
        run: |
          echo "Branch name: $BRANCH_NAME"

          if [[ "$BRANCH_NAME" =~ ^(feature|fix|docs)/.+ ]]; then
            echo "Branch name is valid."
          else
            echo "ERROR: Invalid branch name."
            echo "Use one of these formats:"
            echo "  feature/*"
            echo "  fix/*"
            echo "  docs/*"
            exit 1
          fi

  pr-body-check:
    name: Validate PR Description
    runs-on: ubuntu-latest

    steps:
      - name: Check PR body
        env:
          PR_BODY: ${{ github.event.pull_request.body }}
        run: |
          if [ -z "$PR_BODY" ]; then
            echo "::warning::PR description is empty."
            echo "Please add a meaningful description to the pull request."
          else
            echo "PR description is present."
          fi
```

## Validation rules

### File size

The workflow fails if a file exceeds:

```text
1 MB
```

### Branch naming

Accepted:

```text
feature/*
fix/*
docs/*
```

Example:

```text
feature/login
fix/docker-build
docs/readme-update
```

Rejected:

```text
day-47
test
my-branch
```

### PR body

An empty PR description generates a warning but does not fail the workflow.

---

# 3. Scheduled Workflows

## Workflow

File:

`.github/workflows/scheduled-tasks.yml`

```yaml
name: Scheduled Tasks

on:
  schedule:
    - cron: '30 2 * * 1'
    - cron: '0 */6 * * *'

  workflow_dispatch:

jobs:
  health-check:
    name: Scheduled Health Check
    runs-on: ubuntu-latest

    steps:
      - name: Show Triggering Schedule
        run: |
          echo "Workflow triggered by schedule: ${{ github.event.schedule }}"

      - name: Health Check
        run: |
          echo "Checking GitHub..."
          response=$(curl -s -o /dev/null -w "%{http_code}" https://github.com)

          echo "HTTP response code: $response"

          if [ "$response" -ne 200 ]; then
            echo "Health check failed."
            exit 1
          fi

          echo "Health check passed."
```

## Cron syntax

Cron format:

```text
minute hour day-of-month month day-of-week
```

### Monday at 2:30 AM UTC

```text
30 2 * * 1
```

This is:

```text
Monday
02:30 UTC
08:00 IST
```

### Every 6 hours

```text
0 */6 * * *
```

Runs at:

```text
00:00 UTC
06:00 UTC
12:00 UTC
18:00 UTC
```

---

## Additional cron expressions

### Every weekday at 9 AM IST

GitHub cron uses UTC.

IST is UTC+5:30.

9:00 AM IST = 3:30 AM UTC.

```text
30 3 * * 1-5
```

### First day of every month at midnight

```text
0 0 1 * *
```

---

## Why scheduled workflows may be delayed or skipped

GitHub notes that scheduled workflows can be delayed during periods of high load. Scheduled workflows may also be automatically disabled in repositories with no activity for an extended period.

For this reason, scheduled workflows should not always be treated as precise-time execution mechanisms.

---

# 4. Path and Branch Filters

## Workflow using `paths`

File:

`.github/workflows/smart-triggers.yml`

```yaml
name: Smart Triggers

on:
  push:
    branches:
      - main
      - 'release/**'
    paths:
      - 'src/**'
      - 'app/**'

jobs:
  smart-push:
    runs-on: ubuntu-latest

    steps:
      - name: Show Trigger Details
        run: |
          echo "Smart trigger workflow started"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"

      - name: Run Application Check
        run: |
          echo "Changes detected in src/ or app/"
          echo "Running application checks..."
```

This workflow runs only when:

* The branch is `main` or `release/*`
* Changes occur under `src/` or `app/`

Example:

```text
src/app.py + main
```

Triggers the workflow.

```text
README.md + main
```

Does not trigger the workflow.

---

# 5. `paths-ignore`

## Workflow

File:

`.github/workflows/docs-ignore.yml`

```yaml
name: Docs Ignore Trigger

on:
  push:
    branches:
      - main
      - 'release/**'
    paths-ignore:
      - '*.md'
      - 'docs/**'

jobs:
  code-change:
    runs-on: ubuntu-latest

    steps:
      - name: Show Trigger Details
        run: |
          echo "Workflow triggered by a non-documentation change."
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
```

## `paths` vs `paths-ignore`

### `paths`

Use when a workflow should run **only when specific paths change**.

```yaml
paths:
  - 'src/**'
  - 'app/**'
```

### `paths-ignore`

Use when a workflow should run for changes **except when changes are limited to specific paths**.

```yaml
paths-ignore:
  - '*.md'
  - 'docs/**'
```

This is useful for avoiding unnecessary CI/CD runs for documentation-only changes.

---

# 6. `workflow_run` — Chaining Workflows

## Test workflow

File:

`.github/workflows/tests.yml`

```yaml
name: Run Tests

on:
  push:

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run tests
        run: |
          echo "Running test suite..."
          echo "All tests passed."
```

## Deployment workflow

File:

`.github/workflows/deploy-after-tests.yml`

```yaml
name: Deploy After Tests

on:
  workflow_run:
    workflows: ["Run Tests"]
    types: [completed]

jobs:
  deploy:
    name: Deploy After Tests
    runs-on: ubuntu-latest

    steps:
      - name: Check Test Result
        if: github.event.workflow_run.conclusion == 'success'
        run: |
          echo "Tests completed successfully."
          echo "Starting deployment..."
          echo "Deployment completed successfully."

      - name: Test Failure Warning
        if: github.event.workflow_run.conclusion != 'success'
        run: |
          echo "::warning::The test workflow did not succeed."
          echo "Deployment will not proceed."
          exit 0
```

## Execution flow

```text
Push
  ↓
Run Tests
  ↓
Workflow completed
  ↓
Deploy After Tests
  ↓
Check conclusion
  ↓
Deploy only if successful
```

The triggering workflow result is available through:

```text
github.event.workflow_run.conclusion
```

---

# 7. `workflow_run` vs `workflow_call`

## `workflow_run`

`workflow_run` is useful when one workflow should react to another workflow completing.

Example:

```text
Tests
  ↓
Deploy
```

It is event-driven and works with independently triggered workflows.

## `workflow_call`

`workflow_call` is used to create a reusable workflow that another workflow explicitly calls.

Example:

```text
Build Workflow
      ↓
Reusable Build Workflow
```

It is designed for reusable CI/CD logic.

### Key difference

```text
workflow_run
→ React to another workflow completing.

workflow_call
→ Explicitly call a reusable workflow.
```

---

# 8. `repository_dispatch` — External Event Trigger

## Workflow

File:

`.github/workflows/external-trigger.yml`

```yaml
name: External Deployment Trigger

on:
  repository_dispatch:
    types: [deploy-request]

jobs:
  external-deploy:
    name: External Deployment
    runs-on: ubuntu-latest

    steps:
      - name: Show Deployment Request
        run: |
          echo "External deployment request received."
          echo "Environment: ${{ github.event.client_payload.environment }}"

      - name: Process Deployment
        run: |
          echo "Starting deployment..."
          echo "Target environment: ${{ github.event.client_payload.environment }}"
          echo "External deployment request processed successfully."
```

## Example external event

A GitHub CLI request can send:

```bash
gh api repos/<owner>/<repo>/dispatches \
  -f event_type=deploy-request \
  -f client_payload='{"environment":"production"}'
```

The workflow reads the payload using:

```text
github.event.client_payload.environment
```

Result:

```text
Environment: production
```

---

# 9. Real-World Use Cases for `repository_dispatch`

An external system can trigger a GitHub Actions pipeline when:

* A monitoring system detects a production event
* A release-management system approves a deployment
* A Slack/ChatOps bot requests a deployment
* An external CI system completes successfully
* Infrastructure changes require application deployment
* An external platform publishes a new version

This enables GitHub Actions to participate in event-driven DevOps pipelines beyond GitHub-native events.

---

# 10. Day 47 Workflow Files

The Day 47 workflows created are:

```text
.github/workflows/
├── pr-lifecycle.yml
├── pr-checks.yml
├── scheduled-tasks.yml
├── smart-triggers.yml
├── docs-ignore.yml
├── tests.yml
├── deploy-after-tests.yml
└── external-trigger.yml
```

Existing workflows from previous days remain in the repository as well.

---

# 11. Verification Completed

## PR validation

The branch policy was tested with:

```text
day-47
```

The workflow correctly rejected it because it did not match:

```text
feature/*
fix/*
docs/*
```

A correctly named branch was then used:

```text
feature/day-47-advanced-triggers
```

The branch-name validation passed.

## Scheduled workflow

The scheduled workflow was manually executed using:

```text
workflow_dispatch
```

The health check returned:

```text
HTTP response code: 200
Health check passed.
```

## Workflow chaining

The workflow chain was verified:

```text
Run Tests
     ↓
Deploy After Tests
```

The deployment workflow executed after the test workflow completed successfully.

---

# 12. Key Takeaways

Advanced GitHub Actions triggers allow CI/CD pipelines to respond intelligently to different events instead of running everything on every push.

Key concepts demonstrated:

```text
pull_request types
        ↓
opened / synchronize / reopened / closed

schedule
        ↓
cron-based automation

paths / paths-ignore
        ↓
Selective workflow execution

workflow_run
        ↓
Workflow chaining

repository_dispatch
        ↓
External event-driven automation

workflow_dispatch
        ↓
Manual testing and execution
```

These patterns help reduce unnecessary CI/CD execution, enforce repository policies, automate scheduled operations, and build event-driven deployment pipelines.

---

# 13. Suggested Screenshots for Documentation

Include screenshots showing:

1. PR validation checks passing
2. The intentionally failed branch-name check for `day-47`
3. PR Lifecycle Events showing `opened`
4. Scheduled Tasks manual run
5. Health check returning HTTP 200
6. `Run Tests` workflow completing successfully
7. `Deploy After Tests` running afterward
8. GitHub Actions workflow list showing the Day 47 workflows

---

# 14. Submission

The notes file should be stored at:

```text
2026/day-47/day-47-advanced-triggers.md
```

Commit and push with:

```bash
git add 2026/day-47/day-47-advanced-triggers.md
git commit -m "Add Day 47 advanced triggers notes"
git push origin main
```

If the repository workflow requires a branch and PR for documentation changes, create the appropriate feature branch and PR instead.

---

## Final Architecture

```text
                    GitHub Actions
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
 Pull Requests          Scheduled          Push Events
       │                   │                   │
       ▼                   ▼                   ▼
 PR Validation        Health Check       Path Filters
       │
       ▼
 Branch Policy
       │
       ▼
    Tests
       │
       ▼
 workflow_run
       │
       ▼
   Deployment

External Systems
       │
       ▼
repository_dispatch
       │
       ▼
External Deployment
```

**Day 47 demonstrates event-driven GitHub Actions design rather than simple push-based automation.**
