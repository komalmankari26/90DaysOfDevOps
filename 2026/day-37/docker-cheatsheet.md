# Day 37 – Docker Practical Notes

## 90DaysOfDevOps

---

# 🎯 Day 37 Goal

The goal of Day 37 is to strengthen Docker fundamentals by understanding **how Docker works internally and how to build, run, inspect, troubleshoot, and manage containers independently**.

The focus is not on memorizing commands.

The focus is:

> **Understand the requirement → choose the Docker concept → use the command → verify the result.**

---

# 1. Docker Mental Model

Before using commands, understand the basic flow:

```text
Application Code
       ↓
   Dockerfile
       ↓
   Docker Build
       ↓
 Docker Image
       ↓
 Docker Run
       ↓
 Docker Container
```

If the application needs a database:

```text
                 Docker Network
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
       Application            Database
        Container             Container
             │                   │
             └─────────┬─────────┘
                       ↓
                 Docker Volume
                  (persistent data)
```

---

# 2. What Problem Does Docker Solve?

Without Docker, developers may need to install:

* Python
* Node.js
* Java
* Databases
* Libraries
* Correct versions
* System dependencies

This can lead to:

> "It works on my machine."

Docker packages the application and its required environment into a portable image.

The same image can then run on:

* Developer laptop
* Test environment
* CI/CD
* Cloud server
* Production

---

# 3. Docker Image vs Container

## Docker Image

An image is a packaged, read-only blueprint of an application.

Example:

```text
python:3.11-slim
mysql:8.0
nginx:latest
```

Custom application image:

```text
day36-task-manager:v1
```

Think:

```text
IMAGE = BLUEPRINT
```

---

## Docker Container

A container is a running instance of an image.

Example:

```text
day36-web
day36-mysql
```

Think:

```text
CONTAINER = RUNNING INSTANCE
```

The relationship:

```text
Dockerfile
    ↓
Image
    ↓
Container
```

---

# 4. Dockerfile

A Dockerfile contains instructions for building an image.

Typical flow:

```text
Choose base image
       ↓
Install dependencies
       ↓
Copy application
       ↓
Configure environment
       ↓
Set startup command
```

Example:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app ./app

EXPOSE 5000

CMD ["python", "app/app.py"]
```

---

# 5. Dockerfile Instructions

## FROM

```dockerfile
FROM python:3.11-slim
```

Meaning:

> Start with the Python 3.11 slim image.

Think:

```text
I need Python
       ↓
Start with Python image
```

---

## WORKDIR

```dockerfile
WORKDIR /app
```

Sets the working directory inside the image/container.

Similar idea to:

```bash
cd /app
```

---

## COPY

```dockerfile
COPY requirements.txt .
```

Copies a file from the build context into the image.

Example:

```text
Local computer
requirements.txt
       ↓
Container image
/app/requirements.txt
```

---

## RUN

```dockerfile
RUN pip install -r requirements.txt
```

Executes a command while building the image.

Important:

```text
RUN = BUILD TIME
```

---

## CMD

```dockerfile
CMD ["python", "app/app.py"]
```

Defines the default command when the container starts.

Important:

```text
CMD = CONTAINER START TIME
```

Remember:

```text
RUN → during image build
CMD → when container starts
```

---

## EXPOSE

```dockerfile
EXPOSE 5000
```

Documents the port the application expects to use inside the container.

It does NOT publish the port to the host.

Actual port mapping is done with:

```bash
docker run -p 5050:5000 ...
```

---

# 6. Build an Image

Command:

```bash
docker build -t myapp:v1 .
```

Break it down:

```text
docker build
```

Build an image.

```text
-t myapp:v1
```

Give it a name and tag.

```text
.
```

Use the current directory as the build context.

Flow:

```text
Dockerfile
    +
Application files
    ↓
docker build
    ↓
myapp:v1
```

---

# 7. List Docker Images

```bash
docker images
```

or:

```bash
docker image ls
```

Useful information:

```text
REPOSITORY
TAG
IMAGE ID
CREATED
SIZE
```

---

# 8. Image Tags

Example:

```text
myapp:v1
```

Here:

```text
myapp = repository/name
v1    = tag
```

Another example:

```text
komalmankari26/day36-task-manager:v1
```

Breakdown:

```text
Docker Hub username
        ↓
komalmankari26

Repository
        ↓
day36-task-manager

Tag
        ↓
