# Day 15 – Networking Concepts: DNS, IP, Subnets & Ports

## Objective

The objective of this task is to understand the core networking concepts that every DevOps engineer should know, including DNS, IP addressing, subnetting, CIDR notation, and network ports.

---

# Task 1: DNS – How Names Become IPs

## 1. What happens when you type `google.com` in a browser?

- The browser first checks if the IP address is already cached.
- If not, it sends a DNS request to a DNS server.
- The DNS server looks up the IP address associated with `google.com`.
- Once the IP address is returned, the browser establishes a connection to that server and loads the website.

### DNS Record Types

| Record | Description |
|---------|-------------|
| A | Maps a domain name to an IPv4 address. |
| AAAA | Maps a domain name to an IPv6 address. |
| CNAME | Points one domain name to another domain name. |
| MX | Specifies the mail server responsible for receiving emails. |
| NS | Identifies the authoritative name servers for a domain. |

### Command

```bash
dig google.com
```

### Observation

- **A Record:** (Copy the IP shown in your output)
- **TTL:** (Copy the TTL value from your output)

---

# Task 2: IP Addressing

## What is an IPv4 Address?

An IPv4 address is a unique 32-bit address assigned to a device on a network. It consists of four numbers separated by dots, for example:

```
192.168.1.10
```

Each number ranges from 0 to 255.

---

## Public vs Private IP Address

| Public IP | Private IP |
|------------|------------|
| Accessible over the Internet | Used only within private networks |
| Assigned by Internet Service Provider | Assigned by local router or cloud network |
| Example: 8.8.8.8 | Example: 172.31.15.20 |

---

## Private IP Ranges

- 10.0.0.0 – 10.255.255.255
- 172.16.0.0 – 172.31.255.255
- 192.168.0.0 – 192.168.255.255

---

## Command

```bash
ip addr show
```

### Observation

My AWS EC2 instance has the private IP:

```
(Write your IP here, for example: 172.31.x.x)
```

This belongs to the private IP range `172.16.0.0 – 172.31.255.255`.

---

# Task 3: CIDR & Subnetting

## What does /24 mean?

`/24` means the first 24 bits are used for the network portion of the IP address, while the remaining 8 bits are available for host addresses.

---

## Usable Hosts

| CIDR | Usable Hosts |
|------|--------------|
| /24 | 254 |
| /16 | 65,534 |
| /28 | 14 |

---

## Why do we subnet?

Subnetting divides a large network into smaller networks. This improves network performance, enhances security, and helps manage IP addresses efficiently.

---

## CIDR Table

| CIDR | Subnet Mask | Total IPs | Usable Hosts |
|------|-------------|-----------|--------------|
| /24 | 255.255.255.0 | 256 | 254 |
| /16 | 255.255.0.0 | 65,536 | 65,534 |
| /28 | 255.255.255.240 | 16 | 14 |

---

# Task 4: Ports – The Doors to Services

## What is a Port?

A port is a logical communication endpoint used by applications and services to send and receive network traffic. Ports allow multiple services to run on the same IP address without interfering with each other.

---

## Common Ports

| Port | Service |
|------|---------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 53 | DNS |
| 3306 | MySQL |
| 6379 | Redis |
| 27017 | MongoDB |

---

## Command

```bash
ss -tulpn
```

### Observation

Example:

| Port | Service |
|------|---------|
| 22 | SSH |
| 68 | DHCP Client *(or another service shown on your EC2 instance)* |

*(Replace the second entry with one that actually appears in your `ss -tulpn` output.)*

---

# Task 5: Putting It Together

## 1. What happens when you run?

```bash
curl http://myapp.com:8080
```

The DNS server resolves the domain name into an IP address. The client then establishes a TCP connection to port 8080 of the server using the IP address. Finally, an HTTP request is sent, and the server returns a response.

---

## 2. Your application cannot reach a database at `10.0.1.50:3306`.

I would first verify that the database server is running and listening on port 3306. Next, I would check network connectivity, firewall rules, security groups, routing, and DNS (if a hostname is being used).

---

# Commands Executed

```bash
dig google.com

ip addr show

ss -tulpn
```

---

# What I Learned

1. DNS converts domain names into IP addresses so that computers can communicate over networks.
2. CIDR notation helps divide networks into smaller subnets, improving efficiency and IP address management.
3. Ports allow different applications and services, such as SSH, HTTP, and MySQL, to communicate over the same IP address.

---

# Conclusion

This exercise improved my understanding of networking fundamentals such as DNS resolution, IP addressing, subnetting, CIDR notation, and commonly used ports. These concepts are essential for troubleshooting and managing cloud infrastructure in DevOps environments.
