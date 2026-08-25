# Day 34 – Docker Compose: Real-World Multi-Container Apps

## Objective

Build and manage a more realistic multi-container application using Docker Compose.

The stack contains:

* Flask web application
* MySQL database
* Redis cache
* Custom Dockerfile
* Healthcheck
* `depends_on`
* Restart policies
* Named network
* Named volume
* Service labels
* Scaling experiment

---

# Project Structure

```text
day-34/
├── README.md
├── app/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── docker-compose.yml
└── day-34-compose-advanced.md
```

---

# Task 1 – Build Your Own App Stack

## Goal

Create a 3-service application stack:

```text
                 Flask Web
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
        MySQL               Redis
       Database              Cache
```

Services:

1. `web` – Python Flask
2. `db` – MySQL 8
3. `redis` – Redis 7

---

## Step 1 – Create the project

```bash
cd ~/90DaysofDevops/2026
mkdir -p day-34/app
cd day-34
```

Check:

```bash
pwd
ls
```

---

## Step 2 – Create application files

```bash
touch app/app.py
touch app/requirements.txt
touch app/Dockerfile
touch docker-compose.yml
touch day-34-compose-advanced.md
```

Check:

```bash
ls
ls app
```

---

# Flask Application

## `app/app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Day 34 - Flask App Updated Successfully!"

@app.route("/health")
def health():
    return "OK"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

The application provides:

```text
/        → Flask application message
/health  → Health response
```

The application listens on:

```text
0.0.0.0:5000
```

This is important because the application needs to be reachable from outside the container.

---

# Python Requirements

## `app/requirements.txt`

```text
Flask
```

`requirements.txt` tells pip which Python packages need to be installed.

---

# Dockerfile

## `app/Dockerfile`

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

## Dockerfile Explanation

### `FROM`

```dockerfile
FROM python:3.12-slim
```

Uses a lightweight Python image as the base image.

### `WORKDIR`

```dockerfile
WORKDIR /app
```

Sets `/app` as the working directory inside the container.

### `COPY`

```dockerfile
COPY requirements.txt .
```

Copies the Python requirements file into the container.

### Install dependencies

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

`-r requirements.txt` means:

> Install packages listed in `requirements.txt`.

`--no-cache-dir` tells pip not to keep its package download cache, helping keep the Docker image smaller.

### Copy application

```dockerfile
COPY app.py .
```

Copies the Flask application into the container.

### Expose port

```dockerfile
EXPOSE 5000
```

Documents that the application listens on port 5000.

### Start application

```dockerfile
CMD ["python", "app.py"]
```

Runs the Flask application when the container starts.

---

# Initial Docker Compose Configuration

## `docker-compose.yml`

```yaml
services:

  web:
    build: ./app
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: rootpassword
      DB_NAME: day34
      REDIS_HOST: redis

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: day34

  redis:
    image: redis:7
```

Validate the Compose file:

```bash
docker compose config
```

If valid, Docker Compose displays the expanded configuration.

---

# Start the Application

Build and start:

```bash
docker compose up --build
```

Or run in the background:

```bash
docker compose up -d --build
```

Check services:

```bash
docker compose ps
```

Expected services:

```text
web
db
redis
```

Test Flask:

```bash
curl http://localhost:5000
```

Expected:

```text
Hello from Day 34 Docker Compose!
```

Test health endpoint:

```bash
curl http://localhost:5000/health
```

Expected:

```text
OK
```

---

# Task 2 – depends_on & Healthchecks

## Why healthchecks?

A container being `running` does not necessarily mean that the application inside it is ready.

For example:

```text
MySQL container starts
        ↓
MySQL initializes
        ↓
MySQL becomes ready
```

Docker needs a healthcheck to determine when MySQL is actually ready.

---

## MySQL Healthcheck

Added:

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### Meaning

```text
test
↓
Command Docker runs to check MySQL

interval: 10s
↓
Check every 10 seconds

timeout: 5s
↓
Allow 5 seconds for the check

retries: 5
↓
Allow 5 failed checks before unhealthy
```

---

# depends_on

Added:

```yaml
depends_on:
  db:
    condition: service_healthy
```