v1
```

---

# 9. Run a Container

Basic command:

```bash
docker run myapp:v1
```

Run in background:

```bash
docker run -d myapp:v1
```

`-d` means:

```text
detached mode
```

The container runs in the background.

---

# 10. Container Name

```bash
docker run -d --name myapp myapp:v1
```

Here:

```text
--name myapp
```

gives the container a friendly name.

Without a name, Docker generates one automatically.

---

# 11. Port Mapping

Suppose the application listens inside the container on port 5000.

Run:

```bash
docker run -d -p 5050:5000 myapp:v1
```

Read it as:

```text
HOST : CONTAINER
```

Therefore:

```text
5050 : 5000
```

Flow:

```text
Browser
   ↓
localhost:5050
   ↓
Docker
   ↓
Container:5000
   ↓
Application
```

---

# 12. Why Port Mapping Is Needed

A container has its own network environment.

The application may listen on:

```text
5000
```

inside the container.

Your computer needs a way to reach it.

Port mapping creates that connection:

```text
Host 5050
    ↓
Container 5000
```

---

# 13. Check Running Containers

```bash
docker ps
```

Shows running containers.

Useful information:

* Container ID
* Image
* Command
* Status
* Ports
* Name

---

# 14. Check All Containers

```bash
docker ps -a
```

This includes:

* Running containers
* Stopped containers
* Exited containers

Important distinction:

```text
docker ps
    ↓
running only

docker ps -a
    ↓
all containers
```

---

# 15. Container Logs

```bash
docker logs myapp
```

Follow logs continuously:

```bash
docker logs -f myapp
```

This is one of the first commands to use when an application is not working.

Mental model:

```text
Application not working?
        ↓
Check logs
        ↓
docker logs <container>
```

---

# 16. Execute a Command Inside a Container

```bash
docker exec -it myapp sh
```

Meaning:

```text
docker exec
    ↓
Execute command inside container

-it
    ↓
Interactive terminal

sh
    ↓
Open shell
```

Once inside:

```bash
ls
```

```bash
pwd
```

```bash
env
```

```bash
exit
```

---

# 17. Inspect a Container

```bash
docker inspect myapp
```

This provides detailed JSON information about:

* Network
* Mounts
* Environment
* IP address
* Configuration
* Container state

For example:

```bash
docker inspect myapp --format='{{json .Mounts}}'
```

This is useful when troubleshooting volumes.

---

# 18. Stop a Container

```bash
docker stop myapp
```

Stops a running container gracefully.

---

# 19. Start a Stopped Container

```bash
docker start myapp
```

Starts an existing stopped container.

Important:

```text
docker start
```

does not create a new container.

It starts an existing one.

---

# 20. Restart a Container

```bash
docker restart myapp
```

Equivalent conceptually to:

```text
stop
  ↓
start
```

---

# 21. Remove a Container

```bash
docker rm myapp
```

The container must normally be stopped first.

Force removal:

```bash
docker rm -f myapp
```

Use force carefully.

---

# 22. Stop and Remove Container

Common workflow:

```bash
docker stop myapp
docker rm myapp
```

Or:

```bash
docker rm -f myapp
```

---

# 23. Docker Volumes

Containers are not ideal places for important persistent data.

Databases especially need persistent storage.

Example:

```text
MySQL container
      ↓
/var/lib/mysql
      ↓
Docker volume
```

Create a volume:

```bash
docker volume create mydata
```

List volumes:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect mydata
```

---

# 24. Mount a Volume

Example:

```bash
docker run -d \
  --name mysql \
  -v mydata:/var/lib/mysql \
  mysql:8.0
```

Read:

```text
mydata
  :
/var/lib/mysql
```

The left side is the Docker volume.

The right side is the path inside the container.

---

# 25. Container vs Volume

Remember:

```text
Container = application environment
Volume    = persistent data
```

Example:

```text
Delete container
      ↓
Volume can remain
      ↓
Data can remain
```

This is why databases commonly use volumes.

---

# 26. Docker Networks

Containers need networks to communicate.

List networks:

```bash
docker network ls
```

Create a network:

```bash
docker network create mynetwork
```

Run a container on the network:

```bash
docker run -d \
  --name app \
  --network mynetwork \
  myapp:v1
```

---

# 27. Container-to-Container Communication

Suppose:

```text
app
db
```

are on the same Docker network.

The application should connect to:

```text
db
```

rather than:

```text
localhost
```

Why?

Inside the app container:

```text
localhost
```

