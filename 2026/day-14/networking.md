# Day 14 – Networking Fundamentals & Hands-on Checks

## Objective

Learn the basics of networking by understanding the OSI and TCP/IP models and performing common networking troubleshooting commands on an AWS EC2 Ubuntu instance.

---

# OSI Model vs TCP/IP Model

## OSI Model

- Layer 1 – Physical: Transfers raw bits through cables or wireless.
- Layer 2 – Data Link: Handles MAC addresses and switching.
- Layer 3 – Network: Responsible for IP addressing and routing.
- Layer 4 – Transport: Uses TCP and UDP for communication.
- Layer 5 – Session: Manages communication sessions.
- Layer 6 – Presentation: Handles encryption and data formatting.
- Layer 7 – Application: User-facing protocols like HTTP and DNS.

## TCP/IP Model

- Link Layer
- Internet Layer
- Transport Layer
- Application Layer

---

# Protocol Mapping

| Protocol | Layer |
|-----------|-------|
| IP | Network / Internet |
| TCP | Transport |
| UDP | Transport |
| HTTP | Application |
| HTTPS | Application |
| DNS | Application |

---

# Real Example

```
curl https://google.com
```

Application Layer → HTTP/HTTPS

↓

Transport Layer → TCP

↓

Internet Layer → IP

↓

Link Layer → Ethernet

---

# Hands-on Commands

## Identity

```bash
hostname -I
```

Observation

- Displays the private IP address of the EC2 instance.

---

## Reachability

```bash
ping -c 4 google.com
```

Observation

- Internet connectivity verified.
- Low latency.
- No packet loss.

---

## Path

```bash
traceroute google.com
```

Observation

- Shows the network path from the EC2 instance to Google.
- Some routers may not reply, resulting in `* * *`.

---

## Listening Ports

```bash
ss -tulpn
```

Observation

- SSH service is listening on TCP port 22.

---

## DNS Resolution

```bash
dig google.com
```

Observation

- Successfully resolved the IP address of google.com.

---

## HTTP Check

```bash
curl -I https://google.com
```

Observation

- Received a valid HTTP response.

---

## Connections Snapshot

```bash
netstat -an | head
```

Observation

- Displays active TCP connections and listening ports.

---

# Mini Task – Port Probe

Listening Port

```
22 (SSH)
```

Test

```bash
nc -zv localhost 22
```

Result

- SSH port is reachable.

Next Check (if unreachable)

- Check SSH service status.
- Verify firewall/security group rules.
- Confirm the service is listening on the correct port.

---

# Reflection

## Which command gives the fastest signal when something is broken?

`ping` is the quickest way to determine whether a host is reachable.

## What layer would you inspect if DNS fails?

Application Layer (DNS service) and Internet Layer (IP connectivity).

## What if HTTP 500 appears?

Inspect the Application Layer and the web server logs.

## Two follow-up checks

1. Check service status using `systemctl status`.
2. Review application and system logs using `journalctl` or log files.

---

# What I Learned

- The OSI and TCP/IP models explain how network communication works.
- Networking tools like `ping`, `traceroute`, `curl`, and `ss` help identify connectivity and service issues.
- DNS resolution and HTTP status codes are essential for troubleshooting web applications.
