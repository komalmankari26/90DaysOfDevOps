# Day 35 – Multi-Stage Builds & Docker Hub

## Objective

Today's goal was to:

* Understand the problem with large Docker images
* Build a Go application using Docker
* Create a single-stage Docker image
* Create a multi-stage Docker image
* Compare image sizes
* Use a minimal Alpine runtime image
* Run the application as a non-root user
* Push an image to Docker Hub
* Pull the image back from Docker Hub
* Understand Docker image tags such as `v1` and `latest`

---

# Project Structure

```text
2026/day-35/
└── go-multistage-demo/
    ├── main.go
    ├── Dockerfile.single
    ├── Dockerfile.multistage
    └── Dockerfile.best
```

---

# Task 1 – The Problem with Large Images

## Step 1 – Create the Go application

Created `main.go`.

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello from my Day 35 Docker Multi-Stage Build!")
}
```

### What is Go?

Go is a programming language developed by Google.

A Go source-code file normally has the `.go` extension.

For this exercise:

```text
main.go
```

is our simple Go application.

I did not need Go installed locally because Docker provides the Go environment through the Docker image.

---

# Task 2 – Single-Stage Docker Build

Created:

```text
Dockerfile.single
```

Contents:

```dockerfile
FROM golang:1.25

WORKDIR /app

COPY main.go .

RUN go build -o app main.go

CMD ["./app"]
```

## Build the image

```bash
docker build -f Dockerfile.single -t day35-single:v1 .
```

## Check image size

```bash
docker images day35-single
```

### Single-stage image size

```text
________________
```

## Run the image

```bash
docker run --rm day35-single:v1
```

Expected output:

```text
Hello from my Day 35 Docker Multi-Stage Build!
```

---

# Why is the Single-Stage Image Large?

The single-stage image contains the Go build environment.

It contains things such as:

* Go compiler
* Go tools
* Build environment
* Application

The compiler is needed to build the application, but it is not required to run the final application.

Therefore, keeping the complete Go environment in the final image makes the image unnecessarily large.

---

# Task 3 – Multi-Stage Build

Created:

```text
Dockerfile.multistage
```

Contents:

```dockerfile
# Stage 1: Build
FROM golang:1.25-alpine AS builder

WORKDIR /app

COPY main.go .

RUN go build -o app main.go

# Stage 2: Run
FROM alpine:3.22

WORKDIR /app

COPY --from=builder /app/app .

CMD ["./app"]
```

## Build the multi-stage image

```bash
docker build -f Dockerfile.multistage -t day35-multi:v1 .
```

## Check image size

```bash
docker images day35-multi
```

My multi-stage image size was approximately:

```text
16.4 MB
```

## Run the image

```bash
docker run --rm day35-multi:v1
```

Expected output:

```text
Hello from my Day 35 Docker Multi-Stage Build!
```

---

# How Multi-Stage Builds Work

The Dockerfile contains two stages.

## Stage 1 – Builder

```dockerfile
FROM golang:1.25-alpine AS builder
```

This stage contains the Go compiler and is used to build the application.

```dockerfile
RUN go build -o app main.go
```

This creates the compiled application.

## Stage 2 – Runtime

```dockerfile
FROM alpine:3.22
```

A small Alpine image is used for running the application.

Only the compiled application is copied:

```dockerfile
COPY --from=builder /app/app .
```

The Go compiler and build environment are not copied into the final image.

---

# Why is the Multi-Stage Image Smaller?

The final image contains only the files required to run the application.

The build tools, compiler, and unnecessary development environment remain in the builder stage.

Therefore:

```text
Single-stage
Go compiler + tools + application
        ↓
Large image

Multi-stage
Compiled application + minimal runtime
        ↓
