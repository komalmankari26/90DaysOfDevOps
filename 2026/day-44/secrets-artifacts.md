# Day 44 – Secrets, Artifacts & Running Real Tests in CI

## Overview

Today the CI pipeline moved beyond basic workflow execution and started doing real CI/CD work:

* Managing sensitive values with GitHub Secrets
* Passing secrets securely as environment variables
* Creating and uploading artifacts
* Sharing artifacts between jobs
* Running real scripts and tests in CI
* Using caching to improve workflow performance

---

# Task 1 – GitHub Secrets

## Objective

Learn how to store sensitive information securely using GitHub Secrets and access it from a workflow without exposing the actual value.

## Secret Created

Repository → **Settings → Secrets and variables → Actions**

Created secret:

```text
MY_SECRET_MESSAGE
```

## Workflow

The secret was accessed using:

```yaml
${{ secrets.MY_SECRET_MESSAGE }}
```

The workflow verified whether the secret was available without printing its actual value.

Example:

```yaml
- name: Check Secret
  run: |
    if [ -n "${{ secrets.MY_SECRET_MESSAGE }}" ]; then
      echo "The secret is set: true"
    else
      echo "The secret is set: false"
    fi
```

## What happens if the secret is printed directly?

If a secret is referenced directly in a GitHub Actions log, GitHub automatically masks the secret value.

Example:

```yaml
- name: Test Secret
  run: echo "${{ secrets.MY_SECRET_MESSAGE }}"
```

GitHub masks the secret in the workflow logs instead of displaying the actual value.

## Why should secrets never be printed in CI logs?

Secrets should never be intentionally printed because CI logs may be viewed by people with access to the repository or retained as part of workflow history.

Exposing credentials can lead to:

* Unauthorized access
* Credential theft
* Security breaches
* Compromised cloud or deployment environments
* Accidental exposure through logs or debugging output

**Best practice:** Use GitHub Secrets and only expose secrets to the specific steps that need them.

---

# Task 2 – Secrets as Environment Variables

Secrets were passed to workflow steps through environment variables instead of hardcoding sensitive values.

Example:

```yaml
- name: Use Secret
  env:
    MY_SECRET_MESSAGE: ${{ secrets.MY_SECRET_MESSAGE }}
  run: |
    echo "Secret is available to the script"
```

This approach keeps sensitive values outside the source code.

## Docker Secrets

The following repository secrets were also added for future Docker work:

```text
DOCKER_USERNAME
DOCKER_TOKEN
```

These will be used in the Day 45 Docker workflow.

---

# Task 3 – Upload Artifacts

## Objective

Artifacts allow files generated during a workflow to be saved and downloaded after the workflow completes.

A test report/log file was generated during the workflow.

Example:

```yaml
- name: Generate Report
  run: |
    echo "CI Test Report" > test-report.txt
    echo "Tests completed successfully" >> test-report.txt
```

The file was uploaded using:

```yaml
- name: Upload Test Report
  uses: actions/upload-artifact@v4
  with:
    name: test-report
    path: test-report.txt
```

## Verification

After the workflow completed:

1. Opened the repository on GitHub
2. Opened the **Actions** tab
3. Opened the successful workflow run
4. Located the **Artifacts** section
5. Downloaded the generated artifact

### Screenshot

> Add screenshot of the artifact available for download here.

---

# Task 4 – Download Artifacts Between Jobs

## Objective

Learn how one job can create an artifact and another job can download and use it.

## Job 1 – Generate and Upload

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Create File
        run: |
          echo "Artifact created by Job 1" > output.txt

      - name: Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: shared-file
          path: output.txt
```

## Job 2 – Download and Use

```yaml
  test:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Download Artifact
        uses: actions/download-artifact@v4
        with:
          name: shared-file

      - name: Read Artifact
        run: cat output.txt
```

The `needs: build` dependency ensures that Job 2 waits for Job 1 to finish successfully.

## When would artifacts be used in a real pipeline?

Artifacts are useful when one stage of a pipeline produces files that need to be consumed later.

Examples:

* Test reports
* Build packages
* Compiled applications
* Logs
* Coverage reports
* Deployment packages
* Screenshots from automated tests

Artifacts make it possible to pass generated files between jobs and preserve important outputs from a workflow run.

---

# Task 5 – Run Real Tests in CI

## Objective

Run an actual script from previous DevOps practice inside GitHub Actions.

The workflow performs the following steps:

1. Checks out the repository
2. Installs required dependencies
3. Runs the script/test
4. Fails automatically if the script exits with a non-zero status

Example:

```yaml
- name: Checkout Code
  uses: actions/checkout@v4

- name: Run Tests
  run: python test_script.py
```

If the script exits successfully:

```text
Process completed with exit code 0
```

the workflow passes.

If the script exits with a non-zero code, GitHub Actions marks the workflow as failed.

## Failure Test

The script was intentionally broken to verify CI failure handling.

Result:

```text
Workflow failed ❌
```

This confirmed that CI correctly detects a failing test/script.

## Fixed Test

The script was corrected and executed again.

Result:

```text
Workflow passed ✅
```

This confirmed the complete CI feedback cycle:

```text
Code Change
    ↓
