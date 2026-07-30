# Day 19 – Shell Scripting Project: Log Rotation, Backup & Crontab

## Objective

The objective of this task was to automate routine server maintenance using shell scripts for log rotation, backups, and scheduled execution with cron.

---

# Task 1 – Log Rotation Script

## Description

The `log_rotate.sh` script:

- Accepts a log directory as an argument.
- Compresses `.log` files older than 7 days using `gzip`.
- Deletes compressed `.gz` files older than 30 days.
- Checks whether the provided directory exists before processing.

### Sample Command

```bash
./log_rotate.sh /tmp/myapp/logs
```

### Sample Output

```
Log rotation completed.
Compressed old log files.
Deleted old compressed logs.
```

---

# Task 2 – Server Backup Script

## Description

The `backup.sh` script:

- Accepts a source directory and backup destination.
- Creates a timestamped `.tar.gz` archive.
- Verifies that the archive was created successfully.
- Displays the archive name and size.
- Deletes backup files older than 14 days.

### Sample Command

```bash
./backup.sh /tmp/project-data /tmp/backups
```

### Sample Output

```
Backup created successfully.
Archive: backup-2026-07-30.tar.gz
Size: 4.0K
```

---

# Task 3 – Cron Scheduling

## Current Crontab

```bash
crontab -l
```

(No scheduled jobs were configured.)

### Cron Entries

Run log rotation every day at 2:00 AM:

```cron
0 2 * * * /root/90DaysOfDevOps/2026/day-19/log_rotate.sh /tmp/myapp/logs
```

Run backup every Sunday at 3:00 AM:

```cron
0 3 * * 0 /root/90DaysOfDevOps/2026/day-19/backup.sh /tmp/project-data /tmp/backups
```

Run a health check every 5 minutes:

```cron
*/5 * * * * /root/health_check.sh
```

---

# Task 4 – Maintenance Script

The `maintenance.sh` script combines log rotation and backup tasks, recording output in a log file with timestamps.

### Sample Command

```bash
./maintenance.sh
```

### Log Output

```
Running Log Rotation...
Running Backup...
Maintenance completed.
```

### Daily Cron Entry

```cron
0 1 * * * /root/90DaysOfDevOps/2026/day-19/maintenance.sh
```

---

# What I Learned

1. Shell scripts can automate repetitive maintenance tasks such as log rotation and backups.
2. Cron jobs allow scripts to run automatically at scheduled times without manual intervention.
3. Logging script output helps verify execution and simplifies troubleshooting.

---

# Conclusion

This project demonstrated how Bash scripting and cron scheduling can be combined to automate common system administration tasks, making server maintenance more reliable and efficient.
<img width="1920" height="1080" alt="Screenshot (305)" src="https://github.com/user-attachments/assets/f19c5c99-c9f3-4832-be6f-c343e2701a5c" />
<img width="1920" height="1080" alt="Screenshot (306)" src="https://github.com/user-attachments/assets/37088db2-6c11-4551-900d-8176a9cd123f" />
