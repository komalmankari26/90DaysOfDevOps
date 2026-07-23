

# Linux Command Notes

## Process & Service Commands



| Command                       | What it does                                                |
| ----------------------------- | ----------------------------------------------------------- |
| `ps -ef`                      | Lists all active processes with detailed information.       |
| `top`                         | Displays a live view of CPU, memory, and running processes. |
| `kill -9 <PID>`               | Immediately stops a process using its Process ID.           |
| `pkill <process_name>`        | Ends process(es) based on the process name.                 |
| `pgrep <process_name>`        | Returns the PID of a running process.                       |
| `systemctl status <service>`  | Shows whether a service is active or inactive.              |
| `systemctl restart <service>` | Stops and starts a service again.                           |

---

# File & Directory Utilities

Useful commands for navigating, searching, and managing files.

| Command                 | Purpose                                                    |
| ----------------------- | ---------------------------------------------------------- |
| `pwd`                   | Prints the current directory path.                         |
| `find`                  | Searches for files or folders based on different criteria. |
| `grep "text" file`      | Finds matching text inside a file.                         |
| `awk '{print $1}' file` | Prints the first column from a file.                       |
| `sed 's/old/new/' file` | Replaces matching text in a file.                          |
| `cut -d':' -f1 file`    | Extracts selected fields using a delimiter.                |
| `tail -f logfile.log`   | Continuously displays new log entries.                     |
| `wc -l file.txt`        | Counts the number of lines in a file.                      |
| `ln file link`          | Creates a hard link.                                       |
| `ln -s file link`       | Creates a symbolic (soft) link.                            |
| `rm -rf directory`      | Deletes a directory and everything inside it.              |
| `df -h`                 | Displays available and used disk space.                    |
| `du -sh directory`      | Shows the total size of a directory.                       |

---

# Network Commands

These commands are commonly used while troubleshooting network-related issues.

| Command                   | Purpose                                               |
| ------------------------- | ----------------------------------------------------- |
| `ping google.com`         | Checks if a remote host is reachable.                 |
| `ip addr show`            | Displays IP addresses and network interface details.  |
| `curl https://google.com` | Sends an HTTP request and returns the response.       |
| `dig google.com`          | Retrieves DNS information for a domain.               |
| `ss -tulpn`               | Lists active listening ports and associated services. |
| `hostnamectl`             | Displays hostname and operating system information.   |

---

# System Monitoring Commands

Commands to monitor the health and performance of a Linux system.

| Command                             | Purpose                                                             |
| ----------------------------------- | ------------------------------------------------------------------- |
| `free -h`                           | Shows memory and swap usage in a readable format.                   |
| `uptime`                            | Displays how long the system has been running and the load average. |
| `df -h \| awk 'NR==2 {print $5}'`   | Prints the disk usage percentage of the primary filesystem.         |
| `free -h \| awk 'NR==2 {print $3}'` | Displays the amount of RAM currently in use.                        |

---