This means:

> Start the Flask web service after MySQL passes its healthcheck.

Startup flow:

```text
MySQL starts
    ↓
Healthcheck runs
    ↓
MySQL becomes HEALTHY
    ↓
Flask web starts
```

Validate:

```bash
docker compose config
```

---

# Final Compose Configuration

At the end of the practical exercise, the Compose file was:

```yaml
services:

  web:
    build: ./app
    depends_on:
      db:
        condition: service_healthy
    restart: on-failure
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: rootpassword
      DB_NAME: day34
      REDIS_HOST: redis
    networks:
      - day34-network
    labels:
      project: "day-34"
      service: "web"

  db:
    image: mysql:8
    restart: on-failure
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: day34
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - day34-network
    labels:
      project: "day-34"
      service: "database"

  redis:
    image: redis:7
    networks:
      - day34-network
    labels:
      project: "day-34"
      service: "cache"

networks:
  day34-network:
    name: day34-network

volumes:
  mysql_data:
```

---

# Task 3 – Restart Policies

## `restart: always`

Initially tested:

```yaml
restart: always
```

This tells Docker to automatically restart the container when appropriate if it stops.

Example:

```text
Container stops
      ↓
Docker attempts restart
      ↓
Container runs again
```

---

## Check restart policy

```bash
docker inspect day-34-db-1 --format='{{.HostConfig.RestartPolicy.Name}}'
```

Expected:

```text
always
```

---

## `restart: on-failure`

Then changed to:

```yaml
restart: on-failure
```

This means Docker restarts the container when the container exits because of a failure.

Generally:

```text
Exit code 0
↓
Normal exit
↓
No restart
```

Whereas:

```text
Exit code != 0
↓
Failure
↓
Restart
```

---

## Restart Policy Comparison

| Policy           | Meaning                                                                           | Typical use                                   |
| ---------------- | --------------------------------------------------------------------------------- | --------------------------------------------- |
| `always`         | Keep restarting the container when it stops, subject to Docker's restart behavior | Long-running services                         |
| `on-failure`     | Restart when the container exits with a failure                                   | Applications that should recover from crashes |
| `no`             | Do not automatically restart                                                      | Default behavior                              |
| `unless-stopped` | Restart unless explicitly stopped                                                 | Long-running services                         |

---

# Task 4 – Custom Dockerfiles in Compose

The Compose file uses:

```yaml
build: ./app
```

This tells Compose:

> Build the web image using the Dockerfile inside `app/`.

---

## Make a Code Change

Original message:

```text
Hello from Day 34 Docker Compose!
```

Changed to:

```text
Day 34 - Flask App Updated Successfully!
```

---

## Rebuild and Restart

One command:

```bash
docker compose up -d --build
```

`--build`:

> Rebuild the image.

`-d`:

> Run containers in detached/background mode.

Test:

```bash
curl http://localhost:5000
```

Expected:

```text
Day 34 - Flask App Updated Successfully!
```

This demonstrates the Compose workflow:

```text
Change code
    ↓
docker compose up -d --build
    ↓
New image
    ↓
Updated container
```

---

# Task 5 – Named Networks

Initially Docker Compose created an automatic network:

```text
day-34_default
```

We replaced it with an explicit network:

```yaml
networks:
  day34-network:
    name: day34-network
```

Each service uses:

```yaml
networks:
  - day34-network
```

---

## Check Networks

```bash
docker network ls
```

Expected:

```text
day34-network
```

Inspect:

```bash
docker network inspect day34-network
```

The web, database, and Redis containers should be connected to the network.

---

# Container-to-Container Communication

The services can communicate using service names.

For example:

```text
web → db
```

uses:

```text
DB_HOST=db
```

and:

```text
web → redis
```

uses:

```text
REDIS_HOST=redis
```

Docker Compose provides internal DNS for service names.

Therefore, we don't need to hard-code container IP addresses.

---

# Named Volumes

MySQL uses:

```yaml
volumes:
  - mysql_data:/var/lib/mysql
```

The named volume is declared:

```yaml
volumes:
  mysql_data:
```

