# Day 33 – Docker Compose: Multi-Container Basics

## Task Objective

Today's goal was to learn Docker Compose and understand how multiple containers can be managed together using a single YAML file.

Docker Compose allows us to define:

- Containers/services
- Networks
- Volumes
- Ports
- Environment variables
- Dependencies

in one `docker-compose.yml` file.

Instead of manually creating and connecting containers one by one, Compose allows us to manage the complete application using commands such as:

```bash
docker compose up
docker compose down
```

---

# Task 1 – Install & Verify Docker Compose

## 1. Check Docker Compose

Command:

```bash
docker compose version
```

Output:

```text
Docker Compose version v5.3.1
```

## 2. Check Docker Version

Command:

```bash
docker --version
```

Output:

```text
Docker version 29.6.2, build dfc4efb
```

## 3. Check Docker Information

Command:

```bash
docker info
```

Docker was running successfully using:

```text
Context: desktop-linux
Operating System: Docker Desktop
Kernel: WSL2
Architecture: x86_64
```

### Result

Docker and Docker Compose were successfully installed and working.

---

# Task 2 – First Compose File

## Objective

Create a simple Compose application running one Nginx container.

## Step 1 – Create Folder

```bash
cd ~/90DaysofDevops/2026/day-33
mkdir compose-basics
cd compose-basics
```

## Step 2 – Create Compose File

```bash
touch docker-compose.yml
```

The Compose file:

```yaml
services:
  nginx:
    image: nginx:latest
    ports:
      - "8081:80"
```

## Explanation

```yaml
services:
```

Defines the services/containers in the application.

```yaml
nginx:
```

Defines an Nginx service.

```yaml
image: nginx:latest
```

Uses the Nginx Docker image.

```yaml
ports:
  - "8081:80"
```

Maps:

```text
Host port 8081 → Container port 80
```

## Step 3 – Validate Compose File

```bash
docker compose config
```

This checks the Compose configuration and shows how Docker interprets the YAML file.

## Step 4 – Start Nginx

```bash
docker compose up
```

Nginx started successfully.

Open in browser:

```text
http://localhost:8081
```

The Nginx welcome page was displayed.

## Step 5 – Stop and Remove

```bash
docker compose down
```

This removed:

- Nginx container
- Compose network

The Nginx image remained available.

---

# Task 2 Troubleshooting – Port Conflict

Initially, Nginx was configured to use port `8080`.

The following error occurred:

```text
Bind for 0.0.0.0:8080 failed: port is already allocated
```

Checked running containers:

```bash
docker ps
```

Found an existing container:

```text
nginx-bind-mount
```

using:

```text
0.0.0.0:8080->80/tcp
```

Therefore port `8080` was already occupied.

The problem was solved by removing the old Nginx container that was using port 8080 and then starting the Compose Nginx application.

### Lesson Learned

Only one container/application can normally bind to the same host port at a time.

---

# Task 3 – Two-Container Setup

## Objective

Create a multi-container application containing:

- WordPress
- MySQL

The two services should communicate through the Compose network.

MySQL should use persistent storage.

## Why WordPress + MySQL?

WordPress is a real example of a multi-container application.

WordPress contains the application logic and website files, while MySQL stores the website's structured data such as:

- Users
- Posts
- Pages
- Settings
- Comments
- WordPress configuration data

Therefore we need two services:

```text
WordPress
    |
    | Database connection
    v
MySQL
```

This is useful for learning Docker Compose because real applications often consist of multiple services that communicate with each other.

---

# Step 1 – Create Project

```bash
cd ~/90DaysofDevops/2026/day-33
mkdir wordpress-mysql
cd wordpress-mysql
```

## Step 2 – Create Compose File

```bash
touch docker-compose.yml
```

Compose file:

```yaml
services:

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - mysql_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    ports:
      - "8082:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - db

volumes:
  mysql_data:
```

---

# Understanding the Compose File

## MySQL Service

```yaml
db:
  image: mysql:8
```

The service is called:

```text
db
```

The MySQL image is:

```text
mysql:8
```

---

## MySQL Environment Variables

