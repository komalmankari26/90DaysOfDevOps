# Day 32 – Docker Volumes & Networking

## Tasks 1, 2 & 3 – Docker Volumes and Bind Mounts

### Objective

Today's goal was to understand **data persistence in Docker** using:

* Containers without volumes
* Docker named volumes
* Bind mounts

Containers are **ephemeral**, which means data stored only inside a container can be lost when the container is removed.

---

# Task 1 – The Problem: Data Without a Volume

The first task was to understand what happens to database data when a container is removed **without explicitly using a Docker volume**.

## Step 1 – Run MySQL Container

```bash
docker run -d --name mysql-test -e MYSQL_ROOT_PASSWORD=root mysql
```

### Explanation

* `docker run` → Creates and starts a new container.
* `-d` → Runs the container in detached/background mode.
* `--name mysql-test` → Gives the container the name `mysql-test`.
* `-e MYSQL_ROOT_PASSWORD=root` → Sets the MySQL root password.
* `mysql` → Uses the MySQL image from Docker Hub.

---

## Step 2 – Check Running Containers

```bash
docker ps
```

I used this command to verify that the MySQL container was running.

Expected output should show:

```text
mysql-test
```

---

## Step 3 – Enter the MySQL Container

```bash
docker exec -it mysql-test mysql -uroot -proot
```

### Explanation

* `docker exec` → Executes a command inside a running container.
* `-it` → Opens an interactive terminal.
* `mysql-test` → Target container.
* `mysql -uroot -proot` → Logs into MySQL as the root user.

---

## Step 4 – Create a Test Database

Inside MySQL:

```sql
CREATE DATABASE devopsdb;
```

Check the databases:

```sql
SHOW DATABASES;
```

I confirmed that `devopsdb` was created.

Exit MySQL:

```sql
EXIT;
```

---

## Step 5 – Remove the Container

```bash
docker rm -f mysql-test
```

### Explanation

`docker rm -f` forcefully removes the container.

At this point, the MySQL container and its writable container-layer data were removed.

---

## Step 6 – Create MySQL Container Again

```bash
docker run -d --name mysql-test -e MYSQL_ROOT_PASSWORD=root mysql
```

This creates a completely new MySQL container.

Check the databases again:

```bash
docker exec -it mysql-test mysql -uroot -proot -e "SHOW DATABASES;"
```

The previously created `devopsdb` database is no longer available.

### Observation

This demonstrated the main problem:

> **Container data should not be used as the only place to store important data.**

If the container is removed, data stored in its writable layer can be lost.

---

# Task 2 – Docker Named Volume

The second task was to solve the persistence problem using a **Docker named volume**.

A named volume is managed by Docker and can exist independently of a container.

---

## Step 1 – Create a Named Volume

```bash
docker volume create mysql-data
```

Expected output:

```text
mysql-data
```

---

## Step 2 – List Docker Volumes

```bash
docker volume ls
```

I verified that `mysql-data` was created.

Example:

```text
DRIVER    VOLUME NAME
local     mysql-data
```

---

## Step 3 – Inspect the Volume

```bash
docker volume inspect mysql-data
```

This displays information about the volume, including:

* Volume name
* Driver
* Mount point
* Docker-managed location

---

## Step 4 – Run MySQL Using the Volume

```bash
docker run -d \
  --name mysql-volume \
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql-data:/var/lib/mysql \
  mysql
```

### Explanation

The important part is:

```bash
-v mysql-data:/var/lib/mysql
```

This connects the Docker volume:

```text
mysql-data
```

to MySQL's data directory:

```text
/var/lib/mysql
```

Therefore, MySQL data is stored in the Docker volume instead of only inside the container's writable layer.

---

## Step 5 – Verify the Container

```bash
docker ps
```

I confirmed that `mysql-volume` was running.

---

## Step 6 – Create a Database

```bash
docker exec -it mysql-volume mysql -uroot -proot
```

Inside MySQL:

```sql
CREATE DATABASE devopsdb;
```

Verify:

```sql
SHOW DATABASES;
```