Small image
```

---

# Task 4 – Docker Hub

Docker Hub repository:

```text
https://hub.docker.com/
```

My Docker Hub username:

```text
komalmankari26
```

Repository:

```text
komalmankari26/day35-multi
```

---

# Docker Hub Login

Logged into Docker Hub using:

```bash
docker login
```

---

# Tagging the Image

Tagged the local image for Docker Hub:

```bash
docker tag day35-multi:v1 komalmankari26/day35-multi:v1
```

## Check the tag

```bash
docker images
```

---

# Push Image to Docker Hub

```bash
docker push komalmankari26/day35-multi:v1
```

Push completed successfully.

Docker returned a digest similar to:

```text
sha256:39dad57bac2442b5e9c2c412000ff439fecb7e8185b3d82e1aa858000ccd3b6c
```

---

# Pull the Image from Docker Hub

To verify that the image can be distributed and downloaded:

```bash
docker pull komalmankari26/day35-multi:v1
```

Then run:

```bash
docker run --rm komalmankari26/day35-multi:v1
```

Expected output:

```text
Hello from my Day 35 Docker Multi-Stage Build!
```

This verified that the image stored on Docker Hub can be pulled and executed.

---

# Docker Hub Repository Description

Added the following repository description:

```text
Day 35 Docker multi-stage build practice using Go and Alpine Linux. The application is built in a builder stage and deployed using a minimal runtime image.
```

---

# Docker Image Tags

Created the `latest` tag:

```bash
docker tag day35-multi:v1 komalmankari26/day35-multi:latest
```

Push:

```bash
docker push komalmankari26/day35-multi:latest
```

Check tags:

```bash
docker images komalmankari26/day35-multi
```

The repository contains:

```text
v1
latest
```

---

# Understanding `v1` vs `latest`

Specific version:

```bash
docker pull komalmankari26/day35-multi:v1
```

This requests version `v1`.

Latest tag:

```bash
docker pull komalmankari26/day35-multi:latest
```

This requests the image currently assigned the `latest` tag.

Important:

`latest` does not automatically mean the newest image.

It is simply a tag named `latest`.

---

# Task 5 – Docker Image Best Practices

Created:

```text
Dockerfile.best
```

Contents:

```dockerfile
# Stage 1: Build
FROM golang:1.25-alpine AS builder

WORKDIR /app

COPY main.go .

RUN go build -o app main.go

# Stage 2: Minimal runtime
FROM alpine:3.22

WORKDIR /app

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY --from=builder /app/app .

USER appuser

CMD ["./app"]
```

---

# Best Practice 1 – Multi-Stage Build

Used:

```dockerfile
FROM golang:1.25-alpine AS builder
```

for building.

Then:

```dockerfile
FROM alpine:3.22
```

for running.

Only the compiled application is copied into the final image.

---

# Best Practice 2 – Minimal Base Image

Used:

```dockerfile
FROM alpine:3.22
```

instead of a much larger general-purpose image such as Ubuntu.

Alpine is useful for small runtime images.

---

# Best Practice 3 – Specific Image Tags

Instead of:

```dockerfile
FROM alpine
```

used:

```dockerfile
FROM alpine:3.22
```

Using a specific version makes builds more predictable.

---

# Best Practice 4 – Don't Run as Root

Created a non-root user:

```dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
```

Then:

```dockerfile
USER appuser
```

This means the application runs as `appuser` instead of root.

## Verify

```bash
docker run --rm day35-best:v1 id
```

The output should show:

```text
appuser
```

rather than:

```text
root
```

---

# Best Practice 5 – Reduce Layers

Used a combined command:

```dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
```

Combining related commands can reduce unnecessary image layers.

---

# Build the Best-Practice Image

```bash
docker build -f Dockerfile.best -t day35-best:v1 .
```

Check the size:

```bash
docker images day35-best
```

My best-practice image size:

```text
________________
```

Run:

```bash
docker run --rm day35-best:v1
```

Expected:

```text
Hello from my Day 35 Docker Multi-Stage Build!
```

Verify user:

```bash
docker run --rm day35-best:v1 id
```

---

# Alpine vs Ubuntu Comparison

Pull Alpine:

```bash
docker pull alpine:3.22
```

Check:

```bash
docker images alpine:3.22
```

Pull Ubuntu:

```bash
docker pull ubuntu:24.04
```

Check:

```bash
docker images ubuntu:24.04
```

Compare:

```bash
docker images alpine:3.22 ubuntu:24.04
```

Record the actual sizes:

```text
Alpine 3.22: __________

