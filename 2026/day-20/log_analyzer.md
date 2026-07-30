# Day 20 – Bash Scripting Challenge: Log Analyzer and Report Generator

## Objective

The objective of this project was to automate log analysis using Bash scripting. The script validates user input, searches for errors and critical events, generates a summary report, and archives processed log files.

---

# Task 1 – Input Validation

The script accepts a log file as a command-line argument.

It performs the following validation:

- Checks whether a filename is provided.
- Verifies that the specified file exists.
- Displays an error message and exits if validation fails.

Example:

```bash
./log_analyzer.sh sample_log.log
```

---

# Task 2 – Error Count

The script searches the log file for entries containing:

- ERROR
- Failed

It counts the matching lines and displays the total number of errors.

Example Output

```
Total Errors : 18
```

---

# Task 3 – Critical Events

The script searches for log entries containing **CRITICAL** and displays them together with their line numbers.

Example

```
17: 2026-07-30 [CRITICAL]
42: 2026-07-30 [CRITICAL]
```

---

# Task 4 – Top 5 Error Messages

The script extracts ERROR entries, groups identical messages, counts their occurrences, and displays the five most common error messages.

Example

```
6 Disk full

5 Invalid input

4 Failed to connect

2 Out of memory
```

---

# Task 5 – Summary Report

A report file named

```
log_report_<date>.txt
```

is generated automatically.

The report contains:

- Date of analysis
- Log filename
- Total lines processed
- Total errors
- Top five error messages
- Critical events

---

# Task 6 – Archive Processed Logs

The script creates an **archive/** directory if it does not already exist and copies the processed log file into it.

---

# Commands Used

- grep
- wc
- sed
- sort
- uniq
- head
- cp
- mkdir
- date

---

# Sample Output

```
Log File : sample_log.log

Total Lines : 100

Total Errors : 18

Report Generated : log_report_2026-07-30.txt

Log archived successfully.
```

---

# What I Learned

1. Bash scripting can automate log analysis and reporting tasks efficiently.
2. Linux commands such as grep, sort, uniq, sed, and wc are useful for extracting and summarising log information.
3. Input validation and report generation improve script reliability and make troubleshooting easier.

---

# Conclusion

This project demonstrated how Bash scripting can automate daily system administration tasks by analysing logs, generating reports, and organising processed files. These techniques are valuable for monitoring and maintaining Linux servers in real-world DevOps environments.
<img width="1920" height="1024" alt="Screenshot (310)" src="https://github.com/user-attachments/assets/dd1c0bd6-707a-4988-841e-1b8258cc1927" />
<img width="1920" height="542" alt="Screenshot (309)" src="https://github.com/user-attachments/assets/97f7dd74-b078-4c43-bb5e-d6a0e7f365fe" />
<img width="1920" height="431" alt="Screenshot (308)" src="https://github.com/user-attachments/assets/9d1ef398-d58f-49d6-a078-502e71e65b02" />
<img width="1920" height="785" alt="Screenshot (307)" src="https://github.com/user-attachments/assets/09985ea2-f34c-450c-b511-a9d9df2432fc" />
<img width="1920" height="1011" alt="Screenshot (311)" src="https://github.com/user-attachments/assets/b413acb3-1d3d-43ff-bf3b-a5250719e4eb" />