means:

> The app container itself.

While:

```text
db
```

means:

> The database service/container on the Docker network.

---

# 28. Docker Compose

Docker Compose manages multiple related containers as one application.

Example:

```yaml
services:

  web:
    image: myapp:v1

  db:
    image: mysql:8.0
```

Instead of manually starting both containers, Compose manages them together.

---

# 29. Compose Mental Model

Think:

```text
docker-compose.yml
        ↓
Application definition
        ↓
Services
        ↓
Networks
        ↓
Volumes
        ↓
Containers
```

---

# 30. Compose Commands

Start:

```bash
docker compose up -d
```

Stop/remove containers:

```bash
docker compose down
```

View services:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

Build:

```bash
docker compose build
```

Pull images:

```bash
docker compose pull
```

---

# 31. `build` vs `image`

This is extremely important.

## Build

```yaml
build: .
```

Means:

> Build the image from the Dockerfile in this directory.

Flow:

```text
Dockerfile
   ↓
Build
   ↓
Local image
```

## Image

```yaml
image: username/myapp:v1
```

Means:

> Use this existing image.

Flow:

```text
Docker Hub
   ↓
Pull image
   ↓
Container
```

---

# 32. Docker Hub Workflow

Typical workflow:

```text
Dockerfile
     ↓
docker build
     ↓
Local Image
     ↓
docker tag
     ↓
Docker Hub name
     ↓
docker push
     ↓
Docker Hub
```

Pull workflow:

```text
Docker Hub
     ↓
docker pull
     ↓
Local Image
     ↓
docker run
     ↓
Container
```

---

# 33. Docker Tag

Example:

```bash
docker tag myapp:v1 username/myapp:v1
```

This does not create a completely separate application image.

It creates another reference/tag for the same image.

Then:

```bash
docker push username/myapp:v1
```

publishes it.

---

# 34. Docker Push

```bash
docker push username/myapp:v1
```

Uploads the image to Docker Hub.

Before pushing:

```bash
docker login
```

---

# 35. Docker Pull

```bash
docker pull username/myapp:v1
```

Downloads the image from Docker Hub.

Useful for testing whether the published image can actually be deployed.

---

# 36. `.env`

Environment variables keep configuration separate from application code.

Example:

```env
DB_HOST=db
DB_NAME=taskdb
DB_USER=taskuser
DB_PASSWORD=taskpassword
```

Compose can load them using:

```yaml
env_file:
  - .env
```

Important:

Do not commit real passwords or secrets to GitHub.

Add:

```text
.env
```

to `.gitignore`.

---

# 37. Healthchecks

A service may be running but not actually ready.

Example:

```text
MySQL container started
       ↓
MySQL initialization
       ↓
Database becomes ready
```

A healthcheck tests whether the service is actually ready.

Example:

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

---

# 38. `depends_on`

Example:

```yaml
depends_on:
  db:
    condition: service_healthy
```

Means:

> Start the web service after the database becomes healthy.

Flow:

```text
Start DB
   ↓
Healthcheck
   ↓
Healthy
   ↓
Start Web
```

---

# 39. Docker Compose Architecture

For a Flask + MySQL application:

```text
                 User
                   |
                   ↓
             localhost:5050
                   |
                   ↓
          ┌─────────────────┐
          │ Flask Container │
          │      web        │
          └────────┬────────┘
                   |
                   | Docker Network
                   ↓
          ┌─────────────────┐
          │ MySQL Container │
          │       db        │
          └────────┬────────┘
                   |
                   ↓
             Docker Volume
```

---

# 40. Troubleshooting Method

Do not randomly execute commands when something fails.

Use this sequence.

## Step 1 – Check containers

```bash
docker compose ps
```

Ask:

```text
Is the container running?
Is it healthy?
Are ports mapped?
```

---

## Step 2 – Check logs

```bash
docker compose logs
```

Or:

```bash
docker compose logs web
```

Or:

```bash
docker compose logs db
```

---

## Step 3 – Inspect

```bash
docker inspect <container>
```

Check:

* Network
* Mounts
* Environment
* State

---

## Step 4 – Check network

```bash
docker network ls
```

Then:

```bash
docker network inspect <network>
```

Ask:

```text
Are both containers on the same network?
```

---

## Step 5 – Check volume

```bash
docker volume ls
```

Then:

```bash
docker volume inspect <volume>
```

Ask:

```text
Is the database actually using the volume?
```

---

