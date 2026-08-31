# Day 36 – Docker Project: Dockerize a Full Application

## 90DaysOfDevOps – Day 36

### Project: Flask Task Manager API + MySQL

---

## 1. Objective

The goal of Day 36 was to take a real application and Dockerize it end-to-end.

For this project, I chose a **Python Flask Task Manager API with MySQL**.

The application was containerized using Docker and Docker Compose, with:

* Flask application
* MySQL database
* Persistent Docker volume
* Custom Docker network
* Environment variables
* Database healthcheck
* Non-root Docker user
* Docker Hub image
* Fresh deployment test using the Docker Hub image

---

# 2. Project Structure

```text
task-manager/
│
├── app/
│   └── app.py
│
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .env
└── day-36-docker-project.md
```

> `.env` contains passwords and should NOT be committed to GitHub.

---

# 3. Application Architecture

```text
                 Client / Browser
                       |
                       |
                 localhost:5050
                       |
                       ↓
              ┌─────────────────┐
              │   Flask Web App │
              │    day36-web    │
              │    Port 5000    │
              └────────┬────────┘
                       |
                       | task-manager-network
                       |
                       ↓
              ┌─────────────────┐
              │    MySQL 8.0    │
              │   day36-mysql   │
              │    Port 3306    │
              └────────┬────────┘
                       |
                       ↓
              ┌─────────────────┐
              │   mysql_data    │
              │ Persistent Data │
              └─────────────────┘
```

---

# 4. Task 1 – Choose the Application

## Application Selected

**Flask Task Manager API + MySQL**

## Why I selected this application

I selected a Flask application because it provides practical experience with:

* Python application containerization
* Database connectivity
* Docker networking
* Docker volumes
* Environment variables
* Docker Compose
* Application deployment

This is closer to a real-world multi-container application than running a single standalone container.

---

# 5. Task 2 – Dockerfile

The application was containerized using a Python slim image.

## Dockerfile

```dockerfile
# Use a lightweight Python image
FROM python:3.11-slim

# Create a non-root user
RUN useradd -m appuser

# Set working directory
WORKDIR /app

# Copy dependency file first
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application source code
COPY app ./app

# Give ownership of application files to the non-root user
RUN chown -R appuser:appuser /app

# Switch to the non-root user
USER appuser

# Document the application port
EXPOSE 5000

# Start the Flask application
CMD ["python", "app/app.py"]
```

---

# 6. Dockerfile Explanation

| Dockerfile Instruction    | Purpose                                 |
| ------------------------- | --------------------------------------- |
| `FROM python:3.11-slim`   | Uses a lightweight Python base image    |
| `RUN useradd -m appuser`  | Creates a non-root user                 |
| `WORKDIR /app`            | Sets the application working directory  |
| `COPY requirements.txt .` | Copies Python dependencies              |
| `RUN pip install`         | Installs required Python packages       |
| `COPY app ./app`          | Copies application source code          |
| `RUN chown`               | Gives ownership to the non-root user    |
| `USER appuser`            | Runs the application as a non-root user |
| `EXPOSE 5000`             | Documents the Flask container port      |
| `CMD`                     | Starts the Flask application            |

---

# 7. `.dockerignore`

The `.dockerignore` file prevents unnecessary files from being copied into the Docker build context.

Example:

```text
venv/
__pycache__/
*.pyc
.env
.git/
.gitignore
```

This helps keep the build context smaller and prevents sensitive configuration from being included in the image.

---

# 8. Task 3 – Docker Compose

Docker Compose was used to run the Flask application and MySQL database together.

## Final Docker Compose Configuration

```yaml
services:

  web:
    image: komalmankari26/day36-task-manager:v1
    container_name: day36-web
    ports:
      - "5050:5000"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    networks:
      - task-manager-network

  db:
    image: mysql:8.0
    container_name: day36-mysql
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - task-manager-network

volumes:
  mysql_data:

networks:
  task-manager-network:
    driver: bridge
```

---

# 9. Docker Compose Explanation

## Web Service

The Flask application uses the Docker Hub image:

```yaml
image: komalmankari26/day36-task-manager:v1
```

Port mapping:

```yaml
ports:
  - "5050:5000"
```

This means:

```text
Host Port:      5050
Container Port: 5000
```

The application is accessed using:

```text
http://localhost:5050
```

---

# 10. MySQL Service

The database uses:

```yaml
image: mysql:8.0
```

The MySQL container is named:

```text
day36-mysql
```

The database uses a persistent volume:

```yaml
volumes:
  - mysql_data:/var/lib/mysql
```

---

# 11. Persistent Volume

A named Docker volume was created for MySQL:

```yaml
volumes:
  mysql_data:
```

It is mounted inside MySQL at:

```text
/var/lib/mysql
```

The volume was verified using:

```bash
docker inspect day36-mysql --format='{{json .Mounts}}'
```

The result confirmed:

```text
Type: volume
Name: task-manager_mysql_data
Destination: /var/lib/mysql
Driver: local
Mode: rw
```

