
<h1>RUNBOOK for SSH & Nginx Services</h1>
 SSH & Nginx Service Health Check Runbook

This document outlines the steps used to verify the health and availability of the **SSH** and **Nginx** services on an Ubuntu-based AWS EC2 instance.

---

## 1. System Information

### Check Kernel Details

**Command**

```bash
uname -a
```

**Result**

```text
Linux ip-172-31-4-106 7.0.0-1004-aws #4-Ubuntu SMP PREEMPT Mon Apr 13 13:14:24 UTC 2026 x86_64 GNU/Linux
```

**Summary**

Verified that the instance is running an AWS Ubuntu kernel on a 64-bit architecture.

### Check Operating System

**Command**

```bash
cat /etc/os-release
```

**Summary**

Confirmed the server is running **Ubuntu 26.04 LTS**.

---

## 2. Verify File System

### Create a Test Directory

```bash
mkdir /tmp/runbook-demo
```

**Summary**

Successfully created a temporary directory for testing.

### Test File Operations

```bash
cp /etc/hosts /tmp/runbook-demo/hosts-copy
ls -l /tmp/runbook-demo
```

**Summary**

The file copy completed successfully, confirming the filesystem is writable.

---

## 3. CPU & Memory Status

### Monitor Running Processes

```bash
top
```

**Summary**

CPU usage remained low and the system was mostly idle.

### Check Top CPU Consumers

```bash
ps aux --sort=-%cpu | head
```

**Summary**

No unusual processes consuming excessive CPU resources.

### Check Memory Usage

```bash
ps aux --sort=-%mem | head
free -h
```

**Summary**

Memory utilization was within normal limits with sufficient free memory available.

---

## 4. Disk Usage

### Check Disk Space

```bash
df -h
```

**Summary**

The root partition had enough available space with approximately one-third utilized.

### Monitor Disk I/O

```bash
iostat
```

**Summary**

Disk activity appeared normal with no signs of I/O bottlenecks.

---

## 5. Network Verification

### Check Listening Services

```bash
ss -tulpn
```

**Summary**

Verified that SSH and Nginx were listening on their default ports.

### Verify SSH Port

```bash
sudo ss -tulpn | grep sshd
nc -zv localhost 22
```

**Summary**

SSH daemon was active, and port **22** was accepting connections.

### Verify Web Server

```bash
curl -I localhost
```

**Summary**

Nginx responded successfully with an **HTTP 200 OK** status.

---

## 6. Service Logs

### SSH Logs

```bash
sudo journalctl -u ssh -n 10
```

**Summary**

No recent SSH service issues were found. Authentication events appeared normal.

### Nginx Logs

```bash
sudo journalctl -u nginx -n 10
sudo tail -5 /var/log/nginx/access.log
```

**Summary**

Nginx started without errors, and recent requests were served successfully.

---

## 7. Validate Nginx Configuration

```bash
sudo nginx -t
```

**Summary**

Configuration syntax was successfully validated with no errors reported.

---

# Overall Health Check

✅ SSH service is running normally.

✅ Nginx is serving web requests successfully.

✅ CPU and memory usage are healthy.

✅ Disk utilization is within a safe range.

✅ Required network ports are accessible.

✅ Service logs do not indicate any critical issues.

✅ Nginx configuration validation completed successfully.

---

# Troubleshooting Steps

If any service becomes unavailable, perform the following checks:

### Review SSH authentication failures

```bash
sudo grep "Failed password" /var/log/auth.log
```

### Verify Nginx configuration

```bash
sudo nginx -t
```

### Inspect Nginx error logs

```bash

sudo tail -50 /var/log/nginx/error.log
```

### View active TCP connections

```bash
ss -ant
```

### Restart the required service

```bash
sudo systemctl restart nginx

sudo systemctl restart ssh
```
````


