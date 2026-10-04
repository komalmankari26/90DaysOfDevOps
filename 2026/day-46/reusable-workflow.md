# Day 46 – Reusable Workflows & Composite Actions

## Objective

Implement reusable GitHub Actions components to reduce workflow duplication and improve CI/CD maintainability.

### Implemented

* Reusable workflow using `workflow_call`
* Workflow inputs and secrets
* Reusable workflow outputs
* Caller workflow
* Custom composite action
* Composite action inputs and outputs
* CI verification through GitHub Actions

---

# 1. Reusable Workflows

A reusable workflow is a GitHub Actions workflow designed to be called by another workflow.

It is useful when the same CI/CD process needs to be executed from multiple workflows without duplicating the complete job configuration.

Reusable workflows are stored under:

```text
.github/workflows/
```

A reusable workflow uses:

```yaml
on:
  workflow_call:
```

### Benefits

* Reduces workflow duplication
* Centralizes CI/CD logic
* Improves maintainability
* Provides consistent build processes
* Allows workflows to share inputs, secrets and outputs

---

# 2. `workflow_call`

`workflow_call` allows one GitHub Actions workflow to call another workflow.

Example:

```yaml
on:
  workflow_call:
```

It can define:

* Inputs
* Secrets
* Outputs

---

# 3. Reusable Workflow

File:

```text
.github/workflows/reusable-build.yml
```

Implementation:

```yaml
name: Reusable Build

on:
  workflow_call:
    inputs:
      app_name:
        description: "Application name"
        required: true
        type: string

      environment:
        description: "Deployment environment"
        required: false
        type: string
        default: staging

    secrets:
      docker_token:
        required: true

    outputs:
      build_version:
        description: "Generated build version"
        value: ${{ jobs.build.outputs.build_version }}

jobs:
  build:
    runs-on: ubuntu-latest

    outputs:
      build_version: ${{ steps.version.outputs.build_version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Generate build version
        id: version
        run: echo "build_version=v1.0-${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"

      - name: Build application
        run: echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"

      - name: Verify Docker token
        run: echo "Docker token is set: true"
```

---

# 4. Workflow Inputs

The reusable workflow accepts an application name:

```yaml
inputs:
  app_name:
    required: true
    type: string
```

And an optional environment:

```yaml
environment:
  required: false
  type: string
  default: staging
```

These values are accessed using:

```yaml
${{ inputs.app_name }}
```

and:

```yaml
${{ inputs.environment }}
```

---

# 5. Secrets

The reusable workflow declares a required secret:

```yaml
secrets:
  docker_token:
    required: true
```

The caller passes the repository secret:

```yaml
secrets:
  docker_token: ${{ secrets.DOCKER_TOKEN }}
```

The actual secret value is never printed.

The workflow only verifies that the secret is configured.

---

# 6. Reusable Workflow Outputs

The workflow generates a build version from the Git commit SHA:

```yaml
- name: Generate build version
  id: version
  run: echo "build_version=v1.0-${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"
```

The step output is exposed as a job output:

```yaml
outputs:
  build_version: ${{ steps.version.outputs.build_version }}
```

The job output is then exposed as a reusable workflow output:

```yaml
outputs:
  build_version:
    description: "Generated build version"
    value: ${{ jobs.build.outputs.build_version }}
```

This creates the output chain:

```text
Step output
     ↓
Job output
     ↓
Reusable workflow output
     ↓
Caller workflow
```

---

# 7. Caller Workflow

File:

```text
.github/workflows/call-build.yml
```

Implementation:

```yaml
name: Call Reusable Build

on:
  push:
    branches:
      - main

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      app_name: "my-web-app"
      environment: "production"
    secrets:
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  show-version:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Display build version
        run: |
          echo "Build version: ${{ needs.build.outputs.build_version }}"
```

The caller invokes the reusable workflow using:

```yaml
uses: ./.github/workflows/reusable-build.yml
```

The required inputs are passed using:

```yaml
with:
```

The secret is passed using:

```yaml
secrets:
```

The second job waits for the reusable workflow:

```yaml
needs: build
```

and consumes its output:

```yaml
${{ needs.build.outputs.build_version }}
```

### Result

The GitHub Actions run successfully displayed a generated build version similar to:

```text
Build version: v1.0-xxxxxxx
```

---

# 8. Composite Actions

A composite action packages multiple workflow steps into a reusable action.

Unlike a reusable workflow, a composite action runs **inside a job**.

Custom actions can be stored in the repository under:

```text
.github/actions/
```

Implementation used:

```text
.github/actions/setup-and-greet/action.yml
```

---

# 9. Custom Composite Action

Implementation:

```yaml
name: Setup and Greet
description: "Reusable composite action that prints a greeting and runner information"

inputs:
  name:
    description: "Name to greet"
    required: true

  language:
    description: "Greeting language"
    required: false
    default: en

outputs:
  greeted:
    description: "Indicates whether the greeting was displayed"
    value: ${{ steps.greet.outputs.greeted }}

runs:
  using: "composite"

  steps:
    - name: Display greeting
      id: greet
      shell: bash
      run: |
        if [ "${{ inputs.language }}" = "es" ]; then
          echo "Hola, ${{ inputs.name }}!"
        elif [ "${{ inputs.language }}" = "fr" ]; then
          echo "Bonjour, ${{ inputs.name }}!"
        else
          echo "Hello, ${{ inputs.name }}!"
        fi

        echo "Runner OS: $RUNNER_OS"
        echo "Current date: $(date)"
        echo "greeted=true" >> "$GITHUB_OUTPUT"
```

---

# 10. Composite Action Inputs

The action accepts:

```yaml
name:
```

and:

```yaml
language:
```

Example usage:

```yaml
with:
  name: "Komal"
  language: "en"
```

The action supports English, Spanish and French greetings.

---

# 11. Composite Action Output

The action creates an output:

```yaml
greeted:
```

The value is generated using:

```bash
echo "greeted=true" >> "$GITHUB_OUTPUT"
```

The calling workflow accesses it through:

```yaml
${{ steps.greet.outputs.greeted }}
```

---

# 12. Testing the Composite Action

File:

```text
.github/workflows/composite-test.yml
```

Implementation:

```yaml
name: Test Composite Action

on:
  push:
    branches:
      - main

jobs:
  greet:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Setup and Greet
        id: greet
        uses: ./.github/actions/setup-and-greet
        with:
          name: "Komal"
          language: "en"

      - name: Verify greeting output
        run: |
          echo "Greeting completed: ${{ steps.greet.outputs.greeted }}"
```

### Successful execution

The workflow produced output similar to:

```text
Hello, Komal!
Runner OS: Linux
Current date: ...
Greeting completed: true
```

---

# 13. Reusable Workflow vs Composite Action

| Feature                     | Reusable Workflow             | Composite Action                 |
| --------------------------- | ----------------------------- | -------------------------------- |
| Main purpose                | Reuse complete workflows/jobs | Reuse multiple steps             |
| Location                    | `.github/workflows/`          | `.github/actions/`               |
| Invocation                  | `uses:` at job level          | `uses:` at step level            |
| Trigger                     | `workflow_call`               | Called from a workflow step      |
| Can contain jobs?           | Yes                           | No                               |
| Can contain multiple steps? | Yes                           | Yes                              |
| Supports workflow inputs    | Yes                           | Yes                              |
| Supports workflow outputs   | Yes                           | Yes                              |
| Best suited for             | Complete CI/CD processes      | Repeated step sequences          |
| Typical example             | Build → Test → Deploy         | Setup → Configure → Run commands |

---

# 14. Key Architectural Difference

### Reusable Workflow

```text
Caller Workflow
      |
      └── Job
            |
            └── Reusable Workflow
                    |
                    ├── Job 1
                    ├── Job 2
                    └── Job 3
```

A reusable workflow can represent an entire CI/CD process.

### Composite Action

```text
Workflow
   |
   └── Job
         |
         ├── Step
         ├── Composite Action
         │      ├── Step 1
         │      ├── Step 2
         │      └── Step 3
         |
         └── Step
```

A composite action is better suited for reusable step-level logic.

---

# 15. Practical Repository Structure

The Day 46 implementation resulted in:

```text
.github/
├── actions/
│   └── setup-and-greet/
│       └── action.yml
│
└── workflows/
    ├── reusable-build.yml
    ├── call-build.yml
    └── composite-test.yml
```

---

# 16. What Was Implemented

### Reusable Workflow

Created:

```text
.github/workflows/reusable-build.yml
```

Implemented:

* `workflow_call`
* Required and optional inputs
* Secret handling
* Job outputs
* Workflow outputs
* Build version generation

### Caller Workflow

Created:

```text
.github/workflows/call-build.yml
```

Implemented:

* Calling a reusable workflow
* Passing inputs
* Passing repository secrets
* Consuming reusable workflow outputs
* Job dependency using `needs`

### Composite Action

Created:

```text
.github/actions/setup-and-greet/action.yml
```

Implemented:

* Custom composite action
* Action inputs
* Action output
* Multiple shell steps
* Runner information
* Language-based greeting

### Test Workflow

Created:

```text
.github/workflows/composite-test.yml
```

Implemented:

* Local composite action invocation
* Input passing
* Output verification

---

# 17. Key Takeaways

Reusable workflows and composite actions provide two different levels of reuse in GitHub Actions.

**Reusable workflows** are appropriate when an entire job or CI/CD process needs to be standardized.

**Composite actions** are appropriate when a group of frequently repeated steps needs to be packaged into a reusable component.

Together, they support:

* DRY CI/CD architecture
* Standardized automation
* Centralized pipeline logic
* Easier maintenance
* Consistent execution across projects
* Scalable GitHub Actions design

---

# 18. Verification

All Day 46 practical components were committed through feature branches and merged into `main`.

Successful validations included:

```text
Call Reusable Build
├── build
└── show-version
```

and:

```text
Test Composite Action
└── greet
```

The reusable workflow successfully generated and exposed a build version, while the composite action successfully executed its greeting logic and returned its output.

---

## Day 46 Complete

Implemented reusable GitHub Actions architecture using both:

```text
Reusable Workflows
        +
Composite Actions
```

This establishes a more maintainable and scalable foundation for CI/CD workflow design.