This provides persistent storage for MySQL.

---

## Check Volumes

```bash
docker volume ls
```

Compose may create the actual Docker volume with a project prefix, such as:

```text
day-34_mysql_data
```

Therefore, this may not work:

```bash
docker volume inspect mysql_data
```

Instead, first run:

```bash
docker volume ls
```

Then inspect the actual volume name:

```bash
docker volume inspect day-34_mysql_data
```

---

## Why use a named volume?

Without a volume:

```text
Container deleted
       ↓
Database data may be lost
```

With a named volume:

```text
Container deleted
       ↓
Volume remains
       ↓
Database data remains
```

---

# Service Labels

Labels were added for organization.

Web:

```yaml
labels:
  project: "day-34"
  service: "web"
```

Database:

```yaml
labels:
  project: "day-34"
  service: "database"
```

Redis:

```yaml
labels:
  project: "day-34"
  service: "cache"
```

Labels make it easier to identify and organize containers and services.

Check labels:

```bash
docker inspect day-34-web-1 --format='{{json .Config.Labels}}'
```

---

# Task 6 – Scaling

## Scale web to 3 replicas

Command:

```bash
docker compose up -d --scale web=3
```

The intention is:

```text
web-1
web-2
web-3
```

---

## What happened?

Docker successfully started/kept the first web container, but another replica failed with:

```text
Bind for 0.0.0.0:5000 failed: port is already allocated
```

---

# Why Scaling Failed

Our Compose file contains:

```yaml
ports:
  - "5000:5000"
```

This means:

```text
Host port 5000
       ↓
Container port 5000
```

The first replica can use:

```text
Host 5000 → web-1:5000
```

But the second replica also tries:

```text
Host 5000 → web-2:5000
```

The host port is already occupied.

Therefore:

```text
5000 → web-1   ✅

5000 → web-2   ❌

5000 → web-3   ❌
```

---

# Production Scaling Concept

A reverse proxy/load balancer can sit in front of multiple application replicas:

```text
                 ┌── web-1
                 │
Internet → Proxy ┼── web-2
                 │
                 └── web-3
```

The proxy owns the public port and distributes traffic between the replicas.

This is a common architecture for scalable applications.

---

# Clean Up Scaling Experiment

After testing scaling:

```bash
docker compose down
```

Start the normal single-replica stack:

```bash
docker compose up -d --build
```

Check:

```bash
docker compose ps
```

Expected services:

```text
web
db
redis
```

---

# Useful Docker Compose Commands

## Validate Compose

```bash
docker compose config
```

## Start

```bash
docker compose up
```

## Start in background

```bash
docker compose up -d
```

## Build and start

```bash
docker compose up -d --build
```

## Stop and remove Compose containers

```bash
docker compose down
```

## View running services

```bash
docker compose ps
```

## View all containers including stopped ones

```bash
docker compose ps -a
```

## View logs

```bash
docker compose logs
```

## View logs for one service

```bash
docker compose logs db
```

## Restart a service

```bash
docker compose restart db
```

## Start a stopped service

```bash
docker compose start db
```

## Stop a service

```bash
docker compose stop db
```

## Scale a service

```bash
docker compose up -d --scale web=3
```

---

# Important Docker Compose Concepts Learned

## 1. `build`

```yaml
build: ./app
```

Builds a custom image using the Dockerfile.

---

## 2. `depends_on`

```yaml
depends_on:
  db:
    condition: service_healthy
```

Controls service startup dependency.

---

## 3. `healthcheck`

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

Checks whether the service is actually healthy.

---

## 4. Restart policies

```yaml
restart: always
```

or:

```yaml
restart: on-failure
```

Control automatic container restart behavior.

---

## 5. Networks

```yaml
networks:
  day34-network:
```

Creates an explicit Docker network.

---

## 6. Volumes

```yaml
volumes:
  mysql_data:
```

Provides persistent database storage.

---

## 7. Labels

```yaml
labels:
  project: "day-34"
```

Adds metadata to services/containers.

---

## 8. Scaling

```bash
docker compose up -d --scale web=3
```

Creates multiple service replicas, but fixed host-port mappings can prevent multiple replicas from starting.