```yaml
MYSQL_ROOT_PASSWORD: rootpass
MYSQL_DATABASE: wordpress
MYSQL_USER: wpuser
MYSQL_PASSWORD: wppass
```

These configure the MySQL database.

The database name is:

```text
wordpress
```

The application user is:

```text
wpuser
```

---

# WordPress Service

```yaml
wordpress:
  image: wordpress:latest
```

This creates the WordPress application container.

Port mapping:

```yaml
ports:
  - "8082:80"
```

means:

```text
Host port 8082 → WordPress container port 80
```

WordPress was accessed using:

```text
http://localhost:8082
```

---

# WordPress Database Connection

The most important configuration is:

```yaml
WORDPRESS_DB_HOST: db:3306
```

Here:

```text
db
```

is the MySQL Compose service name.

```text
3306
```

is the MySQL port inside the Docker network.

Therefore:

```text
WordPress → db:3306 → MySQL
```

---

# Why Not localhost?

Inside a container:

```text
localhost
```

means the current container.

Therefore WordPress should NOT use:

```text
localhost:3306
```

because MySQL is running in a different container.

Instead WordPress uses:

```text
db:3306
```

because Docker Compose provides service-name based DNS.

---

# Compose Automatic Network

When the application is started, Docker Compose automatically creates a network.

Example:

```text
wordpress-mysql_default
```

Both services are connected to this network:

```text
WordPress
    |
    | Docker Compose Network
    |
MySQL
```

This allows WordPress to communicate with MySQL using:

```text
db
```

instead of an IP address.

---

# MySQL Named Volume

The Compose file contains:

```yaml
volumes:
  - mysql_data:/var/lib/mysql
```

This means MySQL stores its database files in a Docker named volume.

The volume is declared at the bottom:

```yaml
volumes:
  mysql_data:
```

Compose created the volume:

```text
wordpress-mysql_mysql_data
```

---

# Why Do We Need a Volume?

Containers are temporary.

If the MySQL container is deleted, its writable container filesystem can be deleted as well.

Without a volume:

```text
MySQL Container
      |
      v
Database data
      |
Container deleted
      |
      v
Data can be lost
```

With a volume:

```text
MySQL Container
      |
      v
Named Volume
      |
      v
Database data
```

If the container is deleted:

```text
MySQL Container ❌
Named Volume    ✅
Database Data   ✅
```

A new MySQL container can attach to the same volume.

---

# Step 3 – Validate Compose File

```bash
docker compose config
```

The configuration showed:

```text
db
wordpress
wordpress-mysql_default
wordpress-mysql_mysql_data
```

This confirmed that Compose understood the services, network, and volume.

---

# Step 4 – Start WordPress + MySQL

```bash
docker compose up
```

Both containers started.

MySQL logs showed:

```text
ready for connections
```

WordPress Apache also started successfully.

An Apache ServerName warning appeared, but it did not prevent WordPress from running.

---

# Step 5 – Open WordPress

Open:

```text
http://localhost:8082
```

The WordPress language selection screen appeared.

Selected:

```text
English (United States)
```

Completed the WordPress setup.

---

# Task 3 – Persistence Test

## Step 1 – Check Services

```bash
docker compose ps
```

Both services were running:

```text
db          Up
wordpress   Up
```

## Step 2 – Remove Compose Containers

```bash
docker compose down
```

This removed:

- WordPress container
- MySQL container
- Compose network

But the named volume remained.

## Step 3 – Verify Volume

```bash
docker volume ls
```

The following volume was still present:

```text
wordpress-mysql_mysql_data
```

## Step 4 – Start Again

```bash
docker compose up -d
```

Then:

```bash
docker compose ps
```

Both services started again.

## Step 5 – Open WordPress

```text
http://localhost:8082
```

The existing WordPress installation/data was still available.

### Result

The persistence test was successful.

This proved that the MySQL named volume preserves database data even after the MySQL container is removed and recreated.

---

# Task 4 – Docker Compose Commands

## 4.1 Start Services in Detached Mode

```bash
docker compose up -d
```

The `-d` option means detached mode.

The containers run in the background and the terminal becomes available again.

---

# 4.2 View Running Services

```bash
docker compose ps
```

Example:

