# Day 30 – Docker Images & Container Lifecycle

## Task

Today's goal is to understand how Docker images and containers work.

We will learn:

* The relationship between images and containers
* Docker image layers and caching
* The complete container lifecycle
* How to work with running containers
* Docker cleanup and disk usage

---

# 1. Docker Images

## 1.1 Pull Nginx Image

```bash
docker pull nginx
```

This downloads the Nginx image from Docker Hub to the local machine.

Check the image:

```bash
docker images
```

Expected:

```text
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    xxxxxxxx       ...           ...
```

**Screenshot:** `docker images` showing the Nginx image.

---

## 1.2 Pull Ubuntu Image

```bash
docker pull ubuntu
```

Check:

```bash
docker images
```

The list should now contain:

```text
nginx
ubuntu
```

---

## 1.3 Pull Alpine Image

```bash
docker pull alpine
```

Check:

```bash
docker images
```

The list should now contain:

```text
nginx
ubuntu
alpine
```

---

## 1.4 Compare Ubuntu and Alpine

Run:

```bash
docker images
```

Look at the `SIZE` column.

Alpine is much smaller than Ubuntu because Alpine is designed to be a minimal Linux distribution with fewer packages and libraries.

### Comparison

```text
Ubuntu
More packages and libraries
        ↓
Larger image

Alpine
Minimal packages and libraries
        ↓
Smaller image
```

Image sizes can vary depending on the image version.

---

## 1.5 Inspect an Image

Run:

```bash
docker image inspect nginx
```

This displays detailed JSON information about the image.

Information includes:

* Image ID
* Architecture
* Operating system
* Creation time
* Environment variables
* Entrypoint
* Command
* Root filesystem
* Image layers

Check architecture:

```bash
docker image inspect nginx --format '{{.Architecture}}'
```

Check operating system:

```bash
docker image inspect nginx --format '{{.Os}}'
```

Check image layers:

```bash
docker image inspect nginx --format '{{json .RootFS.Layers}}'
```

**Screenshot:** `docker image inspect nginx`

---

## 1.6 Remove an Image

Remove Alpine:

```bash
docker rmi alpine
```

Verify:

```bash
docker images
```

Alpine should no longer appear.

### Important

Remove an image:

```bash
docker rmi <image>
```

Remove a container:

```bash
docker rm <container>
```

---

# 2. Docker Image Layers

## 2.1 View Image History

Run:

```bash
docker image history nginx
```

This shows the history of the Nginx image.

Look at the `SIZE` column.

Some entries may show an actual size while others may show:

```text
0B
```

**Screenshot:** `docker image history nginx`

---

## 2.2 What Are Docker Layers?

A Docker image is built from multiple layers.

Conceptually:

```text
Nginx configuration
-------------------
Nginx software
-------------------
Libraries
-------------------
Base filesystem
-------------------
Docker Image
```

Each layer represents changes made while building the image.

---

## 2.3 Why Does Docker Use Layers?

Docker uses layers mainly for:

* Reusing existing data
* Saving disk space
* Faster image downloads
* Faster image builds
* Caching

If two images share the same layers, Docker can reuse those layers instead of storing duplicate copies.

Example:

```text
Image A

Layer 1
Layer 2
Layer 3
Layer 4
```

```text
Image B

Layer 1
Layer 2
Layer 3
Layer 5
```

Layers 1, 2 and 3 can be shared.

---

## 2.4 Why Do Some Layers Show 0B?

Some Dockerfile instructions mainly change image metadata or configuration rather than adding filesystem data.

Examples include:

```dockerfile
CMD
ENTRYPOINT
ENV
```

Therefore, some history entries may show:

```text
0B
```

---

# 3. Container Lifecycle

We will practice the complete lifecycle using an Nginx container.

Lifecycle:

```text
CREATE
   ↓
START
   ↓
PAUSE
   ↓
UNPAUSE
   ↓
STOP
   ↓
RESTART
   ↓
KILL
   ↓
REMOVE
```

---

## 3.1 Create a Container Without Starting It

Run:

```bash
docker create --name lifecycle-nginx nginx
```

This creates the container but does not start it.

Check:

```bash
docker ps -a
```

The container should show:

```text
Created
```

**Screenshot:** Container in `Created` state.

---

## 3.2 Start the Container

Run:

```bash
docker start lifecycle-nginx
```

Check:

```bash
docker ps -a
```

