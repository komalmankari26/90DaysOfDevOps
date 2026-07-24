#  Cloud Server Setup: Docker, Nginx & Web Deployment

## Overview

Today's practice focused on setting up a cloud-based Ubuntu server and deploying a basic web server. I connected to my AWS EC2 instance through SSH, installed Docker and Nginx, configured the server for web access, and verified that the Nginx default webpage was accessible from the internet. I also explored how to collect and save Nginx logs for troubleshooting.

---

# Environment

- Cloud Platform: AWS EC2
- Operating System: Ubuntu
- Web Server: Nginx
- Container Platform: Docker

---

# Commands Used

### Connect to EC2

```bash
ssh -i my-key.pem ubuntu@<Public-IP>
```

### Update the Server

```bash
sudo apt update
sudo apt upgrade -y
```

### Install Docker

```bash
sudo apt install docker.io -y
docker --version
sudo systemctl enable docker
sudo systemctl start docker
systemctl status docker
```

### Install Nginx

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
systemctl status nginx
```

### Verify Web Server

```bash
curl -I localhost
```

### View Nginx Logs

```bash
sudo tail -20 /var/log/nginx/access.log
sudo tail -20 /var/log/nginx/error.log
```

### Save Logs

```bash
sudo cp /var/log/nginx/access.log ~/nginx-logs.txt
cat ~/nginx-logs.txt
```

---

# Deployment Verification

After installing Nginx, I updated the EC2 Security Group by allowing inbound HTTP traffic on **port 80**.

I then opened my EC2 **Public IPv4 address** in a web browser and confirmed that the default **Nginx Welcome Page** loaded successfully.

---

# Challenges Faced

- Initially, the webpage was not accessible because the HTTP (port 80) rule was missing from the Security Group.
- After adding the inbound rule and confirming that the Nginx service was running, the website became accessible.
- I also verified that Docker and Nginx services were active using `systemctl status`.

---

# What I Learned

- How to connect securely to a cloud server using SSH.
- The steps involved in installing and managing Docker and Nginx on Ubuntu.
- The importance of configuring Security Groups to allow web traffic.
- How to verify service health using `systemctl`.
- How to inspect and save Nginx log files for troubleshooting.

---

# Why This Is Important in DevOps

Deploying applications on cloud servers is a routine task for DevOps engineers. This exercise provided practical experience with server provisioning, service management, security configuration, and log analysis. These skills are essential for maintaining reliable and secure production environments.

---

# Files Created

- `cloud-deployment.md`
  

---

# Screenshots Included

- SSH connection to the EC2 instance
- Docker and Nginx service status
- Nginx welcome page displayed in the browser
 <img width="1920" height="1021" alt="Screenshot (278)" src="https://github.com/user-attachments/assets/85322b96-d710-4bdf-a047-fbf88e70d009" />
<img width="1920" height="990" alt="Screenshot (279)" src="https://github.com/user-attachments/assets/2e436c8d-718c-4db2-b47e-8be897573c35" />
<img width="1920" height="877" alt="Screenshot (280)" src="https://github.com/user-attachments/assets/4194153c-6856-414d-ab3b-78a1afc5a745" />
<img width="1920" height="1000" alt="Screenshot (283)" src="https://github.com/user-attachments/assets/4da14c96-1593-4e05-b888-83583d487f00" />


<img width="1920" height="990" alt="Screenshot (279)" src="https://github.com/user-attachments/assets/bcf4d038-3ef2-40c3-8287-d2ab95e419ba" />
<img width="1920" height="877" alt="Screenshot (280)" src="https://github.com/user-attachments/assets/5ceae172-0381-4d75-9701-9c8e29967b8f" />