Ubuntu 24.04: __________
```

Alpine is generally much smaller than Ubuntu, which makes it useful when a minimal runtime environment is sufficient.

---

# Final Image Comparison

```bash
docker images day35-single day35-multi day35-best
```

| Image             | Purpose                      |    Size |
| ----------------- | ---------------------------- | ------: |
| `day35-single:v1` | Single-stage build           |  ______ |
| `day35-multi:v1`  | Multi-stage build            | 16.4 MB |
| `day35-best:v1`   | Multi-stage + best practices |  ______ |

---

# Important Docker Commands Practised

## Build

```bash
docker build -f Dockerfile.single -t day35-single:v1 .
```

```bash
docker build -f Dockerfile.multistage -t day35-multi:v1 .
```

```bash
docker build -f Dockerfile.best -t day35-best:v1 .
```

## Run

```bash
docker run --rm day35-single:v1
```

```bash
docker run --rm day35-multi:v1
```

```bash
docker run --rm day35-best:v1
```

## Check images

```bash
docker images
```

```bash
docker images day35-multi
```

## Inspect layers

```bash
docker history day35-single:v1
```

```bash
docker history day35-multi:v1
```

## Tag

```bash
docker tag day35-multi:v1 komalmankari26/day35-multi:v1
```

```bash
docker tag day35-multi:v1 komalmankari26/day35-multi:latest
```

## Push

```bash
docker push komalmankari26/day35-multi:v1
```

```bash
docker push komalmankari26/day35-multi:latest
```

## Pull

```bash
docker pull komalmankari26/day35-multi:v1
```

```bash
docker pull komalmankari26/day35-multi:latest
```

---

# Key Learnings

### 1. Single-stage builds

Build and runtime are in the same image.

```text
Build tools + application
        ↓
Large image
```

### 2. Multi-stage builds

Build and runtime are separated.

```text
Builder
   ↓
Compiled application
   ↓
Minimal runtime image
```

### 3. Multi-stage builds reduce image size

Only the required application artifact is copied into the final image.

### 4. Minimal base images

Alpine can provide a much smaller runtime environment than a general-purpose Ubuntu image.

### 5. Non-root containers

Using:

```dockerfile
USER appuser
```

reduces the security risk of running the application as root.

### 6. Image tags

Examples:

```text
v1
v2
latest
```

Tags allow different versions of an image to be identified and distributed.

### 7. Docker Hub

Docker Hub can be used to store and distribute Docker images.

---

# Interview Questions

## Q1. What is a multi-stage Docker build?

A multi-stage Docker build uses multiple `FROM` instructions. One stage builds the application, while another stage contains only what is required to run it.

## Q2. Why are multi-stage builds useful?

They reduce image size and keep build tools out of the production image.

## Q3. What does `COPY --from=builder` do?

It copies files from the builder stage into the final stage.

## Q4. Why use Alpine?

Alpine is a small Linux distribution that can be useful for creating lightweight container images.

## Q5. Why should containers avoid running as root?

Running as a non-root user reduces the potential impact if the application is compromised.

## Q6. What is a Docker tag?

A tag is a label used to identify a particular image version.

Example:

```text
day35-multi:v1
```

## Q7. Is `latest` always the newest image?

No. `latest` is only a tag name. It does not automatically guarantee that the image is the newest version.

## Q8. What is Docker Hub?

Docker Hub is a registry where Docker images can be stored, shared, and pulled.

---

# Docker Hub Repository

Repository:

```text
komalmankari26/day35-multi
```

Docker Hub:

```text
https://hub.docker.com/
```

---

# Day 35 Completion Checklist

* [x] Created a simple Go application
* [x] Created a single-stage Dockerfile
* [x] Built the single-stage image
* [x] Checked the single-stage image size
* [x] Created a multi-stage Dockerfile
* [x] Built the multi-stage image
* [x] Checked the multi-stage image size
* [x] Compared image sizes
* [x] Tested the multi-stage container
* [x] Logged into Docker Hub
* [x] Created Docker Hub repository
* [x] Added repository description
* [x] Tagged image as `v1`
* [x] Pushed `v1` to Docker Hub
* [x] Pulled `v1` from Docker Hub
* [x] Ran the pulled image
* [x] Created `latest` tag
* [x] Pushed `latest`
* [x] Used Alpine as a minimal runtime
* [x] Used specific base image versions
* [x] Created a non-root user
* [x] Tested the non-root user
* [x] Compared Alpine and Ubuntu
* [x] Practised Docker image layers

---

# Final Summary

Day 35 demonstrated how to build smaller and more secure Docker images using multi-stage builds.

The main concept was:

```text
Large Build Environment
        ↓
      Build
        ↓
Compiled Application
        ↓
Minimal Runtime Image
        ↓
Smaller + More Efficient Image
```

The final image was published to Docker Hub as:

```text
komalmankari26/day35-multi:v1
```

and:

```text
komalmankari26/day35-multi:latest
```

Main lesson:

> Build with everything you need, but ship only what you need to run.