The status should show:

```text
Up ...
```

The container is now running.

---

## 3.3 Pause the Container

Run:

```bash
docker pause lifecycle-nginx
```

Check:

```bash
docker ps -a
```

The status should show something similar to:

```text
Up ... (Paused)
```

**Screenshot:** Container in paused state.

---

## 3.4 Unpause the Container

Run:

```bash
docker unpause lifecycle-nginx
```

Check:

```bash
docker ps -a
```

The container should return to:

```text
Up ...
```

---

## 3.5 Stop the Container

Run:

```bash
docker stop lifecycle-nginx
```

Check:

```bash
docker ps -a
```

The container should show something similar to:

```text
Exited (0)
```

**Screenshot:** Container in stopped/exited state.

---

## 3.6 Restart the Container

Run:

```bash
docker restart lifecycle-nginx
```

Check:

```bash
docker ps -a
```

The container should be running again.

---

## 3.7 Kill the Container

Run:

```bash
docker kill lifecycle-nginx
```

Check:

```bash
docker ps -a
```

The container should now be stopped.

`docker kill` immediately terminates the container.

### Stop vs Kill

```text
docker stop
    ↓
Graceful stop

docker kill
    ↓
Immediate termination
```

---

## 3.8 Remove the Container

Run:

```bash
docker rm lifecycle-nginx
```

Check:

```bash
docker ps -a
```

The `lifecycle-nginx` container should no longer appear.

---

# 4. Working With Running Containers

## 4.1 Run Nginx in Detached Mode

Run:

```bash
docker run -d --name web-nginx -p 8080:80 nginx
```

Explanation:

```text
docker run
    ↓
Creates and starts a container

-d
    ↓
Detached/background mode

--name web-nginx
    ↓
Assigns the container name

-p 8080:80
    ↓
Maps host port 8080 to container port 80

nginx
    ↓
Uses the Nginx image
```

---

## 4.2 Check the Running Container

Run:

```bash
docker ps
```

You should see:

```text
web-nginx
```

and a port mapping similar to:

```text
0.0.0.0:8080->80/tcp
```

---

## 4.3 Open Nginx in the Browser

Open:

```text
http://localhost:8080
```

You should see the Nginx welcome page.

**Screenshot:** Nginx welcome page in the browser.

---

# 5. Container Logs

## 5.1 View Logs

Run:

```bash
docker logs web-nginx
```

Refresh:

```text
http://localhost:8080
```

Then run:

```bash
docker logs web-nginx
```

Nginx access information should appear.

---

## 5.2 Follow Logs in Real Time

Run:

```bash
docker logs -f web-nginx
```

Now refresh the Nginx page in the browser.

New log entries should appear automatically.

Press:

```text
Ctrl + C
```

to stop following the logs.

`Ctrl+C` here only exits log-follow mode. It does not remove the container.

---

# 6. Enter the Running Container

Run:

```bash
docker exec -it web-nginx bash
```

You are now inside the container.

Check the current directory:

```bash
pwd
```

List files:

```bash
ls
```

You should see directories such as:

```text
bin
dev
etc
home
lib
```

---

## 6.1 Explore Nginx Files

Go to the Nginx HTML directory:

```bash
cd /usr/share/nginx/html
```

List the files:

```bash
ls
```

You should see files such as:

```text
index.html
50x.html
```

View the Nginx HTML page:

```bash
cat index.html
```

---

## 6.2 Exit the Container

Run:

```bash
exit
```

You are now back in Git Bash.

Check:

```bash
docker ps
```

The Nginx container should still be running.

---

# 7. Run a Single Command Inside a Container

You don't always need to enter the container.

Run:

```bash
docker exec web-nginx ls /usr/share/nginx/html
```

This executes one command inside the container.

Another example:

```bash
docker exec web-nginx nginx -v
```

This displays the Nginx version.

---

# 8. Inspect the Container

Run:

```bash
docker inspect web-nginx
```

This displays detailed information about the container.

Information includes:

* Container ID
* Container name
* Image
* Network configuration
* IP address
* Port configuration
* Mounts
* Environment variables
* Container state
* Runtime information

---

## 8.1 Find the Container IP Address

Run:

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web-nginx
```

Example:

```text
172.17.0.2
```

The actual IP address may be different on your machine.

---

## 8.2 Check Port Mapping

Run:

```bash
docker port web-nginx
```

Expected format:

```text
80/tcp -> 0.0.0.0:8080
```

This means:

```text
Host
Port 8080
   ↓