## Step 6 – Test from inside the container

```bash
docker exec -it <container> sh
```

Then test:

```bash
env
```

```bash
ls
```

```bash
```

---

# 41. Common Docker Errors

## Error: Port already allocated

Example:

```text
Bind for 0.0.0.0:5000 failed:
port is already allocated
```

Meaning:

> Another process/container is already using that host port.

Check:

```bash
docker ps
```

Choose another host port:

```yaml
ports:
  - "5050:5000"
```

Remember:

```text
Host port can change.
Container port can remain the same.
```

---

# 42. Error: Container Name Already in Use

Example:

```text
Conflict. The container name is already in use
```

Check:

```bash
docker ps -a
```

Remove the old container:

```bash
docker rm <container>
```

Then retry.

---

# 43. Error: Volume Is in Use

Example:

```text
volume is in use
```

Meaning:

> A container is still attached to that volume.

Check:

```bash
docker ps -a
```

Stop/remove the container first.

Do not delete the volume if you need its data.

---

# 44. Error: 404 Not Found

A 404 from Flask does not automatically mean Docker is broken.

Example:

```text
GET /db
404 Not Found
```

Possible reason:

> The Flask application does not define `/db`.

Always distinguish:

```text
Docker problem
vs
Application/API problem
```

---

# 45. Docker Debugging Mindset

When an application doesn't work, think in layers:

```text
Layer 1
Is the container running?
       ↓
Layer 2
Is the application running?
       ↓
Layer 3
Is the port exposed/mapped?
       ↓
Layer 4
Can containers communicate?
       ↓
Layer 5
Is the database running?
       ↓
Layer 6
Is the database healthy?
       ↓
Layer 7
Is persistent storage mounted?
       ↓
Layer 8
Are environment variables correct?
```

This prevents random troubleshooting.

---

# 46. Important Commands Cheat Sheet

## Images

```bash
docker images
docker image ls
docker image inspect <image>
docker rmi <image>
```

## Containers

```bash
docker ps
docker ps -a
docker run
docker stop
docker start
docker restart
docker rm
docker logs
docker exec
docker inspect
```

## Volumes

```bash
docker volume ls
docker volume create <name>
docker volume inspect <name>
docker volume rm <name>
```

## Networks

```bash
docker network ls
docker network create <name>
docker network inspect <name>
docker network rm <name>
```

## Compose

```bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs
docker compose build
docker compose pull
```

## Docker Hub

```bash
docker login
docker tag
docker push
docker pull
```

---

# 47. Command Memory Technique

Do not memorize commands individually.

Group them by purpose.

### See

```text
docker ps
docker images
docker volume ls
docker network ls
```

### Start

```text
docker start
docker compose up
```

### Stop

```text
docker stop
docker compose down
```

### Investigate

```text
docker logs
docker inspect
docker exec
```

### Create

```text
docker build
docker run
docker volume create
docker network create
```

### Share

```text
docker tag
docker push
```

### Download

```text
docker pull
docker compose pull
```

This makes commands much easier to remember.

---

# 48. Dockerization Decision Process

When given a new project, ask these questions:

## Question 1

What is the application?

Example:

```text
Flask
```

## Question 2

What dependencies does it require?

Example:

```text
Flask
mysql-connector-python
```

## Question 3

What base image should I use?

Example:

```text
python:3.11-slim
```

## Question 4

What command starts the application?

Example:

```text
python app/app.py
```

## Question 5

Does it require another service?

Example:

```text
MySQL
```

## Question 6

Does that service require persistent data?

Example:

```text
MySQL → Docker Volume
```

## Question 7

How will services communicate?

Example:

```text
Docker Network
```

## Question 8

What configuration should be externalized?

Example:

```text
.env
```

## Question 9

Does the application depend on another service being ready?

Example:

```text
Healthcheck
depends_on
```

---

# 49. Complete Docker Project Flow

The complete thought process is:

```text
Application
     ↓
Identify dependencies
     ↓
Create requirements.txt
     ↓
Write Dockerfile
     ↓
Build image
     ↓
Run container
     ↓
Test application
     ↓
Add database
     ↓
Create Docker Compose
     ↓
Create network
     ↓
Create volume
     ↓
Add environment variables
     ↓
Add healthcheck
     ↓
Test complete application
     ↓
Tag image
     ↓
Push to Docker Hub
     ↓
Remove local image
     ↓
Pull image from Docker Hub
     ↓
Deploy again
     ↓
Verify
```