```text
NAME                          SERVICE     STATUS
wordpress-mysql-db-1          db          Up
wordpress-mysql-wordpress-1   wordpress   Up
```

### Difference

```bash
docker ps
```

shows all running Docker containers.

```bash
docker compose ps
```

shows containers belonging to the current Compose project.

---

# 4.3 View Logs of All Services

```bash
docker compose logs
```

To view only recent logs:

```bash
docker compose logs --tail=20
```

To continuously follow logs:

```bash
docker compose logs -f
```

The `-f` option means follow the logs.

---

# 4.4 View Logs of a Specific Service

MySQL:

```bash
docker compose logs --tail=20 db
```

WordPress:

```bash
docker compose logs --tail=20 wordpress
```

This is useful for troubleshooting a specific service.

---

# 4.5 Stop Services Without Removing Them

```bash
docker compose stop
```

This stops the containers but does not remove them.

Check stopped containers:

```bash
docker compose ps -a
```

The containers should appear with an exited/stopped status.

---

# 4.6 Start Previously Stopped Services

```bash
docker compose start
```

This starts the existing stopped containers again.

Check:

```bash
docker compose ps
```

Both services should show:

```text
Up
```

---

# 4.7 Remove Containers and Network

```bash
docker compose down
```

This removes:

- Compose containers
- Compose network

Normally, named volumes are kept.

Verify:

```bash
docker volume ls
```

The following volume should still exist:

```text
wordpress-mysql_mysql_data
```

---

# Important Difference – stop vs down

## docker compose stop

```text
Running containers
        |
        v
docker compose stop
        |
        v
Containers stopped
        |
        v
Containers still exist
```

## docker compose down

```text
Running containers
        |
        v
docker compose down
        |
        v
Containers removed
Network removed
Named volume normally kept
```

## docker compose down -v

```text
docker compose down -v
```

removes the named volumes too.

Therefore, `down -v` should be used carefully because it can remove persistent database data.

---

# 4.8 Rebuild Images

Command:

```bash
docker compose build
```

Output during this task:

```text
No services to build
```

This was not an error.

The current Compose file uses existing images:

```yaml
image: mysql:8
image: wordpress:latest
```

and does not contain a custom:

```yaml
build:
```

Therefore there was nothing for Compose to build.

---

# Image vs Build

## image

```yaml
image: nginx:latest
```

Means:

```text
Use an existing Docker image
```

## build

```yaml
build: .
```

Means:

```text
Build an image using a Dockerfile
```

For example:

```yaml
services:
  app:
    build: .
```

would tell Compose to build an image from the Dockerfile in the current directory.

---

# Task 5 – Environment Variables

Environment variables are used to provide configuration to containers.

They are useful because application configuration can be separated from the main Compose structure.

---

# 5.1 Environment Variables Directly in Compose

Created:

```text
day-33/environment/docker-compose.yml
```

Configuration:

```yaml
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: testdb
      MYSQL_USER: testuser
      MYSQL_PASSWORD: testpass
```

Validated using:

```bash
docker compose config
```

Compose displayed the environment variables correctly.

---

# 5.2 Using a .env File

Created:

```text
day-33/environment/.env
```

Contents:

```text
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=testdb
MYSQL_USER=testuser
MYSQL_PASSWORD=testpass
```

Then changed `docker-compose.yml` to:

```yaml
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
```

---

# How .env Works

The `.env` file contains:

```text
MYSQL_DATABASE=testdb
```

The Compose file references it using:

```yaml
${MYSQL_DATABASE}
```

Docker Compose reads the `.env` file and substitutes the value.

Conceptually:

```text
.env
 |
 | MYSQL_DATABASE=testdb
 v
docker-compose.yml
 |
 | ${MYSQL_DATABASE}
 v
testdb
```

Verified using:

```bash
docker compose config
```

The variables were correctly resolved.

---

# Important Security Note

For practice, simple passwords were used:

```text
rootpass
testpass
```

In real projects, passwords and secrets should not be committed to GitHub in plain text.

A `.env` file containing real secrets should normally be added to `.gitignore`.

Example:

```text
.env
```

For production systems, proper secret-management solutions should be used.

---

# Key Concepts Learned

## Docker Image

