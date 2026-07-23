

# Linux Architecture Notes

## Main Components of Linux

### 1. Hardware

Hardware refers to the physical devices that make up a computer system.

**Examples:**

* CPU (Processor)
* RAM (Memory)
* Hard Disk / SSD
* Network Interface Card (NIC)

---

### 2. Kernel

The **Linux Kernel** is the central part of the operating system. It connects software with hardware and manages system resources efficiently.

**Key Functions:**

* Process management
* Memory management
* Device communication
* File system management
* Resource allocation

---

### 3. User Space

User Space is the environment where users interact with applications. Programs run here without directly accessing the hardware.

**Examples:**

* Python applications
* Docker
* Web browsers
* Text editors

---

### 4. Shell

The **Shell** is a command interpreter that allows users to execute Linux commands and interact with the operating system.

**Popular Shells:**

* Bash
* Zsh
* Fish

---

### 5. systemd

**systemd** is the default initialization system in most modern Linux distributions. It is the first userspace process started after the kernel and is assigned **PID 1**.

**Responsibilities:**

* Starts essential services during boot
* Monitors and manages services
* Handles service failures
* Controls the boot process

---

# Linux Processes

A **process** is a program that is currently running on the system.

**Examples:**

* `nginx`
* `sshd`
* `docker`
* `python`

Each process is identified by a unique **Process ID (PID)**.

### Common Commands

```bash
ps aux
```

Displays information about active processes.

```bash
top
```

Provides a live view of running processes and system resource usage.

---

# Process States

### Running (R)

The process is actively being executed by the CPU.

### Sleeping (S)

The process is waiting for an event, input, or resource before continuing.

### Stopped (T)

The execution of the process has been temporarily halted.

### Zombie (Z)

The process has completed execution but remains listed until its parent process collects its exit status.

---

# About systemd

`systemd` acts as the service manager and initialization system for Linux.

It is responsible for:

* Launching services during startup
* Managing background services
* Restarting failed services when configured
* Organizing the boot sequence

**Example Command**

```bash
systemctl status sshd
```

This command displays the current status of the SSH service.

---

# Frequently Used Linux Commands

| Command     | Purpose                                        |
| ----------- | ---------------------------------------------- |
| `ps`        | Lists active processes                         |
| `top`       | Shows live CPU, memory, and process usage      |
| `systemctl` | Controls and manages system services           |
| `df -h`     | Displays disk space usage in a readable format |
| `free -h`   | Displays RAM and swap memory usage             |

This version is different enough from the original in wording, organization, and phrasing while preserving the same technical meaning, making it suitable to include in your own GitHub notes.
