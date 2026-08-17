# Day 31 – Dockerfile: Build My Own Images

## Goal

Today I learned how to create my own Docker images using a `Dockerfile`.

I practiced:

* Creating a Dockerfile
* Building a custom image
* Running a container from my image
* Understanding Dockerfile instructions
* CMD vs ENTRYPOINT
* Creating a simple Nginx website
* Using `.dockerignore`
* Understanding Docker build cache

---

# Task 1 – My First Dockerfile

## 1. Create the folder

I created my Day 31 folder:

```bash
cd ~/90DaysOfDevOps
mkdir -p 2026/day-31
cd 2026/day-31
mkdir my-first-image
cd my-first-image
```

## 2. Create Dockerfile

```bash
touch Dockerfile
```

My Dockerfile:

```dockerfile
FROM ubuntu

RUN apt-get update && apt-get install -y curl

CMD ["echo", "Hello from my custom image!"]
```

## What I learned

`FROM` → selects the base image.

`RUN` → runs commands while building the image.

`CMD` → gives the default command when the container starts.

---

## 3. Build the image

```bash
docker build -t my-ubuntu:v1 .
```

The `.` means Docker uses the current folder as the build context.

---

## 4. Check the image

```bash
docker images
```

I should see:

```text
my-ubuntu    v1
```

---

## 5. Run the container

```bash
docker run my-ubuntu:v1
```

Output:

```text
Hello from my custom image!
```

---

## 6. Check curl

I also verified that `curl` was installed:

```bash
docker run my-ubuntu:v1 curl --version
```

This confirmed that the `RUN` instruction installed curl inside my image.

---

# Task 2 – Dockerfile Instructions

I created another Dockerfile to practice the main Dockerfile instructions.

```dockerfile
FROM ubuntu

RUN apt-get update && apt-get install -y curl

WORKDIR /app

COPY hello.txt .

EXPOSE 8080

CMD ["cat", "hello.txt"]
```

I created `hello.txt`:

```text
Hello from the Docker build context!
```

## Build

```bash
docker build -t dockerfile-demo:v1 .
```

## Run

```bash
docker run dockerfile-demo:v1
```

Output:

```text
Hello from the Docker build context!
```

## Dockerfile instructions I learned

### FROM

```dockerfile
FROM ubuntu
```

Specifies the base image.

### RUN

```dockerfile
RUN apt-get update && apt-get install -y curl
```

Runs commands during the image build.

### COPY

```dockerfile
COPY hello.txt .
```

Copies a file from my computer into the image.

### WORKDIR

```dockerfile
WORKDIR /app
```

Sets the working directory inside the image/container.

### EXPOSE

```dockerfile
EXPOSE 8080
```

Documents the port that the application uses.

It does not publish the port automatically.

### CMD

```dockerfile
CMD ["cat", "hello.txt"]
```

Sets the default command when the container starts.

---

# Task 3 – CMD vs ENTRYPOINT

## CMD

Dockerfile:

```dockerfile
FROM ubuntu

CMD ["echo", "hello"]
```

Build:

```bash
docker build -f Dockerfile.cmd -t cmd-demo:v1 .
```

Run:

```bash
docker run cmd-demo:v1
```

Output:

```text
hello
```

If I run:

```bash
docker run cmd-demo:v1 echo custom-message
```

Output:

```text
custom-message
```

### My understanding

`CMD` provides a default command.

The default command can be replaced when I give another command with `docker run`.

---

## ENTRYPOINT

Dockerfile:

```dockerfile
FROM ubuntu

ENTRYPOINT ["echo"]
```

Build:

```bash
docker build -f Dockerfile.entrypoint -t entrypoint-demo:v1 .
```

Run:

```bash
docker run entrypoint-demo:v1 hello
```

Output:

```text
hello
```

If I run:

```bash
docker run entrypoint-demo:v1 "Hello Docker"
```

The argument is passed to the `echo` entrypoint.

### My understanding

`ENTRYPOINT` defines the main executable of the container.

The arguments supplied with `docker run` are passed to it.

---

## CMD vs ENTRYPOINT – My Notes

| CMD                          | ENTRYPOINT                                  |
| ---------------------------- | ------------------------------------------- |
| Provides a default command   | Defines the main executable                 |
| Can easily be replaced       | Arguments are normally passed to it         |
| Good for flexible containers | Good when the container has a fixed purpose |

---

# Task 4 – Simple Nginx Website

I created a folder:

```bash
mkdir my-website
cd my-website
```

## My `index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Docker Website</title>
</head>
<body>
    <h1>Hello from my Docker website!</h1>
    <p>Built using Dockerfile and Nginx.</p>
