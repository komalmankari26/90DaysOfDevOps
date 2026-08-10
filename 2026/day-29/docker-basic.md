# Day 29 – Introduction to Docker

## Task

Today's goal is to **understand what Docker is and run your first container**.

You will:

- Learn why containers exist and how they differ from VMs
- Install Docker on your machine
- Run and explore containers from Docker Hub
- Use Docker Desktop
- Practice Docker commands using Git Bash

---

# Expected Output

- A markdown file: `day-29-docker-basics.md`
- Screenshots of running containers
- Docker Desktop working successfully
- Hello World container running successfully
- Nginx container running successfully
- Ubuntu container explored interactively

---

# Task 1: What is Docker?

## 1. What is Docker?

Docker is a platform used to **build, package, and run applications inside containers**.

Docker packages an application together with its dependencies so that it can run consistently across different environments.

### Why Do We Need Containers?

Without containers, an application may work on one machine but fail on another because of differences in:

- Operating system
- Libraries
- Dependencies
- Software versions
- Configuration

Docker solves this problem by packaging the application and its dependencies together.

### Benefits of Docker

- Lightweight
- Fast startup
- Portable
- Isolated
- Consistent
- Easy to deploy
- Efficient resource usage

---

## 2. What is a Container?

A container is an isolated environment used to run an application.

A container can contain:

- Application
- Dependencies
- Libraries
- Configuration

A container is created from a Docker image.

```text
Docker Image
     |
     | docker run
     ↓
Docker Container
