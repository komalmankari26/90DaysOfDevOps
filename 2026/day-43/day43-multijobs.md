# Day 43 – Jobs, Steps & Workflow Logic

## Overview

Day 43 focused on building GitHub Actions workflows with multiple jobs, environment variables, job outputs, conditionals, and job dependencies.

---

## 1. Multi-Job Workflow

The workflow contains three jobs:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building the app"

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running tests"

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying"