---

# 50. Practical Exercise

When practicing independently, do NOT immediately search for the complete solution.

Start with the requirement.

Example:

> "Containerize a Flask application."

Ask yourself:

```text
What does Flask need?
```

Answer:

```text
Python
Dependencies
Application code
Startup command
```

Then design:

```text
Dockerfile
```

Next requirement:

> "Run Flask and MySQL together."

Think:

```text
Multiple services
       ↓
Docker Compose
```

Next:

> "Database data must survive container deletion."

Think:

```text
Persistent storage
       ↓
Docker Volume
```

Next:

> "Flask must communicate with MySQL."

Think:

```text
Docker Network
```

Next:

> "Don't hard-code passwords."

Think:

```text
.env
```

Next:

> "Don't start Flask before MySQL is ready."

Think:

```text
Healthcheck
+
depends_on
```

This is how you become independent.

---

# 51. What to Memorize vs What to Look Up

## Memorize

Understand these concepts:

```text
Image
Container
Dockerfile
Volume
Network
Compose
Environment variables
Healthcheck
Port mapping
Docker Hub
```

Understand these flows:

```text
Dockerfile → Image → Container

Host port → Container port

Container → Network → Container

Container → Volume → Persistent data

Docker Hub → Image → Container
```

## Look up when needed

It is completely normal to look up:

```text
Exact Docker command syntax
Dockerfile syntax
Compose syntax
Healthcheck syntax
Advanced networking
Advanced volume options
Security configuration
```

Real DevOps engineers also use documentation.

The goal is not to memorize every command.

The goal is to know:

> **What needs to happen and what Docker feature solves it.**

---

# 52. Day 37 Practice Checklist

Before considering the practical complete, try to perform these tasks without copying previous commands:

* [ ] Create a simple Dockerfile
* [ ] Build an image
* [ ] List images
* [ ] Run a container
* [ ] Map a port
* [ ] Check container status
* [ ] View logs
* [ ] Enter a container
* [ ] Inspect a container
* [ ] Create a volume
* [ ] Mount a volume
* [ ] Create a network
* [ ] Connect containers to a network
* [ ] Run two services
* [ ] Create a Compose file
* [ ] Use `.env`
* [ ] Add a healthcheck
* [ ] Push an image to Docker Hub
* [ ] Pull the image again
* [ ] Deploy without rebuilding locally

---

# 53. Golden Rules

### Rule 1

```text
Dockerfile = How to build my image
```

### Rule 2

```text
Image = Packaged application
```

### Rule 3

```text
Container = Running image
```

### Rule 4

```text
Compose = Manage multiple services
```

### Rule 5

```text
Network = Container communication
```

### Rule 6

```text
Volume = Persistent data
```

### Rule 7

```text
.env = Configuration
```

### Rule 8

```text
Healthcheck = Is the service actually ready?
```

### Rule 9

```text
Port mapping = Host ↔ Container
```

### Rule 10

```text
Logs first when troubleshooting
```

---

# 54. Final Mental Picture

If you remember only one diagram, remember this:

```text
                 DOCKER
                    |
       ┌────────────┴────────────┐
       ↓                         ↓
   Dockerfile                  Compose
       ↓                         ↓
     Image              ┌────────┴────────┐
       ↓                ↓                 ↓
   Container          Web               Database
                         │                 │
                         └──── Network ────┘
                                           │
                                           ↓
                                      Volume
                                           │
                                           ↓
                                    Persistent Data
```

And for deployment:

```text
Dockerfile
    ↓
Build
    ↓
Image
    ↓
Tag
    ↓
Docker Hub
    ↓
Pull
    ↓
Container
```

---

# 55. Day 37 Summary

The purpose of Docker is not simply to memorize commands.

The important skill is to look at an application and identify:

```text
Application
    ↓
Dependencies
    ↓
Image
    ↓
Container
    ↓
Ports
    ↓
Network
    ↓
Database
    ↓
Volume
    ↓
Environment
    ↓
Health
    ↓
Deployment
```

Once this mental model is clear, Docker commands become tools for implementing the design rather than commands that must be memorized blindly.

---

## 🚀 Practice Principle

> **Don't copy the command first. Understand the problem first.**

Ask:

**What am I trying to achieve?**

Then:

**Which Docker concept solves it?**

Then:

**Which command implements that concept?**

Finally:

**How do I verify that it worked?**

That is the approach to use for all future DevOps projects.