Container
Port 80
```

---

## 8.3 Check Mounts

Run:

```bash
docker inspect -f '{{json .Mounts}}' web-nginx
```

If no volume was configured, the output may be:

```text
[]
```

This means the container has no additional mounts configured.

---

# 9. Cleanup

## 9.1 Check Running Containers

Run:

```bash
docker ps
```

---

## 9.2 Stop All Running Containers

Run:

```bash
docker stop $(docker ps -q)
```

Check:

```bash
docker ps
```

There should be no running containers.

Check all containers:

```bash
docker ps -a
```

---

## 9.3 Remove All Stopped Containers

Run:

```bash
docker container prune
```

Docker will ask for confirmation.

Type:

```text
y
```

and press Enter.

This removes stopped containers.

---

## 9.4 Remove Unused Images

Run:

```bash
docker image prune
```

Confirm with:

```text
y
```

This normally removes dangling/unused image data that Docker considers eligible for pruning.

---

## 9.5 Check Docker Disk Usage

Run:

```bash
docker system df
```

This shows Docker's disk usage for:

* Images
* Containers
* Local volumes
* Build cache

**Screenshot:** `docker system df`

---

# 10. Important Commands Summary

## Images

```bash
docker pull nginx
docker pull ubuntu
docker pull alpine

docker images

docker image inspect nginx

docker rmi alpine

docker image history nginx
```

## Container Lifecycle

```bash
docker create --name lifecycle-nginx nginx

docker start lifecycle-nginx

docker pause lifecycle-nginx

docker unpause lifecycle-nginx

docker stop lifecycle-nginx

docker restart lifecycle-nginx

docker kill lifecycle-nginx

docker rm lifecycle-nginx
```

Check status after each step:

```bash
docker ps -a
```

## Running Containers

```bash
docker run -d --name web-nginx -p 8080:80 nginx

docker ps

docker logs web-nginx

docker logs -f web-nginx

docker exec -it web-nginx bash

docker exec web-nginx ls /usr/share/nginx/html

docker inspect web-nginx

docker port web-nginx
```

## Cleanup

```bash
docker stop $(docker ps -q)

docker container prune

docker image prune

docker system df
```

---

# 11. Key Concepts Learned

## Image

An image is a read-only template used to create containers.

```text
Docker Image
     ↓
Container
```

## Container

A container is a runnable instance of an image.

One image can create multiple containers.

```text
             nginx image
             /    |    \
            /     |     \
     container1 container2 container3
```

## Image Layers

Images consist of multiple layers.

Layers allow Docker to:

* Reuse data
* Save disk space
* Build images faster
* Download images efficiently
* Use caching

## Container Lifecycle

```text
Created
   ↓
Running
   ↓
Paused
   ↓
Running
   ↓
Stopped
   ↓
Restarted
   ↓
Killed
   ↓
Removed
```

---

# 12. Screenshot Checklist

* [ ] `docker images` showing images
* [ ] Ubuntu vs Alpine image sizes
* [ ] `docker image inspect nginx`
* [ ] `docker image history nginx`
* [ ] Container in `Created` state
* [ ] Container in `Paused` state
* [ ] Container in `Exited` state
* [ ] Nginx container running with port mapping
* [ ] Nginx welcome page at `localhost:8080`
* [ ] Nginx container logs
* [ ] Inside the Nginx container
* [ ] Container inspection/IP information
* [ ] `docker system df`

---

# 13. Submission

Place this file in:

```text
2026/day-30/day-30-images.md
```

Then check:

```bash
git status
```

Add the file:

```bash
git add 2026/day-30/day-30-images.md
```

Commit:

```bash
git commit -m "Complete Day 30 Docker images and container lifecycle"
```

Push:

```bash
git push
```

---

# 14. Learn in Public

Things to mention in a LinkedIn post:

* Docker images are built from layers.
* Layers can be reused between images.
* Containers have a complete lifecycle.
* `docker create` creates without starting.
* `docker run` creates and starts.
* `docker exec` allows commands to be run inside a running container.
* `docker inspect` provides detailed container information.
* Docker cleanup commands help manage disk usage.

Hashtags:

```text
#90DaysOfDevOps
#DevOpsKaJosh
#TrainWithShubham
#Docker
#DevOps
#Containers
```
