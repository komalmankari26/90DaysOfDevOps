# Day 11 – File Ownership Challenge

## Objective

The objective of this challenge was to understand Linux file ownership and group management using the `chown` and `chgrp` commands. I learned how to assign ownership to different users, change file groups, and apply ownership changes recursively to directories.

---

## Files & Directories Created

### Files

* `devops-file.txt`
* `team-notes.txt`
* `project-config.yaml`
* `heist-project/vault/gold.txt`
* `heist-project/plans/strategy.conf`
* `bank-heist/access-codes.txt`
* `bank-heist/blueprints.pdf`
* `bank-heist/escape-plan.txt`

### Directories

* `app-logs/`
* `heist-project/`
* `heist-project/vault/`
* `heist-project/plans/`
* `bank-heist/`

---

## Ownership Changes

| File/Directory        | Ownership Before | Ownership After                    |
| --------------------- | ---------------- | ---------------------------------- |
| `devops-file.txt`     | `ubuntu:ubuntu`  | `berlin:ubuntu`                    |
| `team-notes.txt`      | `ubuntu:ubuntu`  | `ubuntu:heist-team`                |
| `project-config.yaml` | `ubuntu:ubuntu`  | `professor:heist-team`             |
| `app-logs/`           | `ubuntu:ubuntu`  | `berlin:heist-team`                |
| `heist-project/`      | `ubuntu:ubuntu`  | `professor:planners` *(recursive)* |
| `access-codes.txt`    | `ubuntu:ubuntu`  | `tokyo:vault-team`                 |
| `blueprints.pdf`      | `ubuntu:ubuntu`  | `berlin:tech-team`                 |
| `escape-plan.txt`     | `ubuntu:ubuntu`  | `nairobi:vault-team`               |

---

## Commands Used

### View Ownership

```bash
ls -l
ls -ld directory-name
ls -lR directory-name
```

### Create Users

```bash
sudo useradd -m tokyo
sudo useradd -m berlin
sudo useradd -m professor
sudo useradd -m nairobi
```

### Create Groups

```bash
sudo groupadd heist-team
sudo groupadd planners
sudo groupadd vault-team
sudo groupadd tech-team
```

### Create Files & Directories

```bash
touch devops-file.txt
touch team-notes.txt
touch project-config.yaml

mkdir app-logs
mkdir -p heist-project/vault
mkdir -p heist-project/plans

touch heist-project/vault/gold.txt
touch heist-project/plans/strategy.conf

mkdir bank-heist

touch bank-heist/access-codes.txt
touch bank-heist/blueprints.pdf
touch bank-heist/escape-plan.txt
```

### Change Ownership

```bash
sudo chown tokyo devops-file.txt
sudo chown berlin devops-file.txt

sudo chgrp heist-team team-notes.txt

sudo chown professor:heist-team project-config.yaml
sudo chown berlin:heist-team app-logs

sudo chown -R professor:planners heist-project

sudo chown tokyo:vault-team bank-heist/access-codes.txt
sudo chown berlin:tech-team bank-heist/blueprints.pdf
sudo chown nairobi:vault-team bank-heist/escape-plan.txt
```

---

## Verification Commands

```bash
ls -l
ls -ld app-logs
ls -lR heist-project
ls -l bank-heist
```

---

## Key Learnings

* Every file and directory in Linux has an associated **owner** and **group**.
* The **owner** controls the file and can modify its permissions.
* The **group** allows multiple users to share access based on assigned permissions.
* The `chown` command changes the file owner and can also change the group simultaneously.
* The `chgrp` command is used to change only the group ownership.
* The `-R` (recursive) option applies ownership changes to a directory and all of its contents.
* Verifying ownership using `ls -l` after every change helps ensure the correct permissions are applied.

---

## Why File Ownership is Important in DevOps

Proper ownership management is essential for securing Linux systems and ensuring applications run with the correct permissions. It is commonly used for:

* Managing application deployment files
* Configuring shared project directories
* Securing log files
* Managing container volumes
* Controlling CI/CD pipeline artifacts
* Enforcing least-privilege access for users and services

---

## Conclusion

This challenge provided practical experience with Linux file ownership and group management. By creating users, assigning groups, modifying ownership, and applying recursive changes, I gained a better understanding of how Linux permissions are managed in real-world DevOps environments.
<img width="1920" height="729" alt="Screenshot (287)" src="https://github.com/user-attachments/assets/b5d8b6cf-f6c8-4abb-ac58-87fed5e3cd37" />