An image is a template used to create containers.

Examples:

```text
nginx:latest
mysql:8
wordpress:latest
```

---

## Container

A container is a running instance of an image.

Examples:

```text
wordpress-mysql-db-1
wordpress-mysql-wordpress-1
```

---

## Service

A service is a container definition inside a Compose file.

Example:

```yaml
services:
  db:
  wordpress:
```

---

## Network

Compose automatically creates a network for services.

Example:

```text
wordpress-mysql_default
```

The services can communicate using service names.

```text
wordpress → db:3306 → mysql
```

---

## Volume

A volume provides persistent storage.

Example:

```text
wordpress-mysql_mysql_data
```

It stores MySQL data outside the container's writable layer.

---

## Environment Variables

Environment variables provide configuration to containers.

Example:

```text
MYSQL_DATABASE=testdb
```

They can be referenced in Compose using:

```text
${MYSQL_DATABASE}
```

---

# Docker Compose Architecture Learned

```text
                    Docker Compose
                          |
             docker-compose.yml
                          |
             +------------+------------+
             |                         |
             v                         v
       WordPress Service          MySQL Service
             |                         |
             |                         |
             +------ db:3306 ----------+
                          |
                          v
                    Named Volume
                    mysql_data
```

---

# Most Important Commands Learned

```bash
docker compose version

docker compose config

docker compose up

docker compose up -d

docker compose ps

docker compose ps -a

docker compose logs

docker compose logs --tail=20

docker compose logs db

docker compose logs wordpress

docker compose stop

docker compose start

docker compose down

docker compose build
```

---

# Common Troubleshooting

## Port Already Allocated

Error encountered:

```text
Bind for 0.0.0.0:8080 failed: port is already allocated
```

Checked:

```bash
docker ps
```

Found an existing Nginx container using port 8080:

```text
nginx-bind-mount
```

The old container was removed and the Compose application was started successfully.

### Lesson

A host port cannot normally be used by two containers at the same time.

---

# Final Project Structure

```text
2026/
└── day-33/
    |
    ├── compose-basics/
    │   └── docker-compose.yml
    |
    ├── wordpress-mysql/
    │   └── docker-compose.yml
    |
    ├── environment/
    │   ├── docker-compose.yml
    │   └── .env
    |
    └── day-33-compose.md
```

---

# Day 33 Final Summary

Day 33 helped me understand how Docker Compose can manage multi-container applications.

I learned that instead of manually creating containers, networks, and volumes, I can define them in a single Compose YAML file.

The WordPress + MySQL example helped me understand real multi-container communication:

```text
WordPress
    |
    | db:3306
    v
MySQL
    |
    v
Named Volume
```

The most important concepts I learned were:

1. Docker Compose manages multiple services.
2. Compose automatically creates a network.
3. Services can communicate using service names.
4. WordPress uses `db:3306` to reach MySQL.
5. Named volumes provide persistent database storage.
6. `docker compose stop` stops containers without removing them.
7. `docker compose down` removes containers and networks.
8. `.env` files can provide configuration values.
9. `docker compose config` helps validate and inspect the final configuration.
10. Port conflicts occur when another process/container is already using the requested host port.

---

# Submission Checklist

- [x] Docker Compose installed and verified
- [x] Docker version verified
- [x] First Nginx Compose file created
- [x] Nginx started with Compose
- [x] Nginx tested in browser
- [x] Nginx stopped with Compose
- [x] WordPress + MySQL Compose file created
- [x] WordPress and MySQL started together
- [x] Compose network used automatically
- [x] WordPress connected to MySQL using service name
- [x] MySQL named volume created
- [x] WordPress persistence tested
- [x] Detached mode practiced
- [x] Service status practiced
- [x] Logs practiced
- [x] Service-specific logs practiced
- [x] Stop/start commands practiced
- [x] Down command practiced
- [x] Build command practiced
- [x] Environment variables practiced
- [x] `.env` file created
- [x] `.env` variable substitution verified
- [x] `day-33-compose.md` created

---

# Day 33 Completed Successfully

Docker Compose Multi-Container Basics successfully practiced.

#90DaysOfDevOps
#DevOpsKaJosh
#TrainWithShubham
