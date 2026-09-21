# Day 42 – GitHub Actions Runners

## Overview

GitHub Actions workflows run on runners. A runner is the machine that executes the jobs and steps defined in a workflow.

There are two main types:

- GitHub-hosted runners
- Self-hosted runners

---

## Task 1 – GitHub-Hosted Runners

A workflow was created using three GitHub-hosted environments:

- Ubuntu
- Windows
- macOS

Each job displayed:

- Operating system
- Hostname
- Current user

### Key Learning

GitHub-hosted runners are managed by GitHub and provide clean environments for running GitHub Actions jobs.

Multiple jobs can run in parallel when runner capacity is available.

---

## Task 2 – Preinstalled Tools

The Ubuntu GitHub-hosted runner was used to check common development tools:

```bash
docker --version
python --version
node --version
git --version
