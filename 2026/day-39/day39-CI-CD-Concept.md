# Day 39 – CI/CD Concepts

## Overview

CI/CD is a software development practice used to automate the process of integrating code, testing applications, building software, and delivering or deploying applications.

CI/CD is not a single tool. Tools such as GitHub Actions, Jenkins, GitLab CI/CD, and CircleCI can be used to implement CI/CD practices.

---

# Task 1 – The Problem

## What can go wrong when 5 developers manually deploy?

When multiple developers push code to the same repository and deployment is done manually, several problems can occur:

1. Code conflicts can occur between developers.
2. Bugs can reach production without automated testing.
3. Someone may forget an important deployment step.
4. Different development and production environments can cause failures.
5. Manual deployment takes time.
6. Human errors can cause downtime.
7. It becomes difficult to deploy frequently and consistently.
8. Dependency or configuration differences can cause unexpected failures.
9. It can be difficult to know exactly which version was deployed.
10. Repeating the same deployment process manually is inefficient.

---

## What does "It works on my machine" mean?

"It works on my machine" means that an application works correctly in a developer's environment but may fail in another environment.

The environments may have different:

* Operating systems
* Programming language versions
* Dependencies
* Database versions
* Environment variables
* Configuration
* Installed software

For example:

Developer machine:

```text
Python 3.12
MySQL 8.0
Dependency version 1.5

Application → WORKS
```

Production server:

```text
Python 3.10
MySQL 5.7
Dependency version 1.2

Application → FAILS
```

Docker can help provide a consistent application environment, while CI/CD automates building, testing, and delivery/deployment.

---

## How many times a day can a team safely deploy manually?

There is no fixed number of deployments that is considered universally safe.

Manual deployment becomes increasingly difficult and error-prone as deployment frequency increases.

CI/CD allows teams to automate repeatable checks and deployment processes so applications can be released more frequently and consistently.

---

# Task 2 – CI vs CD

## 1. Continuous Integration (CI)

Continuous Integration means developers frequently integrate their code changes into a shared repository.

Automated processes can then build the application and run tests whenever changes are pushed or a pull request is created.

### CI catches:

* Build errors
* Test failures
* Code integration problems
* Dependency problems
* Some bugs before they reach production

### Example

A developer changes `app.py` and pushes the code to GitHub.

```text
Developer
    ↓
git push
    ↓
GitHub
    ↓
CI
    ↓
Install dependencies
    ↓
Run tests
    ↓
PASS / FAIL
```

---

# 2. Continuous Delivery

Continuous Delivery means that software is automatically built, tested, and prepared so that it is always in a releasable state.

The application may be deployed to a staging environment automatically, while production deployment can still require human approval.

### Example

```text
Developer
    ↓
Push code
    ↓
Build
    ↓
Test
    ↓
Create Docker image
    ↓
Deploy to staging
    ↓
Human approval
    ↓
Production
```

---

# 3. Continuous Deployment

Continuous Deployment goes one step further than Continuous Delivery.

When the automated checks pass, the application is automatically deployed to production without requiring manual approval for each release.

### Example

```text
Developer
    ↓
Push code
    ↓
Build
    ↓
Test
    ↓
Create Docker image
    ↓
Automatic deployment
    ↓
Production
```

---

## CI vs Continuous Delivery vs Continuous Deployment

| Concept                | Main Purpose                             | Production Deployment      |
| ---------------------- | ---------------------------------------- | -------------------------- |
| Continuous Integration | Build and test code frequently           | Usually not the main focus |
| Continuous Delivery    | Keep software ready for release          | May require human approval |
| Continuous Deployment  | Automatically release successful changes | Automatic                  |

### Easy way to remember

```text
CI
Build + Test

Continuous Delivery
Build + Test + Ready to Release

Continuous Deployment
Build + Test + Automatically Deploy
```

---

# Task 3 – Pipeline Anatomy

A CI/CD pipeline is made up of different components.

---

## 1. Trigger

A trigger is the event that starts a pipeline or workflow.

Common triggers include:

* Git push
* Pull request
* Manual workflow execution
* Scheduled execution

Example:

```yaml
on:
  push:
    branches: [main]
```

This means the workflow can start when code is pushed to the `main` branch.

---

## 2. Stage

A stage is a logical phase of the pipeline.

Common stages are:

```text
Build
Test
Deploy
```

Example:

```text
Stage 1 → Test
Stage 2 → Build
Stage 3 → Deploy
```

---

## 3. Job

A job is a unit of work executed by a runner.

For example, a test job could contain:

```text
Checkout code
Install dependencies
Run tests
```

Another job could build a Docker image.

```text
Build Docker image
Tag image
```

---

## 4. Step

A step is an individual command or action inside a job.

Example:

```text
Job: Test

Step 1 → Checkout code
Step 2 → Install Python
Step 3 → Install dependencies
Step 4 → Run tests
```

The basic relationship is:

```text
Stage
  ↓
Job
  ↓
Steps
```

---

## 5. Runner

A runner is the machine that executes a job.

GitHub Actions can use runners such as:

```text
Ubuntu
Windows
macOS
```

Example:

```yaml
runs-on: ubuntu-latest
```

This means the job runs on an Ubuntu GitHub-hosted runner.

---

## 6. Artifact

An artifact is an output produced by a job that can be stored or used later.

Examples include:

```text
Build package
Test report
Compiled application
Log files
ZIP file
```

Conceptually:

```text
Job
 ↓
Produces output
 ↓
Artifact
```

