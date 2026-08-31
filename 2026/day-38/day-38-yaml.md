# Day 38 – YAML Basics

## 90DaysOfDevOps

---

# 🎯 Day 38 Goal

The goal of Day 38 is to understand **YAML syntax and structure** before using YAML for CI/CD pipelines.

YAML is widely used in:

* GitHub Actions
* GitLab CI/CD
* Docker Compose
* Kubernetes
* Ansible
* CI/CD configuration
* Infrastructure and automation tools

The objective is not to memorize YAML.

The objective is to understand:

```text
Key
Value
List
Nested Object
Indentation
Boolean
Multi-line String
```

---

# 1. What is YAML?

YAML stands for:

> **YAML Ain't Markup Language**

YAML is a human-readable data serialization/configuration format.

It is commonly used for configuration files.

Example:

```yaml
name: Komal
role: DevOps Learner
experience_years: 1
learning: true
```

This is much easier for humans to read than many other configuration formats.

---

# 2. Why is YAML Important in DevOps?

YAML appears everywhere in DevOps.

Examples:

```text
Docker Compose
     ↓
docker-compose.yml

GitHub Actions
     ↓
.github/workflows/*.yml

Kubernetes
     ↓
deployment.yaml

Ansible
     ↓
playbook.yml
```

Therefore:

> Understanding YAML is essential before working with CI/CD and Kubernetes.

---

# 3. YAML File Extensions

Common extensions:

```text
.yaml
.yml
```

Both are valid YAML extensions.

Examples:

```text
person.yaml
server.yaml
docker-compose.yml
deployment.yaml
```

---

# 4. Basic YAML Structure

The simplest YAML structure is:

```yaml
key: value
```

Example:

```yaml
name: Komal
role: DevOps Learner
```

Here:

```text
name
 ↓
key

Komal
 ↓
value
```

Think:

```text
KEY : VALUE
```

---

# 5. Task 1 – Key-Value Pairs

Create:

```text
person.yaml
```

Example:

```yaml
name: Komal
role: DevOps Learner
experience_years: 1
learning: true
```

The values can have different data types.

---

# 6. YAML Data Types

## String

```yaml
name: Komal
```

`Komal` is a string.

---

## Number

```yaml
experience_years: 1
```

`1` is a number.

---

## Boolean

```yaml
learning: true
```

`true` is a boolean.

Valid boolean examples:

```yaml
learning: true
learning: false
```

Important:

```yaml
learning: true
```

is a boolean.

But:

```yaml
learning: "true"
```

is a string.

---

# 7. Quotes in YAML

Quotes are often optional.

This works:

```yaml
name: Komal
role: DevOps Learner
```

Quotes can be used when necessary:

```yaml
message: "Hello: DevOps"
```

Be careful with special characters.

For example:

```yaml
message: "Hello # DevOps"
```

Quotes can prevent special characters from being interpreted as YAML syntax.

---

# 8. Comments

Comments begin with:

```yaml
#
```

Example:

```yaml
# Person information
name: Komal
role: DevOps Learner
```

Everything after `#` is treated as a comment when used appropriately.

Comments are useful for explaining configuration.

---

# 9. YAML Indentation

Indentation is one of the most important YAML concepts.

Example:

```yaml
person:
  name: Komal
  role: DevOps Learner
```

Notice:

```text
person:
  name
  role
```

`name` and `role` are nested under `person`.

The spaces represent hierarchy.

---

# 10. YAML Uses Spaces

YAML uses spaces for indentation.

Use:

```text
2 spaces
```

as a common standard.

Example:

```yaml
server:
  name: web-server
  port: 5000
```

Do NOT use tabs.

Incorrect:

```text
server:
<TAB>name: web-server
```

Tabs can cause YAML parsing errors.

---

# 11. Why Indentation Matters

Consider:

```yaml
server:
  name: web-server
  port: 5000
```

This means:

```text
server
├── name
└── port
```

But:

```yaml
server:
name: web-server
port: 5000
```

has a completely different structure.

YAML uses indentation to understand relationships.

---

# 12. Task 2 – Lists

YAML supports lists.

The first common method is the **block list**.

Example:

```yaml
tools:
  - docker
  - kubernetes
  - git
  - linux
  - terraform
```

Each item begins with:

```text
-
```

---

# 13. Inline List

The second method is an inline/flow list.

Example:

```yaml
hobbies: [reading, football, coding]
```

This is useful when the list is short.

---

# 14. Two Ways to Write Lists

## Method 1 – Block Style

```yaml
tools:
  - docker
  - kubernetes
  - git
```

## Method 2 – Inline/Flow Style

```yaml
tools: [docker, kubernetes, git]
```

Remember:

```text
Block list
    ↓
Multiple lines

Inline list
    ↓
Single line
```

---

# 15. Example `person.yaml`

```yaml
name: Komal
role: DevOps Learner
experience_years: 1
learning: true

tools:
  - Docker
  - Git
  - GitHub
  - Linux
  - AWS

hobbies: [reading, coding, learning]
```

---

# 16. Task 3 – Nested Objects

YAML can represent objects inside other objects.

Example:

```yaml
server:
  name: web-server
  ip: 192.168.1.10
  port: 5000
```

Here:

```text
server
├── name
├── ip
└── port
```

---

# 17. Multiple Nested Levels

Example:

```yaml
database:
  host: db
  name: taskdb
  credentials:
    user: taskuser
    password: taskpassword
```

Structure:

```text
database
├── host
├── name
└── credentials
    ├── user
    └── password
```

Notice the indentation.

```yaml
database:
  credentials:
    user: taskuser
```

`user` is nested inside `credentials`, which is nested inside `database`.

---

# 18. Example `server.yaml`

```yaml
server:
  name: web-server
  ip: 192.168.1.10
  port: 5000

database:
  host: db
  name: taskdb
  credentials:
    user: taskuser
    password: taskpassword
```

---

# 19. YAML Tree Thinking

When reading YAML, visualize it as a tree.

Example:

```yaml
server:
  name: web-server
  port: 5000
```

Think:

```text
server
 ├── name
 └── port
```

Another example:

```yaml
database:
  credentials:
    user: taskuser
```

Think:

```text
database
 └── credentials
      └── user
```

This makes nested YAML much easier to understand.

---

# 20. Tabs vs Spaces

YAML should use spaces for indentation.

Correct:

```yaml
server:
  name: web-server
```

Incorrect:

```yaml
server:
	name: web-server
```

The second example contains a tab.

Depending on the parser/linter, this can produce an error such as:

```text
found character '\t' that cannot start any token
```

or another indentation/parsing error.

The exact wording depends on the YAML parser being used.

---

# 21. Task 4 – Multi-line Strings

YAML provides two important styles for multi-line text:

```text
|
>
```

---

# 22. Pipe `|` Style

Example:

```yaml
startup_script: |
  echo "Starting application"
  echo "Checking database"
  echo "Application started"
```

The `|` style preserves line breaks.

Conceptually:

```text
Line 1
Line 2
Line 3
```

remains:

```text
Line 1
Line 2
Line 3
```

Use `|` when the exact line structure matters.

Good examples:

* Shell scripts
* Configuration snippets
* Certificates
* Multi-line commands
* Text where line breaks are meaningful

---

# 23. Fold `>` Style

Example:

```yaml
startup_script: >
  echo "Starting application"
  echo "Checking database"
  echo "Application started"
```

The `>` style folds normal line breaks into spaces.

Conceptually:

```text
Line 1
Line 2
Line 3
```

becomes approximately:

```text
Line 1 Line 2 Line 3
```

Use `>` when the content should be treated more like a single paragraph or logical line.

---

# 24. `|` vs `>`

Remember:

```text
|  → preserve newlines

>  → fold newlines
```

Simple memory trick:

```text
| = lines stay separate

> = lines come together
```

---

# 25. Example Comparing `|` and `>`

```yaml
preserve: |
  First line
  Second line
  Third line

fold: >
  First line
  Second line
  Third line
```

Think:

```text
preserve
↓
First line
Second line
Third line
```

while:

```text
fold
↓
First line Second line Third line
```

---

# 26. Task 5 – YAML Validation

YAML is sensitive to:

* Indentation
* Spaces
* Colons
* List formatting
* Special characters
* Quotes

Therefore YAML should be validated before using it in automation.

---

# 27. Install yamllint

On systems where Python/pip is available:

```bash
pip install yamllint
```

Check installation:

```bash
yamllint --version
```

If `yamllint` is not available in your PATH, use an online YAML validator or a suitable editor extension.

---

# 28. Validate a YAML File

Example:

```bash
yamllint person.yaml
```

And:

```bash
yamllint server.yaml
```

A valid YAML file should pass YAML syntax checks, although a linter may also report style issues.

Important distinction:

```text
Parser validation
    ↓
Is the YAML structurally valid?

Linting
    ↓
Is the YAML also following recommended style rules?
```

---

# 29. Intentionally Break Indentation

Correct:

```yaml
server:
  name: web-server
  port: 5000
```

Break it:

```yaml
server:
  name: web-server
    port: 5000
```

Now `port` has incorrect indentation.

Run:

```bash
yamllint server.yaml
```

The validator should report an indentation/syntax problem.

Fix it:

```yaml
server:
  name: web-server
  port: 5000
```

Then validate again.

---

# 30. Task 6 – Spot the Difference

## Correct

```yaml
name: devops
tools:
  - docker
  - kubernetes
```

## Broken

```yaml
name: devops
tools:
- docker
  - kubernetes
```

The problem is inconsistent indentation of the list items.

Correct:

```yaml
tools:
  - docker
  - kubernetes
```

Both list items belong to `tools` and should be aligned.

---

# 31. YAML List Rule

Remember:

```yaml
tools:
  - docker
  - kubernetes
  - git
```

The hyphens should be aligned.

Think:

```text
tools
  ├── docker
  ├── kubernetes
  └── git
```

---

# 32. YAML vs JSON

YAML:

```yaml
name: Komal
tools:
  - Docker
  - Git
```

JSON:

```json
{
  "name": "Komal",
  "tools": [
    "Docker",
    "Git"
  ]
}
```

YAML is often easier for humans to write and read.

---

# 33. YAML Rules to Remember

### Rule 1 – Use spaces

```text
NO TABS
```

---

### Rule 2 – Indentation defines structure

```yaml
server:
  name: web-server
```

---

### Rule 3 – Key and value use `:`

```yaml
name: Komal
```

---

### Rule 4 – Lists use `-`

```yaml
tools:
  - Docker
  - Git
```

---

### Rule 5 – Boolean values

```yaml
learning: true
```

Not:

```yaml
learning: "true"
```

if you want an actual boolean.

---

### Rule 6 – Use quotes when needed

Especially when values contain characters that could be interpreted as YAML syntax.

---

### Rule 7 – Validate before using

```bash
yamllint file.yaml
```

---

# 34. Common YAML Mistakes

## Mistake 1 – Tabs

```yaml
server:
	name: web
```

Use spaces.

---

## Mistake 2 – Incorrect indentation

```yaml
server:
  name: web
    port: 5000
```

Correct:

```yaml
server:
  name: web
  port: 5000
```

---

## Mistake 3 – Missing colon

Incorrect:

```yaml
name Komal
```

Correct:

```yaml
name: Komal
```

---

## Mistake 4 – Misaligned lists

Incorrect:

```yaml
tools:
  - docker
    - git
```

Correct:

```yaml
tools:
  - docker
  - git
```

---

## Mistake 5 – Confusing boolean and string

Boolean:

```yaml
learning: true
```

String:

```yaml
learning: "true"
```

---

# 35. YAML Mental Model

When writing YAML, think in this order:

```text
1. What is the key?
        ↓
2. What is its value?
        ↓
3. Is the value a list?
        ↓
4. Is the value nested?
        ↓
5. Does indentation represent the relationship?
        ↓
6. Validate the YAML
```

---

# 36. Practical Workflow

When creating a YAML file:

```text
Create file
   ↓
Write key-value pairs
   ↓
Add lists
   ↓
Add nested objects
   ↓
Check indentation
   ↓
Validate
   ↓
Intentionally test an error
   ↓
Fix the error
   ↓
Validate again
```

---

# 37. YAML Files for Day 38

## `person.yaml`

Example:

```yaml
name: Komal
role: DevOps Learner
experience_years: 1
learning: true

tools:
  - Docker
  - Git
  - GitHub
  - Linux
  - AWS

hobbies: [coding, reading, learning]
```

---

## `server.yaml`

Example:

```yaml
server:
  name: web-server
  ip: 192.168.1.10
  port: 5000

database:
  host: db
  name: taskdb
  credentials:
    user: taskuser
    password: taskpassword

startup_script_pipe: |
  echo "Starting application"
  echo "Checking database"
  echo "Application started"

startup_script_fold: >
  echo "Starting application"
  echo "Checking database"
  echo "Application started"
```

---

# 38. Important Security Note

The `server.yaml` example contains:

```yaml
password: taskpassword
```

This is only an example for YAML practice.

Do not store real production passwords directly in a YAML file that will be committed to GitHub.

In real DevOps projects, secrets should be handled using appropriate secret-management mechanisms or protected environment configuration.

---

# 39. Useful YAML Commands

Display file:

```bash
cat person.yaml
```

Display server file:

```bash
cat server.yaml
```

Validate:

```bash
yamllint person.yaml
```

```bash
yamllint server.yaml
```

Check files:

```bash
ls
```

Check Git status:

```bash
git status
```

---

# 40. What Does YAML Have to Do With CI/CD?

CI/CD configuration is often written in YAML.

For example, a future GitHub Actions workflow may look conceptually like:

```yaml
name: CI

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run tests
        run: pytest
```

Notice the same concepts:

```text
key-value
   ↓
nested objects
   ↓
lists
   ↓
indentation
```

