# Day 32 – Docker Volumes & Networking

## Task

Today's goal was to understand and practically implement:

* Data persistence using Docker volumes
* Bind mounts
* Docker networking
* Custom bridge networks
* Container-to-container communication
* Database and application communication using container names

---

# Task 1: The Problem – Data Does Not Persist Without a Volume

## Objective

To understand what happens to data when a container is removed without using a volume.

## Step 1: Run a MySQL Container

I started a MySQL container without attaching a volume.

```bash
docker run -d \
  --name mysql-no-volume \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  mysql:8
```

## Step 2: Check the Container

```bash
docker ps
```

The MySQL container was running successfully.

## Step 3: Enter MySQL

```bash
docker exec -it mysql-no-volume mysql -uroot -proot123
```

## Step 4: Create Test Data

Inside MySQL:

```sql
USE testdb;

CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

INSERT INTO students VALUES
(1, 'Komal'),
(2, 'Student2');

SELECT * FROM students;
```

The records were displayed successfully.

## Step 5: Exit MySQL

```sql
EXIT;
```

## Step 6: Stop and Remove the Container

```bash
docker stop mysql-no-volume
docker rm mysql-no-volume
```

## Step 7: Run a New Container

I created another MySQL container without using the previous container's storage.

```bash
docker run -d \
  --name mysql-new \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  mysql:8
```

After connecting to the new container and checking the database, the previous table and records were not available.

## Observation

The data was lost after the original container was removed.

## Why?

Containers are temporary/ephemeral. The writable layer belongs to the container. When the container is removed, the data stored only inside that container is removed as well.

### Conclusion

For important database data, persistent storage such as a Docker named volume should be used.

---

# Task 2: Named Volumes

## Objective

To use a Docker named volume so that database data remains available even after the container is removed.

## Step 1: Create a Named Volume

```bash
docker volume create mysql-data
```

## Step 2: Verify the Volume

```bash
docker volume ls
```

The volume `mysql-data` was displayed.

## Step 3: Inspect the Volume

```bash
docker volume inspect mysql-data
```

This displayed information about the volume, including its mount point.

## Step 4: Run MySQL with the Volume

```bash
docker run -d \
  --name mysql-volume \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  -v mysql-data:/var/lib/mysql \
  mysql:8
```

The volume was mounted to:

```text
/var/lib/mysql
```

This is the MySQL data directory.

## Step 5: Create Test Data

```bash
docker exec -it mysql-volume mysql -uroot -proot123
```

Inside MySQL:

```sql
USE testdb;

CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

INSERT INTO students VALUES
(1, 'Komal'),
(2, 'Student2');

SELECT * FROM students;
```

The data was displayed successfully.

Exit MySQL:

```sql
EXIT;
```

## Step 6: Remove the Container

```bash
docker stop mysql-volume
docker rm mysql-volume
```

The container was removed, but the named volume was not removed.

## Step 7: Create a New Container Using the Same Volume

```bash
docker run -d \
  --name mysql-volume-new \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  -v mysql-data:/var/lib/mysql \
  mysql:8
```

## Step 8: Verify the Data

```bash
docker exec -it mysql-volume-new mysql -uroot -proot123
```

Inside MySQL:

```sql
USE testdb;

SELECT * FROM students;
```

The previously created records were still available.

## Observation

The data remained available even though the original container was removed.

## Why?

The data was stored in the named volume instead of only inside the container's writable layer.

### Important Commands

```bash
docker volume ls
```

Lists Docker volumes.

```bash
docker volume inspect mysql-data
```

Displays detailed information about the volume.

### Conclusion

Named volumes provide persistent storage for containers and are especially useful for databases.

---

# Task 3: Bind Mounts

## Objective

To mount a folder from the host machine directly into a container.

## Step 1: Create a Host Folder

I created the following folder:

```text
2026/day-32/bind-mount-demo
```

Inside the folder, I created:

```text
index.html
```

## Step 2: Create Custom HTML

The `index.html` file contained a custom web page.

Example:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Bind Mount</title>
</head>
<body>
    <h1>Hello from Docker Bind Mount!</h1>
    <p>This page is coming from my host machine.</p>
</body>
</html>
```

## Step 3: Run Nginx with a Bind Mount

Because I was using Git Bash on Windows, I used the Windows path format.

```bash
MSYS_NO_PATHCONV=1 docker run -d \
  --name nginx-bind-mount \
  -p 8080:80 \
  -v "C:/Users/komsh/90DaysofDevops/2026/day-32/bind-mount-demo:/usr/share/nginx/html" \
  nginx