---

# Pipeline Anatomy – Simple View

```text
Trigger
   ↓
Stage
   ↓
Job
   ↓
Steps
   ↓
Runner executes the job
   ↓
Artifact may be produced
```

---

# Task 4 – CI/CD Pipeline Diagram

## Scenario

A developer pushes code to GitHub.

The application is:

1. Tested
2. Built into a Docker image
3. Deployed to a staging server

---

## Pipeline Diagram

```text
                         DEVELOPER
                             |
                             | git push
                             ↓
                          GITHUB
                             |
                             | Trigger
                             ↓
                  ┌────────────────────┐
                  │      STAGE 1       │
                  │        TEST        │
                  └─────────┬──────────┘
                            |
                    Checkout source code
                            |
                    Install dependencies
                            |
                       Run tests
                            |
                         PASS
                            ↓
                  ┌────────────────────┐
                  │      STAGE 2       │
                  │       BUILD        │
                  └─────────┬──────────┘
                            |
                    Build Docker image
                            |
                      Tag the image
                            |
                            ↓
                      Docker Image
                            |
                  ┌────────────────────┐
                  │      STAGE 3       │
                  │      DEPLOY        │
                  └─────────┬──────────┘
                            |
                            ↓
                    STAGING SERVER
                            |
                            ↓
                       APPLICATION
```

---

# Pipeline Flow – Short Version

```text
Developer
    ↓
git push
    ↓
GitHub
    ↓
Trigger
    ↓
Test
    ↓
Build Docker Image
    ↓
Deploy
    ↓
Staging Server
```

---

# Task 5 – Explore CI/CD in the Wild

## Repository

For this task, choose a popular open-source GitHub repository such as:

* FastAPI
* React
* Kubernetes

Open the repository and look for:

```text
.github/
└── workflows/
    └── workflow-file.yml
```

Open one workflow YAML file.

Record the following:

### 1. What triggers it?

Look for:

```yaml
on:
```

Possible triggers:

```text
push
pull_request
schedule
workflow_dispatch
```

### 2. How many jobs does it have?

Look for:

```yaml
jobs:
```

Then count the jobs underneath it.

Example:

```yaml
jobs:
  test:
  
  build:
  
  deploy:
```

This would contain 3 jobs.

### 3. What does it do?

Read the steps and make your best guess.

Look for commands or actions such as:

```text
Checkout code
Install dependencies
Run tests
Build application
Build Docker image
Upload artifact
Deploy
```

### 4. What runner does it use?

Look for:

```yaml
runs-on:
```

For example:

```yaml
runs-on: ubuntu-latest
```

---

# Important CI/CD Concepts

## CI/CD is a practice, not just a tool

CI/CD describes a way of working and automating software delivery.

Tools implement the practice.

Examples:

```text
GitHub Actions
Jenkins
GitLab CI/CD
CircleCI
```

---

## A failed pipeline is not necessarily a bad thing

A pipeline failure can be useful.

For example:

```text
Developer pushes code
        ↓
Pipeline starts
        ↓
Tests run
        ↓
Test fails ❌
        ↓
Deployment stops
```

This can prevent broken code from reaching the next environment.

Therefore:

> A pipeline failing can mean that CI/CD successfully detected a problem.

---

# CI/CD in a Real Project

A typical application may use a pipeline like this:

```text
Developer
    ↓
Git Push
    ↓
GitHub
    ↓
CI/CD Pipeline
    ↓
┌───────────────┐
│ Test          │
└───────┬───────┘
        ↓
┌───────────────┐
│ Build         │
└───────┬───────┘
        ↓
┌───────────────┐
│ Docker Image  │
└───────┬───────┘
        ↓
┌───────────────┐
│ Staging       │
└───────┬───────┘
        ↓
┌───────────────┐
│ Production    │
└───────────────┘
```

---

# Key Takeaways

### CI

```text
Integrate code frequently
        +
Build
        +
Test
```

### Continuous Delivery

```text
CI
 +
Prepare software for release
 +
Keep it ready for deployment
```

### Continuous Deployment

```text
Continuous Delivery
 +
Automatic production deployment
```

---

# Day 39 Summary

Today I learned:

* Why CI/CD is needed.
* Problems with manual deployments.
* Meaning of "It works on my machine."
* Continuous Integration.
* Continuous Delivery.
* Continuous Deployment.
* Difference between Delivery and Deployment.
* Pipeline triggers.
* Pipeline stages.
* Pipeline jobs.
* Pipeline steps.
* Runners.
* Artifacts.
* How a CI/CD pipeline works.
* How to inspect workflows in a real GitHub repository.

---

# Final Mental Model

```text
                 DEVELOPER
                     |
                     | Push Code
                     ↓
                  GITHUB
                     |
                   Trigger
                     ↓
               ┌───────────┐
               │    TEST   │
               └─────┬─────┘
                     ↓
               ┌───────────┐
               │   BUILD   │
               └─────┬─────┘
                     ↓
              DOCKER IMAGE
                     ↓
               ┌───────────┐
               │  DEPLOY   │
               └─────┬─────┘
                     ↓
               STAGING SERVER
                     ↓
                APPLICATION
```

---

# Next Step

After completing the notes:

```bash
git status
```

Then:

```bash
git add 2026/day-39/day-39-cicd-concepts.md
```

Commit:

```bash
git commit -m "Add Day 39 CI/CD concepts notes"
```

Push:

```bash
git push
```

# Day 39 Complete ✅