---

# Final Architecture

```text
                         Docker Compose
                              │
                    ┌─────────┴─────────┐
                    │   day34-network   │
                    └─────────┬─────────┘
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ↓                ↓                ↓
       ┌───────────┐    ┌───────────┐    ┌───────────┐
       │   Flask   │    │   MySQL   │    │   Redis   │
       │    Web    │    │    DB     │    │   Cache   │
       │   :5000   │    │   :3306   │    │   :6379   │
       └───────────┘    └─────┬─────┘    └───────────┘
                              │
                              ↓
                       mysql_data volume
```

---

# Final Verification

Run:

```bash
docker compose config
```

Then:

```bash
docker compose ps
```

Test the application:

```bash
curl http://localhost:5000
```

Expected:

```text
Day 34 - Flask App Updated Successfully!
```

Test health:

```bash
curl http://localhost:5000/health
```

Expected:

```text
OK
```

Check network:

```bash
docker network ls
```

Check volume:

```bash
docker volume ls
```

---

# Troubleshooting Notes

## Error: `empty compose file`

Usually means Docker Compose is being executed from the wrong directory or the `docker-compose.yml` file is empty.

Check:

```bash
pwd
ls
cat docker-compose.yml
```

Make sure you are inside:

```text
90DaysofDevops/2026/day-34
```

---

## Error: `services.db.environment.test must be a boolean...`

Usually caused by incorrect YAML indentation.

Correct:

```yaml
environment:
  MYSQL_DATABASE: day34

healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
```

Incorrect:

```yaml
environment:
  MYSQL_DATABASE: day34
  healthcheck:
    test: ...
```

---

## Error: `port is already allocated`

Example:

```text
Bind for 0.0.0.0:5000 failed:
port is already allocated
```

Check which container uses the port:

```bash
docker ps
```

Or:

```bash
docker compose ps
```

For scaling, this happens because multiple replicas cannot all bind the same host port.

---

## Accidentally Created Nested Directory

Be careful when already inside `day-34`.

If your prompt shows:

```text
~/90DaysofDevops/2026/day-34
```

do NOT run:

```bash
mkdir -p day-34/app
```

because it creates:

```text
day-34/day-34/app
```

Instead use:

```bash
mkdir -p app
```

---

# Git Submission

Check status:

```bash
git status
```

Add Day 34 files:

```bash
git add 2026/day-34/
```

Commit:

```bash
git commit -m "Complete Day 34 Docker Compose advanced"
```

Push:

```bash
git push
```

If your branch requires a specific remote/branch:

```bash
git remote -v
git branch
```

Then push to the appropriate branch.

---

# Day 34 Summary

Today I built a real-world multi-container application using Docker Compose.

I practiced:

* Flask application containers
* MySQL database containers
* Redis cache containers
* Custom Dockerfiles
* `docker compose up`
* `docker compose up -d`
* `docker compose up --build`
* `depends_on`
* Database healthchecks
* Restart policies
* Named networks
* Named volumes
* Docker labels
* Service scaling
* Port mapping limitations
* Container troubleshooting

The most important lesson:

> **Docker Compose allows multiple related services to be defined, configured, connected, and managed together from a single YAML file.**

Another important lesson:

> **A container being "running" does not necessarily mean the application inside it is ready. Healthchecks and `depends_on` conditions can be used to coordinate service startup.**

And from the scaling experiment:

> **Multiple replicas cannot all bind the same host port. In a production architecture, a reverse proxy or load balancer can distribute traffic among application replicas.**

---

# Day 34 Completed ✅

```text
Task 1 – 3-service application       ✅
Task 2 – Healthchecks                ✅
Task 2 – depends_on                  ✅
Task 3 – restart: always             ✅
Task 3 – restart: on-failure         ✅
Task 4 – Custom Dockerfile           ✅
Task 4 – Rebuild after code change   ✅
Task 5 – Named network               ✅
Task 5 – Named volume                ✅
Task 5 – Service labels              ✅
Task 6 – Scaling experiment          ✅
Task 6 – Port mapping limitation     ✅
```

**#90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham**