Then exit:

```sql
EXIT;
```

---

## Step 7 – Remove the Container

```bash
docker rm -f mysql-volume
```

The container was removed.

However, the volume was **not** removed.

---

## Step 8 – Check the Volume

```bash
docker volume ls
```

I verified that:

```text
mysql-data
```

was still present.

This proves that the volume has a lifecycle independent of the container.

---

## Step 9 – Create Another MySQL Container Using the Same Volume

```bash
docker run -d \
  --name mysql-volume-new \
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql-data:/var/lib/mysql \
  mysql
```

---

## Step 10 – Verify the Database

```bash
docker exec -it mysql-volume-new mysql -uroot -proot -e "SHOW DATABASES;"
```

The previously created:

```text
devopsdb
```

should still exist.

### Observation

The container was deleted, but the database remained because the data was stored in the named volume.

### Important Learning

> **Docker volumes provide persistent storage that survives container deletion.**

---

## Step 11 – Remove the Test Container

```bash
docker rm -f mysql-volume-new
```

The volume still exists.

---

## Step 12 – Remove the Volume

Only remove the volume when the data is no longer required:

```bash
docker volume rm mysql-data
```

Verify:

```bash
docker volume ls
```

The `mysql-data` volume should no longer appear.

---

# Task 3 – Bind Mount

The third task was to understand **bind mounts**.

A bind mount connects a specific directory on the host machine to a directory inside the container.

### Difference

**Named volume:**

```text
Docker manages the storage
```

**Bind mount:**

```text
I choose the storage location on my computer
```

---

# Step 1 – Create a Project Directory

In Git Bash, I created a directory for the bind mount:

```bash
mkdir -p ~/90DaysofDevOps/2026/day-32/bind-mount-demo
```

Move into the directory:

```bash
cd ~/90DaysofDevOps/2026/day-32/bind-mount-demo
```

---

# Step 2 – Create a Custom HTML Page

Create a file named:

```text
index.html
```

Example content:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Bind Mount Demo</title>
</head>
<body>
    <h1>Hello from Docker Bind Mount!</h1>
    <p>This page is coming from my host machine.</p>
</body>
</html>
```

The important point is that this file exists on the **host machine**, not originally inside the container.

---

# Step 3 – Run Nginx With a Bind Mount

From the project directory:

```bash
docker run -d \
  --name nginx-bind-mount \
  -p 8080:80 \
  -v "$(pwd):/usr/share/nginx/html" \
  nginx
```

### Explanation

```text
--name nginx-bind-mount
```

Gives the container a name.

```text
-p 8080:80
```

Maps:

```text
Host port 8080 → Container port 80
```

```text
-v "$(pwd):/usr/share/nginx/html"
```

Maps the current host directory to Nginx's default web directory.

```text
Host directory
       ↓
/usr/share/nginx/html
       ↓