This confirms that MySQL data is stored in a persistent Docker volume rather than only inside the container.

---

# 12. Persistence Test

The containers were stopped using:

```bash
docker compose down
```

The containers were started again using:

```bash
docker compose up -d
```

The status was checked:

```bash
docker compose ps
```

Result:

```text
day36-mysql   mysql:8.0               Up (healthy)
day36-web     day36-task-manager:v1   Up
```

The MySQL volume remained available because `docker compose down` was used without `-v`.

The volume was therefore successfully configured for persistence.

> Do not use `docker compose down -v` when you want to preserve the database volume.

---

# 13. Custom Docker Network

A custom bridge network was created:

```yaml
networks:
  task-manager-network:
    driver: bridge
```

Both services use this network:

```yaml
networks:
  - task-manager-network
```

This allows the Flask container to communicate with MySQL using the Compose service name:

```text
db
```

Therefore the Flask database host is:

```env
DB_HOST=db
```

Instead of:

```text
localhost
```

Inside a Docker container, `localhost` refers to that same container, not the MySQL container.

---

# 14. Database Healthcheck

MySQL was configured with a healthcheck:

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

The Flask application waits for MySQL to become healthy:

```yaml
depends_on:
  db:
    condition: service_healthy
```

This helps prevent the Flask application from starting before the database is ready.

---

# 15. Environment Variables

The project uses a `.env` file.

```env
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DATABASE=taskdb
MYSQL_USER=taskuser
MYSQL_PASSWORD=taskpassword

DB_HOST=db
DB_NAME=taskdb
DB_USER=taskuser
DB_PASSWORD=taskpassword
```

The `.env` file is supplied to both services:

```yaml
env_file:
  - .env
```

The environment variables were verified inside the containers.

The database container received the MySQL variables, and the Flask container received the `DB_` variables.

---

# 16. Security Note

The `.env` file contains database passwords.

Therefore:

```text
.env
```

should not be committed to GitHub.

It should be added to `.gitignore`:

```text
.env
venv/
__pycache__/
```

For a production application, secrets should preferably be managed using a dedicated secrets-management solution rather than storing passwords directly in a local `.env` file.

---

# 17. Testing the Flask Application

The containers were started using:

```bash
docker compose up -d
```

Container status was checked using:

```bash
docker compose ps
```

The Flask application was tested using:

```bash
curl http://localhost:5050
```

The application successfully returned:

```json
{
  "message": "Task Manager API is running!",
  "status": "success"
}
```

This confirmed that the Flask container was running correctly.

---

# 18. Database Endpoint Test

The following command was also tested:

```bash
curl http://localhost:5050/db
```

It returned:

```text
404 Not Found
```

This was not a Docker or MySQL error.

The current Flask application does not define a `/db` route, so Flask correctly returned HTTP 404.

The main application endpoint was successfully tested and returned the expected response.

---

# 19. Docker Image

The application image was:

```text
day36-task-manager:v1
```

The Docker image ID was:

```text
e4ef33499a42
```

The locally displayed image size was approximately:

```text
83 MB
```

The image was tagged for Docker Hub as:

```text
komalmankari26/day36-task-manager:v1
```

---

# 20. Docker Hub

The image was successfully pushed to Docker Hub.

## Tag Command

```bash
docker tag day36-task-manager:v1 komalmankari26/day36-task-manager:v1
```

## Push Command

```bash
docker push komalmankari26/day36-task-manager:v1
```

The push completed successfully.

Docker Hub image:

```text
komalmankari26/day36-task-manager:v1
```

---

# 21. Docker Hub Verification

The image was pulled from Docker Hub using:

```bash
docker pull komalmankari26/day36-task-manager:v1
```

Docker returned:

```text
Status: Image is up to date for komalmankari26/day36-task-manager:v1
```

The image digest was:

```text
sha256:e4ef33499a42d99e54b3722629ee9eaef2662ffd400cf8a3087cf9701500b2ce
```

This confirmed that the published Docker Hub image was available successfully.

---

# 22. Task 5 – Fresh Deployment Test

The final test was performed to verify that the project could run using the Docker Hub image instead of rebuilding the application locally.

## Step 1 – Stop containers

```bash
docker compose down
```

The MySQL volume was intentionally preserved.

## Step 2 – Remove local application image

The local application image was removed.

## Step 3 – Use Docker Hub image

The Compose configuration was changed to:

```yaml
web:
  image: komalmankari26/day36-task-manager:v1
```

The local `build:` instruction was removed for the final deployment test.

## Step 4 – Pull the image

```bash
docker compose pull
```

The application image was downloaded from Docker Hub.

## Step 5 – Start the application

```bash
docker compose up -d
```

## Step 6 – Verify containers

```bash
docker compose ps
```

MySQL started successfully and reported:

```text
healthy
```

The Flask application was also running.

## Step 7 – Test the application

```bash
curl http://localhost:5050
```

Response:

```json
{
  "message": "Task Manager API is running!",
  "status": "success"
}
```

The fresh deployment test was successful.