That is why Day 38 comes before CI/CD.

---

# 41. YAML and Docker Compose

You have already used YAML in your Docker project.

For example:

```yaml
services:
  web:
    image: myapp:v1
    ports:
      - "5050:5000"

  db:
    image: mysql:8.0
```

This is YAML.

Understanding YAML makes Docker Compose much easier to understand.

---

# 42. YAML and Kubernetes

Kubernetes manifests are also commonly written in YAML.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
```

Again:

```text
key-value
nested objects
lists
indentation
```

The YAML fundamentals remain the same.

---

# 43. Three Key Points

## 1️⃣ Indentation defines structure

YAML uses spaces to represent hierarchy.

```yaml
server:
  name: web-server
```

---

## 2️⃣ YAML supports different data structures

YAML can represent:

```text
Strings
Numbers
Booleans
Lists
Nested objects
Multi-line strings
```

---

## 3️⃣ Validation is essential

A small indentation mistake can break a configuration file.

Always validate before using YAML in automation.

---

# 44. Day 38 Checklist

* [ ] Create `person.yaml`
* [ ] Add key-value pairs
* [ ] Add a boolean
* [ ] Add 5 DevOps tools
* [ ] Add hobbies using inline list syntax
* [ ] Understand block vs inline lists
* [ ] Create `server.yaml`
* [ ] Add nested server configuration
* [ ] Add nested database credentials
* [ ] Test tabs vs spaces
* [ ] Add `|` multi-line string
* [ ] Add `>` folded string
* [ ] Understand `|` vs `>`
* [ ] Install/use `yamllint`
* [ ] Validate both YAML files
* [ ] Intentionally break indentation
* [ ] Observe the validation error
* [ ] Fix the YAML
* [ ] Validate again
* [ ] Commit the files
* [ ] Push to GitHub

---

# 45. Day 38 Command Cheat Sheet

```bash
# Create directory
mkdir day-38

# Enter directory
cd day-38

# Create files
touch person.yaml
touch server.yaml

# View files
cat person.yaml
cat server.yaml

# Validate
yamllint person.yaml
yamllint server.yaml

# Check files
ls

# Git status
git status

# Add files
git add .

# Commit
git commit -m "Complete Day 38 YAML basics"

# Push
git push
```

If `touch` is unavailable in your shell, create the files using:

```bash
notepad person.yaml
notepad server.yaml
```

---

# 46. How to Learn YAML Without Memorizing

Don't memorize YAML examples.

Instead, practice converting a requirement into a structure.

Requirement:

> "Create a server with a name and port."

Think:

```yaml
server:
  name: web-server
  port: 5000
```

Requirement:

> "The server has three tools."

Think:

```yaml
tools:
  - Docker
  - Git
  - Linux
```

Requirement:

> "The server has database credentials."

Think:

```yaml
database:
  credentials:
    user: admin
    password: password
```

The structure comes from the requirement.

---

# 47. YAML Memory Trick

Remember:

```text
:  → value

-  → list item

spaces → hierarchy

[] → inline list

|  → preserve newlines

>  → fold newlines

#  → comment
```

This small set of symbols covers a large part of everyday YAML.

---

# 48. Final YAML Mental Model

```text
                 YAML
                   |
        ┌──────────┼──────────┐
        ↓          ↓          ↓
      Values      Lists     Objects
        |           |          |
     key:value     - item    nested
        |
     Strings
     Numbers
     Boolean
```

And:

```text
Indentation
     ↓
Structure
     ↓
Validation
     ↓
Working Configuration
```

---

# 🚀 Final Day 38 Principle

> **YAML is not difficult because of syntax. YAML becomes difficult when indentation and structure are not understood.**

Before moving to CI/CD, make sure you can look at YAML and immediately identify:

```text
What is the key?
What is the value?
What is a list?
What is nested?
How deep is it nested?
Is this a boolean or string?
Are the spaces correct?
Will the YAML validate?
```

Once these become natural, GitHub Actions, Docker Compose, Kubernetes YAML, and other DevOps configuration files become much easier to read and write.

---

# 📌 Submission

The `2026/day-38/` directory should contain:

```text
2026/
└── day-38/
    ├── person.yaml
    ├── server.yaml
    └── day-38-yaml.md
```

Commit and push:

```bash
git add .
git commit -m "Complete Day 38 YAML basics"
git push
```

---

# 🔥 Day 38 Complete

**YAML Basics → Syntax → Lists → Nested Objects → Multi-line Strings → Validation → DevOps Applications**

Next step:

> **Day 39 – CI/CD Fundamentals**

The YAML knowledge from Day 38 will be used directly when creating the first CI/CD pipeline.
