# Linux File Permissions & File Operations

## Overview

Today's practice focused on understanding Linux file operations and permission management. I created different types of files, explored ways to read file contents, modified file permissions using `chmod`, and tested how permissions affect file access and execution.

---

# Files Created

During this exercise, I created the following files:

- `devops.txt` – Empty file created using `touch`
- `notes.txt` – Text file containing Linux notes
- `script.sh` – Simple shell script to display a message

### Commands Used

```bash
touch devops.txt

cat > notes.txt

nano script.sh
```

---

# Reading File Contents

I practiced different commands to view file contents.

### Commands

```bash
cat notes.txt

vim -R script.sh

head -5 /etc/passwd

tail -5 /etc/passwd
```

### Purpose

- `cat` displays the complete file content.
- `vim -R` opens a file in read-only mode.
- `head` shows the beginning of a file.
- `tail` displays the last few lines of a file.

---

# Understanding File Permissions

I used the following command to check file permissions:

```bash
ls -l
```

### Initial Permissions

| File | Permissions |
|------|-------------|
| devops.txt | `-rw-rw-r--` |
| notes.txt | `-rw-rw-r--` |
| script.sh | `-rw-rw-r--` |

### Permission Breakdown

```
-rwxrwxrwx
```

- `r` = Read (4)
- `w` = Write (2)
- `x` = Execute (1)

Permissions are divided into three sections:

- Owner
- Group
- Others

---

# Permission Changes

## 1. Made the script executable

```bash
chmod +x script.sh
```

Verification:

```bash
ls -l script.sh
```

The execute (`x`) permission was added, allowing the script to run.

Run the script:

```bash
./script.sh
```

Output:

```text
Hello DevOps
```

---

## 2. Made `devops.txt` Read-Only

```bash
chmod a-w devops.txt
```

Verification:

```bash
ls -l devops.txt
```

The write permission was removed for all users.

---

## 3. Changed `notes.txt` Permission to 640

```bash
chmod 640 notes.txt
```

Permission Meaning:

- Owner → Read & Write
- Group → Read Only
- Others → No Access

Verification:

```bash
ls -l notes.txt
```

---

## 4. Created a Project Directory

```bash
mkdir project

chmod 755 project
```

Verification:

```bash
ls -ld project
```

Permission Meaning:

- Owner → Full access
- Group → Read & Execute
- Others → Read & Execute

---

# Permission Testing

### Read-Only File

Attempting to modify a file without write permission resulted in a permission-related error, demonstrating how Linux protects files from unauthorized changes.

### Executing Without Execute Permission

After removing the execute permission from `script.sh`, trying to run it produced a **Permission denied** error.

```bash
chmod -x script.sh

./script.sh
```

Output:

```text
Permission denied
```

---

# Commands Practiced

```bash
touch
cat
nano
vim -R
head
tail
ls -l
chmod
mkdir
```

---

# Key Learnings

- Learned how Linux uses file permissions to control access.
- Understood the difference between read, write, and execute permissions.
- Practiced modifying permissions using both symbolic and numeric methods.
- Explored different commands to create and read files.
- Learned why executable permission is required to run shell scripts.

---

# Why This Matters for DevOps

Managing file permissions is an essential Linux administration skill. Correct permissions help secure systems, protect sensitive files, and ensure scripts and applications run with the appropriate level of access. Understanding these concepts is important for managing production servers and automating deployments.

---


- Permission changes after using `chmod`
- Script execution before and after adding execute permission
- Project directory with `755` permissions