```

## Step 4: Check the Container

```bash
docker ps
```

The Nginx container was running.

## Step 5: Open the Page

I opened the following in my browser:

```text
http://localhost:8080
```

The custom HTML page was displayed.

## Step 6: Edit the Host File

I changed the contents of:

```text
index.html
```

Then I saved the file and refreshed:

```text
http://localhost:8080
```

The updated content appeared immediately.

## Observation

Changes made to the host `index.html` were reflected inside the Nginx container.

## Named Volume vs Bind Mount

### Named Volume

```text
Docker manages the storage location.
```

Example:

```bash
-v mysql-data:/var/lib/mysql
```

### Bind Mount

```text
I choose the exact folder on my host machine.
```

Example:

```bash
-v "C:/path/to/folder:/usr/share/nginx/html"
```

### Difference

| Named Volume                                           | Bind Mount                                    |
| ------------------------------------------------------ | --------------------------------------------- |
| Managed by Docker                                      | Managed by the user                           |
| Docker decides storage location                        | User specifies host path                      |
| Commonly used for persistent application/database data | Useful for sharing host files with containers |
| Example: database storage                              | Example: website development                  |

---

# Task 4: Docker Networking Basics

## Objective

To understand Docker's default networks and test container communication.

## Step 1: List Docker Networks

```bash
docker network ls
```

This displayed the available Docker networks.

Common default networks include:

```text
bridge
host
none
```

## Step 2: Inspect the Default Bridge Network

```bash
docker network inspect bridge
```

This displayed information about the default bridge network and connected containers.

## Step 3: Run Two Containers on the Default Bridge

Example:

```bash
docker run -d --name bridge-container1 alpine sleep 3600
```

```bash
docker run -d --name bridge-container2 alpine sleep 3600
```

## Step 4: Test Name-Based Communication

```bash
docker exec bridge-container1 ping -c 4 bridge-container2
```

On the default `bridge` network, container-name resolution is not provided in the same way as it is on a user-defined bridge network.

Therefore, name-based communication may fail.

## Step 5: Find the Container IP

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' bridge-container2
```

Example:

```text
172.17.0.3
```

## Step 6: Test IP-Based Communication

```bash
docker exec bridge-container1 ping -c 4 172.17.0.3
```

The containers were able to communicate using the IP address.

## Observation

On the default bridge network, IP-based communication works, but automatic container-name resolution is not available in the same way as it is on a custom bridge network.

### Conclusion

User-defined bridge networks are preferred when containers need to communicate with each other by name.

---

# Task 5: Custom Networks

## Objective

To create a custom bridge network and test name-based communication.

## Step 1: Create a Custom Network

```bash
docker network create my-app-net
```

## Step 2: Verify the Network

```bash
docker network ls
```

I verified that:

```text
my-app-net
```

was present.

## Step 3: Run Two Containers

```bash
docker run -d \
  --name app-container1 \
  --network my-app-net \
  alpine sleep 3600
```

```bash
docker run -d \
  --name app-container2 \
  --network my-app-net \
  alpine sleep 3600
```

## Step 4: Test Name-Based Communication

```bash
docker exec app-container1 ping -c 4 app-container2
```

The second container was successfully resolved by its name.

## Observation

The custom bridge network provided automatic DNS-based name resolution between containers.

### Why Does This Work?

User-defined bridge networks provide Docker's embedded DNS service.

This allows containers on the same custom network to communicate using their container names instead of requiring hard-coded IP addresses.

### Example

```text
app-container1
       |
       | ping app-container2
       |
       v
app-container2
```

### Conclusion

Custom bridge networks are useful for multi-container applications because containers can communicate using stable names rather than changing IP addresses.

---

# Task 6: Put It Together

## Objective

To combine Docker volumes and custom networking by creating:

* A custom network
* A MySQL database container
* A named volume
* An application container
* Communication between the application and database using the database container name

---

## Step 1: Create the Custom Network

```bash
docker network create task6-net
```

## Step 2: Verify the Network

```bash
docker network ls
```

I verified that:

```text
task6-net
```

was created successfully.

---

## Step 3: Create the Database Volume

```bash
docker volume create task6-db-data
```

## Step 4: Verify the Volume

```bash
docker volume ls
```

The following volume was displayed:

```text
task6-db-data
```

## Step 5: Inspect the Volume

```bash
docker volume inspect task6-db-data
```

This displayed the volume details and mount point.

---

## Step 6: Run MySQL on the Custom Network

```bash
docker run -d \
  --name task6-mysql \
  --network task6-net \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=task6db \
  -v task6-db-data:/var/lib/mysql \
  mysql:8
```

### What I configured

```text
Container Name: task6-mysql
Network: task6-net
Database: task6db
Volume: task6-db-data
MySQL Port: 3306
```

---

## Step 7: Verify MySQL Container

```bash
docker ps
```

The container was running:

```text
task6-mysql
```

---