GitHub Actions
    ↓
Run Test
    ↓
Test Failed → Pipeline Red
    ↓
Fix Code
    ↓
Run Test Again
    ↓
Test Passed → Pipeline Green
```

### Screenshot

> Add screenshot of the successful GitHub Actions test run here.

---

# Task 6 – Caching

## Objective

Caching is used to store dependencies or other reusable files so future workflow runs can execute faster.

Example:

```yaml
- name: Cache Dependencies
  uses: actions/cache@v4
  with:
    path: ~/.cache
    key: ${{ runner.os }}-dependencies-${{ hashFiles('**/requirements.txt') }}
```

The cache key changes when the dependency file changes.

## What is being cached?

Depending on the workflow, caches can contain:

* Downloaded dependencies
* Package manager caches
* Build dependencies
* Other reusable files

## Where is the cache stored?

GitHub Actions stores the cache remotely and makes it available to future workflow runs when the cache key matches.

## Cache behavior

First run:

```text
Cache miss → Dependencies downloaded → Cache created
```

Later run:

```text
Cache hit → Existing cached data restored → Faster workflow
```

Caching can reduce workflow execution time by avoiding repeated downloads.

---

# Key GitHub Actions Concepts Learned

| Concept               | Purpose                                   |
| --------------------- | ----------------------------------------- |
| GitHub Secrets        | Securely store sensitive information      |
| Environment Variables | Pass values to workflow steps             |
| Artifacts             | Store files generated by workflows        |
| Upload Artifact       | Save files from a job                     |
| Download Artifact     | Retrieve files in another job             |
| `needs`               | Define job dependencies                   |
| Real Tests            | Validate application/script behavior      |
| Exit Code             | Determines success or failure             |
| Cache                 | Reuse dependencies and speed up workflows |

---

# Important GitHub Actions Syntax

## Secrets

```yaml
${{ secrets.SECRET_NAME }}
```

## Upload Artifact

```yaml
uses: actions/upload-artifact@v4
```

## Download Artifact

```yaml
uses: actions/download-artifact@v4
```

## Cache

```yaml
uses: actions/cache@v4
```

## Job Dependency

```yaml
needs: build
```

## Environment Variable

```yaml
env:
  MY_SECRET: ${{ secrets.MY_SECRET_MESSAGE }}
```

---

# What I Learned

Day 44 was an important step toward building practical CI pipelines.

I learned how to:

* Secure sensitive information using GitHub Secrets
* Avoid hardcoding credentials in workflow files
* Pass secrets safely to workflow steps
* Generate and store CI artifacts
* Transfer artifacts between jobs
* Run real tests automatically
* Verify that failing tests make the pipeline fail
* Fix the test and verify a successful pipeline
* Use caching to improve workflow performance

The biggest takeaway is that CI is not only about running commands. A useful CI pipeline should securely manage credentials, validate code, preserve important outputs, and provide fast feedback when something breaks.

---

# Day 44 CI Flow

```text
Developer Push
      ↓
GitHub Actions
      ↓
Checkout Code
      ↓
Load Secure Secrets
      ↓
Install Dependencies
      ↓
Restore Cache
      ↓
Run Tests
      ↓
   ┌───────────────┐
   │ Tests Passing?│
   └───────┬───────┘
       Yes │ No
           │
           ↓
     Pipeline Green
           │
           ↓
    Generate Report
           │
           ↓
      Upload Artifact
```

If tests fail:

```text
Test Failure
     ↓
Pipeline Red
     ↓
Fix Code
     ↓
Push Again
     ↓
Run Tests
     ↓
Pipeline Green
```

---

# Documentation / Evidence

## Artifact Download Screenshot

> Insert screenshot here showing the artifact available in the GitHub Actions run.

## Passing Test Screenshot

> Insert screenshot here showing the successful CI test run.

---

# Submission

The documentation file was created as:

```text
2026/day-44/day-44-secrets-artifacts.md
```

Git commands used:

```bash
git status
git add 2026/day-44/day-44-secrets-artifacts.md
git commit -m "Add Day 44 secrets artifacts and CI test notes"
git push
```

---

# Day 44 Completion Checklist

* [x] Created `MY_SECRET_MESSAGE` GitHub Secret
* [x] Used GitHub Secret securely in workflow
* [x] Learned secret masking in GitHub Actions logs
* [x] Added `DOCKER_USERNAME`
* [x] Added `DOCKER_TOKEN`
* [x] Passed secrets using environment variables
* [x] Generated a workflow artifact
* [x] Uploaded artifact using `upload-artifact@v4`
* [x] Downloaded artifact from Actions
* [x] Shared artifact between jobs
* [x] Ran a real script/test in CI
* [x] Verified failing test makes pipeline red
* [x] Fixed test and verified pipeline becomes green
* [x] Added caching
* [x] Documented Day 44 work

---

# Key Takeaway

**Secrets protect sensitive data.
Artifacts preserve useful outputs.
Tests validate the code.
Caching makes CI faster.**

Together, these features turn GitHub Actions into a more practical and production-oriented CI pipeline.

---

## Day 44 Status

**Completed — Secrets, Artifacts, Real CI Tests & Caching**
