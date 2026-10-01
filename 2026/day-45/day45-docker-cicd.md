# Day 45 – Docker Build & Push in GitHub Actions

## Task

Built a complete CI/CD pipeline using GitHub Actions and Docker.

The pipeline automatically:

1. Builds the Docker image.
2. Logs in to Docker Hub using GitHub Secrets.
3. Tags the image with `latest` and a short commit SHA.
4. Pushes the image to Docker Hub only from the `main` branch.
5. Provides a GitHub Actions status badge.
6. Allows the image to be pulled and run locally.

---

## Docker Hub Image

Docker Hub Repository:

https://hub.docker.com/r/komalmankari26/day45-python-app

Image:

```text
komalmankari26/day45-python-app
```

Tags used:

```text
latest
sha-<short-commit-hash>
```

---

## GitHub Actions Workflow

File:

```text
.github/workflows/docker-publish.yml
```

The workflow builds the Docker image and pushes it to Docker Hub.

```yaml
name: Docker Publish

on:
  push:
    branches:
      - main

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set short SHA
        id: short-sha
        run: echo "sha_short=$(git rev-parse --short HEAD)" >> "$GITHUB_OUTPUT"

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            komalmankari26/day45-python-app:latest
            komalmankari26/day45-python-app:sha-${{ steps.short-sha.outputs.sha_short }}
```

---

## GitHub Secrets

The Docker Hub credentials were stored securely in GitHub repository secrets.

```text
DOCKER_USERNAME
DOCKER_TOKEN
```

The Docker Hub token was not hard-coded in the workflow.

---

## Docker Image Verification

The image was successfully pulled from Docker Hub using:

```bash
docker pull komalmankari26/day45-python-app:latest
```

Result:

```text
Status: Downloaded newer image for komalmankari26/day45-python-app:latest
```

---

## Run the Container

The Docker image was then executed locally using:

```bash
docker run --rm komalmankari26/day45-python-app:latest
```

The application ran successfully and all tests passed.

---

## Status Badge

The repository README contains the Docker Publish GitHub Actions status badge:

```markdown
[![Docker Publish](https://github.com/komalmankari26/git-action-practice/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/komalmankari26/git-action-practice/actions/workflows/docker-publish.yml)
```

The badge shows the status of the Docker Publish workflow.

---

## Full CI/CD Journey

The complete journey is:

```text
Developer changes code
        ↓
git add
        ↓
git commit
        ↓
git push
        ↓
GitHub receives the push
        ↓
GitHub Actions workflow starts
        ↓
Checkout repository
        ↓
Generate short commit SHA
        ↓
Login to Docker Hub using GitHub Secrets
        ↓
Build Docker image
        ↓
Tag image as latest
        ↓
Tag image with short commit SHA
        ↓
Push image to Docker Hub
        ↓
Docker image becomes available on Docker Hub
        ↓
docker pull
        ↓
docker run
        ↓
Application runs successfully
```

---

## What I Learned

### GitHub Actions

* How to automate Docker builds with GitHub Actions.
* How workflows can be triggered by pushes to `main`.
* How GitHub Secrets can securely store Docker Hub credentials.
* How to use GitHub Actions Docker login and build/push actions.

### Docker

* How to build Docker images in CI.
* How to apply multiple image tags.
* How to push images to Docker Hub.
* How to pull and run a Docker image locally.

### CI/CD

This exercise demonstrated an end-to-end CI/CD workflow where a code push can automatically produce a deployable Docker image.

---

## Final Verification

* [x] Dockerfile available in repository
* [x] GitHub Secrets configured
* [x] Docker image built successfully
* [x] Docker image pushed to Docker Hub
* [x] `latest` tag created
* [x] Short SHA tag created
* [x] Push restricted to `main`
* [x] README status badge added
* [x] Docker image pulled successfully
* [x] Docker container ran successfully
* [x] All tests passed

---

## Key Takeaway

GitHub Actions can automate the complete journey from source-code change to a Docker image available on Docker Hub.

This removes repetitive manual Docker build and push steps and provides a practical foundation for production CI/CD pipelines.