## Step 8: Connect to MySQL

```bash
docker exec -it task6-mysql mysql -uroot -proot123
```

Inside MySQL:

```sql
USE task6db;
```

---

## Step 9: Create Test Data

```sql
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
```

Insert records:

```sql
INSERT INTO students VALUES
(1, 'Komal'),
(2, 'Student2');
```

Verify:

```sql
SELECT * FROM students;
```

The records were displayed successfully.

Exit MySQL:

```sql
EXIT;
```

---

## Step 10: Create the App Container

I used Alpine as the application/test container.

```bash
docker run -d \
  --name task6-app \
  --network task6-net \
  alpine \
  sleep 3600
```

## Step 11: Verify Both Containers

```bash
docker ps
```

Both containers were running:

```text
task6-app
task6-mysql
```

---

## Step 12: Enter the App Container

```bash
docker exec -it task6-app sh
```

The Alpine shell opened:

```text
/ #
```

---

## Step 13: Test Communication by Container Name

From inside `task6-app`:

```bash
ping -c 4 task6-mysql
```

The result showed:

```text
PING task6-mysql (172.19.0.2)
```

and:

```text
4 packets transmitted, 4 packets received, 0% packet loss
```

## Observation

The application container successfully resolved the MySQL container using its name:

```text
task6-mysql
```

This proves that Docker's custom network provided name-based communication.

---

## Step 14: Test MySQL Port

From inside the app container:

```bash
nc -zv task6-mysql 3306
```

The MySQL port was reachable.

This confirmed that the application container could reach the MySQL service on:

```text
task6-mysql:3306
```

---

## Step 15: Exit the App Container

```bash
exit
```

---

## Step 16: Final Container Verification

```bash
docker ps
```

Both containers were running:

```text
task6-app
task6-mysql
```

---

## Step 17: Inspect the Custom Network

```bash
docker network inspect task6-net
```

Both containers were listed under the network:

```text
task6-app
task6-mysql
```

---

## Step 18: Inspect the Database Volume

```bash
docker volume inspect task6-db-data
```

The volume details were displayed successfully.

---

# Final Architecture

The final Task 6 setup was:

```text
                         task6-net
                    Custom Bridge Network
                           |
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
       task6-app                  task6-mysql
       Alpine container            MySQL 8
             │                           │
             │                           ▼
             │                    task6-db-data
             │                    Named Volume
             │
             └──────► task6-mysql:3306
                  Name-based communication
```

---

# Key Learnings

## 1. Containers Are Ephemeral

Without persistent storage, data stored inside a container can disappear when the container is removed.

## 2. Named Volumes Provide Persistence

A named volume allows data to survive container deletion.

```bash
-v task6-db-data:/var/lib/mysql
```

## 3. Bind Mounts Connect Host Files to Containers

A bind mount allows a specific host directory to be mounted inside a container.

```bash
-v "C:/host/path:/container/path"
```

## 4. Custom Networks Enable Name-Based Communication

Containers connected to the same user-defined bridge network can communicate using their container names.

Example:

```bash
ping task6-mysql
```

## 5. Container Names Are Better Than Hard-Coded IP Addresses

Container IP addresses can change, but the container name remains the same.

Therefore, applications can connect using:

```text
task6-mysql:3306
```

instead of:

```text
172.19.0.2:3306
```

---

# Important Commands Learned

### Docker Volumes

```bash
docker volume create volume-name
docker volume ls
docker volume inspect volume-name
```

### Bind Mount

```bash
docker run -v /host/path:/container/path image
```

### Docker Networks

```bash
docker network ls
docker network create network-name
docker network inspect network-name
```

### Run Container on a Network

```bash
docker run --network network-name image
```

### Container Communication

```bash
docker exec container1 ping container2
```

### Inspect Running Containers

```bash
docker ps
docker ps -a
```

---

# Final Conclusion

Day 32 helped me understand two important Docker concepts: **persistent storage and container networking**.

I learned that containers are temporary, so important data should be stored in Docker volumes. I also learned that custom Docker networks allow containers to communicate with each other using container names.

In Task 6, I combined both concepts by running a MySQL database with a named volume and an Alpine application container on the same custom network. The application successfully reached MySQL using:

```text
task6-mysql:3306
```

This demonstrated how a simple multi-container application can be connected using Docker networking and persistent storage.

---

# Submission Checklist

* [x] Task 1 completed
* [x] Task 2 completed
* [x] Task 3 completed
* [x] Task 4 completed
* [x] Task 5 completed
* [x] Task 6 completed
* [ ] Add `day-32-volumes-networking.md` to `2026/day-32/`
* [ ] Add screenshots of experiments
* [ ] Commit changes
* [ ] Push to GitHub fork

# Day 32 Completed ✅

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`