Nginx container
```

---

# Step 4 – Check the Container

```bash
docker ps
```

I verified that the Nginx container was running.

---

# Step 5 – Open the Website

Open a browser and visit:

```text
http://localhost:8080
```

The custom `index.html` page should be displayed.

---

# Step 6 – Modify the HTML File

I changed the content of `index.html` on my host machine.

For example:

```html
<h1>Docker Bind Mount Working!</h1>
<p>I changed this file on my host machine.</p>
```

Save the file.

Refresh:

```text
http://localhost:8080
```

The updated page should appear.

### Observation

I did not need to rebuild the Docker image or recreate the container.

The change was immediately available because the host directory is mounted into the container.

---

# Step 7 – Verify the Bind Mount

Run:

```bash
docker inspect nginx-bind-mount
```

Look for the `Mounts` section.

It shows information about:

* Source
* Destination
* Type

The destination should contain:

```text
/usr/share/nginx/html
```

The mount type should be:

```text
bind
```

---

# Step 8 – Stop the Container

```bash
docker stop nginx-bind-mount
```

Check:

```bash
docker ps
```

The container should no longer appear in the running-container list.

---

# Step 9 – Start the Container Again

```bash
docker start nginx-bind-mount
```

Check:

```bash
docker ps
```

Then open:

```text
http://localhost:8080
```

The custom page should still be available.

---

# Step 10 – Remove the Container

```bash
docker rm -f nginx-bind-mount
```

The container is removed.

But the `index.html` file is still present on my host machine.

This is because the file belongs to the host directory.

---

# Named Volume vs Bind Mount

| Feature                         | Named Volume                | Bind Mount                     |
| ------------------------------- | --------------------------- | ------------------------------ |
| Storage managed by              | Docker                      | User/Host OS                   |
| Location                        | Docker-managed              | User-selected                  |
| Easy to move between containers | Yes                         | Yes                            |
| Good for databases              | Yes                         | Possible                       |
| Good for development            | Yes                         | Excellent                      |
| Directly edit files from host   | Not normally                | Yes                            |
| Example                         | `mysql-data:/var/lib/mysql` | `$(pwd):/usr/share/nginx/html` |

---

# Important Docker Commands Learned

## Containers

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

Start a container:

```bash
docker start <container-name>
```

Stop a container:

```bash
docker stop <container-name>
```

Remove a container:

```bash
docker rm <container-name>
```

Force remove:

```bash
docker rm -f <container-name>
```

---

## Volumes

Create a volume:

```bash
docker volume create <volume-name>
```

List volumes:

```bash
docker volume ls
```

Inspect a volume:

```bash
docker volume inspect <volume-name>
```

Remove a volume:

```bash
docker volume rm <volume-name>
```

---

## Inspect Container Mounts

```bash
docker inspect <container-name>
```

This can be used to check the container's:

* Mounts
* Network
* Environment
* Ports
* Configuration

---

# Key Concepts

## 1. Container Storage

Data stored only inside a container can disappear when the container is removed.

```text
Container
   |
   └── Writable Layer
           |
           └── Data
```

Remove the container:

```text
Container ❌
Data ❌
```

---

## 2. Named Volume

A named volume exists separately from the container.

```text
Container
    |
    ↓
Named Volume
    |
    ↓
Persistent Data
```

Remove container:

```text
Container ❌
Volume ✅
Data ✅
```

---

## 3. Bind Mount

A bind mount connects a host directory to a container directory.

```text
Host Machine
    |
    |  Bind Mount
    ↓
Container
```

Example:

```bash
-v "$(pwd):/usr/share/nginx/html"
```

---

# What I Learned

### Task 1

I learned that container storage is not suitable as the only place for important persistent data.

### Task 2

I learned how to create and use a **named Docker volume**.

I also verified that database data remains available even after deleting the container.

### Task 3

I learned how to use a **bind mount** to connect a directory on my computer to a directory inside a container.

I also learned that changes made to the host files can be reflected inside the running container.

---

# Final Summary

```text
Without Volume
      ↓
Container removed
      ↓
Data can be lost


Named Volume
      ↓
Container
      ↓
Docker Volume
      ↓
Data persists after container removal


Bind Mount
      ↓
Host Directory
      ↕
Container Directory
      ↓
Changes can be shared directly
```

## Main Takeaway

> **Containers are temporary, but application data often needs to survive. Docker volumes and bind mounts provide ways to separate data from the container lifecycle.**

---

# Day 32 – Quick Revision

### Named Volume

```bash
docker volume create myvolume
```

```bash
docker run -d --name mycontainer -v myvolume:/data nginx
```

### Bind Mount

```bash
docker run -d --name mycontainer -v "$(pwd):/data" nginx
```

### Check Volumes

```bash
docker volume ls
```

### Inspect Volume

```bash
docker volume inspect myvolume
```

### Inspect Container

```bash
docker inspect mycontainer
```

### Remove Volume

```bash
docker volume rm myvolume
```

**Remember:**

* **Container** = application runtime
* **Image** = template used to create containers
* **Named volume** = Docker-managed persistent storage
* **Bind mount** = host directory connected to a container
* **Persistence** = data survives container deletion