---

# 23. Challenges Faced

## Challenge 1 – Port Conflict

Port 5000 was already being used by another application during development.

The Flask container continued to listen on port 5000, but the host port was changed to 5050:

```yaml
ports:
  - "5050:5000"
```

The application was then accessed using:

```text
http://localhost:5050
```

---

## Challenge 2 – Docker Volume in Use

An attempt was made to remove the MySQL volume while the container was still using it.

Docker returned a volume-in-use error.

The reason was that the MySQL container was still attached to the volume.

The container was stopped before performing volume operations.

The volume was then retained for the persistence requirement.

---

## Challenge 3 – Database Startup

The Flask application depends on MySQL.

MySQL can take some time to initialize when the container starts.

A MySQL healthcheck was therefore configured:

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
```

The Flask service was configured to wait for the database:

```yaml
depends_on:
  db:
    condition: service_healthy
```

---

## Challenge 4 – Docker Compose Build vs Docker Hub Pull

Initially the Compose file used:

```yaml
build: .
```

For the final fresh deployment test, this was changed to:

```yaml
image: komalmankari26/day36-task-manager:v1
```

This ensured that Docker Compose pulled the application image from Docker Hub instead of rebuilding it locally.

---

# 24. Useful Docker Commands Used

## Build image

```bash
docker build -t day36-task-manager:v1 .
```

## Run Compose

```bash
docker compose up -d
```

## Stop Compose

```bash
docker compose down
```

## Check containers

```bash
docker compose ps
```

## View logs

```bash
docker compose logs
```

## View images

```bash
docker images
```

## Inspect volume

```bash
docker volume inspect task-manager_mysql_data
```

## Inspect container mounts

```bash
docker inspect day36-mysql --format='{{json .Mounts}}'
```

## Tag image

```bash
docker tag day36-task-manager:v1 komalmankari26/day36-task-manager:v1
```

## Push image

```bash
docker push komalmankari26/day36-task-manager:v1
```

## Pull image

```bash
docker pull komalmankari26/day36-task-manager:v1
```

## Test application

```bash
curl http://localhost:5050
```

---

# 25. Final Project Requirements

| Requirement                     | Status |
| ------------------------------- | ------ |
| Real application selected       | ✅      |
| Flask application               | ✅      |
| Dockerfile                      | ✅      |
| Lightweight Python image        | ✅      |
| Non-root user                   | ✅      |
| `.dockerignore`                 | ✅      |
| Docker Compose                  | ✅      |
| MySQL database                  | ✅      |
| Persistent volume               | ✅      |
| Custom network                  | ✅      |
| Environment variables           | ✅      |
| `.env` configuration            | ✅      |
| Database healthcheck            | ✅      |
| Docker Hub image                | ✅      |
| Image pushed successfully       | ✅      |
| Fresh Docker Hub pull           | ✅      |
| Fresh deployment test           | ✅      |
| Application tested successfully | ✅      |

---

# 26. Final Docker Hub Image

```text
komalmankari26/day36-task-manager:v1
```

Docker Hub repository:

```text
https://hub.docker.com/r/komalmankari26/day36-task-manager
```

---

# 27. How to Run the Project

Make sure the `.env` file exists with the required database configuration.

Then run:

```bash
docker compose pull
```

Start the application:

```bash
docker compose up -d
```

Check the services:

```bash
docker compose ps
```

Open:

```text
http://localhost:5050
```

Or test using:

```bash
curl http://localhost:5050
```

Expected response:

```json
{
  "message": "Task Manager API is running!",
  "status": "success"
}
```

To stop the application:

```bash
docker compose down
```

---

# 28. Key Learnings

Through this Day 36 project, I learned:

* How to Dockerize a Flask application
* How to create a Dockerfile
* How to use a lightweight Python base image
* How to run containers as a non-root user
* How to use Docker Compose
* How to run Flask and MySQL together
* How Docker networking works
* How containers communicate using service names
* How to use environment variables
* How to use `.env` files
* How to persist database data with Docker volumes
* How to configure MySQL healthchecks
* How to use `depends_on` with health conditions
* How to tag Docker images
* How to push images to Docker Hub
* How to pull images from Docker Hub
* How to perform a fresh deployment test
* How to troubleshoot Docker port and volume issues

---

# 29. Final Status

## 🚀 Day 36 – Dockerize a Full Application: COMPLETED

The Flask Task Manager API was successfully Dockerized with MySQL.

The application was:

1. Containerized
2. Connected to MySQL
3. Configured with persistent storage
4. Configured with a custom Docker network
5. Configured with environment variables
6. Protected with a database healthcheck
7. Published to Docker Hub
8. Pulled from Docker Hub
9. Tested successfully after fresh deployment

### Docker Hub Image

```text
komalmankari26/day36-task-manager:v1
```

### Final Application Test

```bash
curl http://localhost:5050
```

Result:

```json
{
  "message": "Task Manager API is running!",
  "status": "success"
}
```

**Day 36 successfully completed. 🎯**

#90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham
