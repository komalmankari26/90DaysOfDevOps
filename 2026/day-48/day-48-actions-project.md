# Day 48 – GitHub Actions: End-to-End CI/CD Project

## Project Overview

Built an end-to-end CI/CD pipeline using GitHub Actions for a Python Flask application containerized with Docker.

The implementation covers:

* Python Flask application
* Automated testing with pytest
* Docker image creation
* Reusable GitHub Actions workflows
* Pull Request CI validation
* Main branch CI/CD pipeline
* Docker Hub image publishing
* Production deployment workflow
* Scheduled container health checks
* Docker image versioning using `latest` and short Git SHA tags
* GitHub Actions job dependencies and workflow outputs

---

# 1. Application Structure

The project uses a simple Flask application with two endpoints:

### `/`

Returns application status.

Example response:

```json
{
  "application": "GitHub Actions Capstone",
  "status": "running"
}
```

### `/health`

Used as the application health endpoint.

Example response:

```json
{
  "status": "healthy"
}
```

---

# 2. Project Files

Important Day 48 files:

```text
git-action-practice/
│
├── app.py
├── requirements.txt
├── test_app.py
├── Dockerfile
├── README.md
├── .gitignore
│
├── .github/
│   └── workflows/
│       ├── reusable-build-test.yml
│       ├── reusable-docker.yml
│       ├── pr-pipeline.yml
│       ├── main-pipeline.yml
│       └── health-check.yml
│
└── 2026/
    └── day-48/
        └── day-48-actions-project.md
```

---

# 3. Python Application

The Flask application provides the main application endpoint and a dedicated health endpoint.

```python
from flask import Flask, jsonify

app = Flask(__name__)


@app.route("/")
def home():
    return jsonify({
        "application": "GitHub Actions Capstone",
        "status": "running"
    })


@app.route("/health")
def health():
    return jsonify({
        "status": "healthy"
    })


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

# 4. Dependencies

`requirements.txt`

```text
Flask==3.1.2
pytest==8.4.2
```

Dependencies were installed locally and the test suite was executed successfully.

---

# 5. Automated Tests

`test_app.py`

```python
from app import app


def test_home():
    client = app.test_client()
    response = client.get("/")

    assert response.status_code == 200
    assert response.json["status"] == "running"


def test_health():
    client = app.test_client()
    response = client.get("/health")

    assert response.status_code == 200
    assert response.json["status"] == "healthy"
```

Tests were executed using:

```bash
pytest
```

Result:

```text
2 passed
```

---

# 6. Dockerization

The application was containerized using the following Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

Docker image build:

```bash
docker build -t github-actions-capstone:latest .
```

Container execution:

```bash
docker run -d --name github-actions-capstone -p 5000:5000 github-actions-capstone:latest
```

Application verification:

```bash
curl.exe http://localhost:5000/
```

Health verification:

```bash
curl.exe http://localhost:5000/health
```

---

# 7. Reusable Build and Test Workflow

File:

```text
.github/workflows/reusable-build-test.yml
```

The workflow uses:

```yaml
on:
  workflow_call:
```

Inputs:

* `python_version`
* `run_tests`

Output:

* `test_result`

The workflow:

1. Checks out the repository.
2. Sets up the requested Python version.
3. Installs dependencies.
4. Runs pytest.
5. Exposes the test result as a workflow output.

This workflow can be called by multiple pipelines instead of duplicating the same build/test logic.

---

# 8. Reusable Docker Workflow

File:

```text
.github/workflows/reusable-docker.yml
```

The workflow accepts:

* Docker image name
* Docker tag
* Docker Hub username secret
* Docker Hub token secret

It performs:

1. Repository checkout
2. Docker Hub authentication
3. Short Git SHA generation
4. Docker image build
5. Docker image push
6. Docker image URL output

Images are published using both:

```text
latest
```

and:

```text
sha-<short-git-sha>
```

Example:

```text
komalmankari26/github-actions-capstone:latest
komalmankari26/github-actions-capstone:sha-xxxxxxx
```

This provides both a stable deployment tag and a traceable immutable-style version tag.

---

# 9. Pull Request CI Pipeline

File:

```text
.github/workflows/pr-pipeline.yml
```

Trigger:

```yaml
on:
  pull_request:
    branches:
      - main
```

The workflow runs when a PR is opened or synchronized.

Pipeline:

```text
Pull Request
     ↓
Reusable Build & Test
     ↓
PR Validation
```

The PR pipeline does **not** build or push a Docker image.

This keeps Pull Request validation fast while preventing unmerged code from being published.

The validation step reports:

```text
PR checks passed for branch: <branch-name>
```

---

# 10. Main CI/CD Pipeline

File:

```text
.github/workflows/main-pipeline.yml
```

Trigger:

```yaml
on:
  push:
    branches:
      - main
```

Pipeline architecture:

```text
Push to main
     │
     ▼
Build & Test
     │
     ▼
Docker Build & Push
     │
     ▼
Production Deployment
```

Job dependency chain:

```yaml
needs: build-test
```

and:

```yaml
needs: docker
```

This ensures that:

* Docker publishing does not happen when tests fail.
* Deployment does not happen when Docker publishing fails.

---

# 11. Docker Hub Publishing

Docker Hub repository:

```text
komalmankari26/github-actions-capstone
```

Published tags:

```text
latest
sha-<short-sha>
```

Docker credentials are stored as GitHub Actions secrets:

```text
DOCKER_USERNAME
DOCKER_TOKEN
```

Secrets are referenced through:

```yaml
${{ secrets.DOCKER_USERNAME }}
${{ secrets.DOCKER_TOKEN }}
```

Credentials are never hard-coded into workflow files.

---

# 12. Production Deployment Job

The deployment job uses:

```yaml
environment: production
```

The workflow receives the Docker image URL from the reusable Docker workflow.

Deployment output:

```text
Deploying image: ...
Deployment completed successfully.
```

The production environment can also be configured with GitHub environment protection rules and manual approval requirements.

---

# 13. Container Health Check

File:

```text
.github/workflows/health-check.yml
```

Triggers:

### Scheduled

```yaml
cron: "0 */12 * * *"
```

This runs every 12 hours.

### Manual

```yaml
workflow_dispatch:
```

The workflow:

1. Pulls the latest Docker image.
2. Starts a container.
3. Waits for application startup.
4. Calls `/health`.
5. Fails if the health endpoint does not respond successfully.
6. Stops and removes the test container.
7. Writes a summary to `$GITHUB_STEP_SUMMARY`.

Health check command:

```bash
curl --fail http://localhost:5000/health
```

This provides an automated runtime validation layer in addition to the CI tests.

---

# 14. GitHub Actions Workflow Architecture

Overall architecture:

```text
                     ┌──────────────────┐
                     │   Pull Request   │
                     └────────┬─────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │ Reusable Build/Test   │
                  └───────────┬───────────┘
                              │
                              ▼
                       PR Validation
                             
                             
                     ┌──────────────────┐
                     │   Push to main   │
                     └────────┬─────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │ Reusable Build/Test   │
                  └───────────┬───────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │ Reusable Docker Build │
                  │       & Push          │
                  └───────────┬───────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │ Production Deployment │
                  └───────────────────────┘


             Scheduled / Manual Health Check
                              │
                              ▼
                    Pull Docker Image
                              │
                              ▼
                       Run Container
                              │
                              ▼
                       /health Check
                              │
                              ▼
                       Cleanup + Summary
```

---

# 15. Reusable Workflow Benefits

Reusable workflows provide:

* Less YAML duplication
* Standardized CI/CD processes
* Easier maintenance
* Consistent testing
* Centralized Docker publishing logic
* Better scalability across projects

Instead of repeating build and test steps in multiple workflows, the same workflow can be called using:

```yaml
uses: ./.github/workflows/reusable-build-test.yml
```

---

# 16. Workflow Outputs

The reusable workflows demonstrate communication between jobs and workflows.

Build/test output:

```text
test_result
```

Docker workflow output:

```text
image_url
```

The main pipeline consumes the Docker output:

```yaml
${{ needs.docker.outputs.image_url }}
```

This allows later stages to use information generated by earlier stages.

---

# 17. CI/CD Controls

The project implements several important CI/CD controls:

### Pull Request protection

Tests must pass before code reaches `main`.

### Job dependencies

Docker publishing waits for successful tests.

### Deployment dependency

Production deployment waits for successful Docker publishing.

### Secrets management

Docker credentials are stored in GitHub Secrets.

### Environment control

Production deployment uses a GitHub environment.

### Health monitoring

The published image is periodically tested through the `/health` endpoint.

---

# 18. Git Ignore Configuration

The project ignores generated Python files and local environments:

```gitignore
__pycache__/
*.py[cod]
.pytest_cache/
.venv/
venv/
.env
```

This keeps generated files and local configuration out of version control.

---

# 19. Validation Performed

### Local Python tests

```bash
pytest
```

Result:

```text
2 passed
```

### Docker build

```bash
docker build -t github-actions-capstone:latest .
```

### Local container

```bash
docker run -d --name github-actions-capstone -p 5000:5000 github-actions-capstone:latest
```

### Application endpoint

```bash
curl.exe http://localhost:5000/
```

### Health endpoint

```bash
curl.exe http://localhost:5000/health
```

### GitHub Actions

Validated:

* Pull Request CI
* Build and test
* Docker build
* Docker Hub push
* Short SHA tagging
* Production deployment job
* Scheduled/manual health check

---

# 20. Docker Image Versioning

Using only:

```text
latest
```

makes it difficult to identify exactly which source revision produced an image.

The pipeline therefore also publishes:

```text
sha-<short-git-sha>
```

Example:

```text
komalmankari26/github-actions-capstone:latest
komalmankari26/github-actions-capstone:sha-a1b2c3d
```

This improves traceability between:

```text
Git commit → GitHub Actions run → Docker image
```

---

# 21. Security Considerations

Implemented:

* Docker credentials stored in GitHub Secrets
* No credentials hard-coded in YAML
* Production environment separation
* PR validation before merge
* Automated test execution
* Docker image health validation

Future security improvements could include:

* Trivy vulnerability scanning
* Dependency vulnerability scanning
* Image signing
* SBOM generation
* Least-privilege GitHub token permissions
* Deployment to a managed cloud platform

Trivy scanning was kept as an optional enhancement rather than part of the core implementation.

---

# 22. Future Improvements

Possible production-level extensions:

* Deploy to AWS ECS/EKS
* Deploy to Azure Container Apps
* Deploy to Kubernetes
* Add staging environment
* Add production approval gates
* Add rollback strategy
* Add Slack/Teams notifications
* Add Docker image vulnerability scanning
* Add SBOM generation
* Add monitoring and observability
* Add automated integration tests
* Add infrastructure as code using Terraform

---

# 23. Key GitHub Actions Concepts Demonstrated

This project combines several GitHub Actions capabilities into one CI/CD system:

```text
workflow_call
workflow inputs
workflow secrets
workflow outputs
job dependencies
needs
pull_request
push
schedule
workflow_dispatch
GitHub environments
GitHub Secrets
Docker authentication
Docker build/push
GITHUB_OUTPUT
GITHUB_STEP_SUMMARY
```

---

# 24. Final CI/CD Flow

```text
Developer
    │
    ▼
Feature Branch
    │
    ▼
Pull Request
    │
    ▼
Build + Automated Tests
    │
    ├── FAIL → PR blocked
    │
    ▼
Merge to main
    │
    ▼
Build + Automated Tests
    │
    ├── FAIL → Pipeline stops
    │
    ▼
Build Docker Image
    │
    ▼
Push to Docker Hub
    │
    ├── latest
    └── sha-<short-sha>
    │
    ▼
Production Deployment
    │
    ▼
Scheduled / Manual Health Check
    │
    ▼
Application Health Verified
```

---

# 25. Project Outcome

The project demonstrates a complete GitHub Actions CI/CD implementation rather than isolated workflow exercises.

The final pipeline provides:

* Automated quality validation
* Reusable workflow architecture
* Secure Docker publishing
* Versioned container images
* Controlled production deployment
* Automated runtime health verification
* Clear separation between PR validation and production delivery

This creates a practical foundation for extending the pipeline toward cloud-native and production deployment platforms.