</body>
</html>
```

## My Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

I learned that Nginx serves website files from:

```text
/usr/share/nginx/html/
```

---

## Build the website image

```bash
docker build -t my-website:v1 .
```

Check:

```bash
docker images
```

---

## Run the website

```bash
docker run -d -p 8080:80 --name my-website-container my-website:v1
```

### Port mapping

```text
8080:80
```

Means:

```text
My computer port 8080
        ↓
Container port 80
```

I opened:

```text
http://localhost:8080
```

My Docker website was displayed in the browser.

---

## Check the container

```bash
docker ps
```

## Stop it

```bash
docker stop my-website-container
```

## Remove it

```bash
docker rm my-website-container
```

---

# Task 5 – .dockerignore

I created a `.dockerignore` file:

```text
node_modules
.git
*.md
.env
```

## Why I use `.dockerignore`

It tells Docker which files and folders should not be included in the build context.

### `node_modules`

Avoids copying Node.js dependencies.

### `.git`

Avoids copying Git information.

### `*.md`

Ignores Markdown files.

### `.env`

Helps prevent environment configuration files from being sent in the build context.

---

# Task 6 – Docker Build Cache

I learned that Docker builds images in layers.

When I build the same Dockerfile again, Docker can reuse unchanged layers.

Example:

```bash
docker build -t cache-demo:v1 .
```

Then I build again:

```bash
docker build -t cache-demo:v2 .
```

Docker can show:

```text
CACHED
```

for steps that have not changed.

---

## Testing the cache

If I change:

```text
hello.txt
```

and build again:

```bash
docker build -t cache-demo:v3 .
```

Docker can reuse previous unchanged layers and rebuild the affected part.

---

# Why Dockerfile Layer Order Matters

Dockerfile instructions create image layers.

I learned that frequently changing instructions should generally be placed later.

Example:

```dockerfile
FROM ubuntu

RUN apt-get update && apt-get install -y curl

WORKDIR /app

COPY hello.txt .

CMD ["cat", "hello.txt"]
```

If `hello.txt` changes, Docker can reuse the earlier layers.

This makes the build faster.

---

# Important Commands I Practiced

## Build an image

```bash
docker build -t name:tag .
```

## List images

```bash
docker images
```

## Run a container

```bash
docker run name:tag
```

## Run in background

```bash
docker run -d name:tag
```

## Run with port mapping

```bash
docker run -d -p 8080:80 name:tag
```

## List running containers

```bash
docker ps
```

## List all containers

```bash
docker ps -a
```

## Stop a container

```bash
docker stop container-name
```

## Remove a container

```bash
docker rm container-name
```

## Remove an image

```bash
docker rmi image-name:tag
```

## View container logs

```bash
docker logs container-name
```

## Enter a running container

```bash
docker exec -it container-name bash
```

For Alpine:

```bash
docker exec -it container-name sh
```

---

# What I Learned Today

Today I learned that a Dockerfile is a set of instructions used to build a Docker image.

The basic process is:

```text
Dockerfile
     ↓
docker build
     ↓
Docker Image
     ↓
docker run
     ↓
Container
```

I also learned:

* Docker images are made of layers.
* Docker can reuse cached layers.
* `FROM` selects the base image.
* `RUN` executes commands during build.
* `COPY` copies files into the image.
* `WORKDIR` sets the working directory.
* `EXPOSE` documents a port.
* `CMD` provides a default command.
* `ENTRYPOINT` defines the main executable.
* `.dockerignore` prevents unnecessary files from entering the build context.
* Proper layer ordering can make Docker builds faster.
* I can create and run my own Docker images.

---

# Day 31 Project Structure

My Day 31 folder contains:

```text
2026/
└── day-31/
    ├── my-first-image/
    │   └── Dockerfile
    │
    ├── dockerfile-instructions/
    │   ├── Dockerfile
    │   └── hello.txt
    │
    ├── cmd-entrypoint/
    │   ├── Dockerfile.cmd
    │   └── Dockerfile.entrypoint
    │
    ├── my-website/
    │   ├── Dockerfile
    │   ├── index.html
    │   └── .dockerignore
    │
    └── day-31-dockerfile.md
```

---

# Git – Save My Work

From the repository root:

```bash
cd ~/90DaysOfDevOps
```

Check my changes:

```bash
git status
```

Add Day 31:

```bash
git add 2026/day-31/
```

Commit:

```bash
git commit -m "Complete Day 31 Dockerfile practice"
```

Push:

```bash
git push
```

---

# Day 31 Completed ✅

I built my first custom Docker image and learned how Dockerfiles are used to turn application files and instructions into Docker images.

**My key learning:**

> Dockerfile → Build → Image → Run → Container
