# Linux User & Group Management

## Overview

Today's hands-on practice focused on Linux user and group management. I learned how to create users, organize them into groups, configure shared directories, and manage file permissions. These are essential administrative tasks that help control access and collaboration in a multi-user Linux environment.

---

# Users Created

The following users were created with their own home directories and passwords:

- yono
- berlin
- professor


### Verification Commands

```bash
cat /etc/passwd | grep -E "ypno|berlin|professor"
ls -l /home
```

---

# Groups Created

To organize users based on their roles, I created these groups:

- developers
- admins
- project-team

### Verification Command

```bash
cat /etc/group | grep -E "developers|admins|project-team"
```

---

# Group Membership

The users were assigned to groups as follows:

| User | Group Membership |
|------|------------------|
| yono | developers, project-team |
| berlin | developers, admins |
| professor | admins |
| nairobi | project-team |

### Verification Command

```bash
groups yono
groups berlin
groups professor

```

---

# Shared Directories

## Development Project Directory

Directory created:

```text
/opt/dev-project
```

Configuration:

- Group Owner: developers
- Permissions: 775 (rwxrwxr-x)

### Verification

```bash
ls -ld /opt/dev-project
```

### Testing

Created test files as:

- yono
- berlin

using:

```bash
sudo -u yono touch /opt/dev-project/tokyo.txt
sudo -u berlin touch /opt/dev-project/berlin.txt
```

---


# Challenges Faced

- Initially, I needed to understand the difference between a user's primary group and supplementary groups.
- While testing directory access, I verified that the correct group ownership and permissions were applied before users could create files.
- Using verification commands like `groups` and `ls -ld` helped confirm that everything was configured correctly.

---

# Key Learnings

- Learned how to create and manage Linux users with dedicated home directories.
- Understood the purpose of groups in managing user permissions efficiently.
- Practiced assigning users to multiple groups using `usermod`.
- Configured shared directories with appropriate group ownership and permissions.
- Verified user access and permissions by creating files as different users.

---

# Why This Matters for DevOps

User and group management is a fundamental Linux administration skill. In DevOps environments, managing user access, securing shared resources, and applying the correct file permissions are important for maintaining secure and collaborative systems. Understanding these concepts helps administrators manage production servers more effectively.

---

# Screenshots Included

- User creation and home directories
- Group creation
- Group membership verification
- Shared directory permissions
- File creation tests by different users

  <img width="1920" height="1080" alt="Screenshot (285)" src="https://github.com/user-attachments/assets/b917d292-6f69-4a8d-99b6-20fd1801572b" />
