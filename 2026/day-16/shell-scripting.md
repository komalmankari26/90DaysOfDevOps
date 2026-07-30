# Day 16 – Shell Scripting Basics

## Objective

The objective of this task was to learn the basics of shell scripting, including shebang, variables, user input, and if-else conditions.

---

# Task 1 – Hello Script

## Code

```bash
#!/bin/bash
echo "Hello, DevOps!"
```

### Commands

```bash
chmod +x hello.sh
./hello.sh
```

### Output

```
Hello, DevOps!
```

### Observation

The shebang (`#!/bin/bash`) tells Linux to execute the script using the Bash interpreter. Without it, the script may not run correctly if executed directly in a different shell.

---

# Task 2 – Variables

## Code

```bash
#!/bin/bash

NAME="Komal"
ROLE="DevOps Engineer"

echo "Hello, I am $NAME and I am a $ROLE"
echo 'Hello, I am $NAME and I am a $ROLE'
```

### Output

```
Hello, I am Komal and I am a DevOps Engineer
Hello, I am $NAME and I am a $ROLE
```

### Observation

- Double quotes expand variables.
- Single quotes print variables as plain text.

---

# Task 3 – User Input

## Code

```bash
#!/bin/bash

read -p "Enter your name: " NAME
read -p "Enter your favourite tool: " TOOL

echo "Hello $NAME, your favourite tool is $TOOL."
```

### Example Output

```
Enter your name: Komal
Enter your favourite tool: Docker

Hello Komal, your favourite tool is Docker.
```

---

# Task 4 – If-Else Conditions

## Number Check

```bash
#!/bin/bash

read -p "Enter a number: " NUM

if [ "$NUM" -gt 0 ]; then
    echo "The number is Positive."
elif [ "$NUM" -lt 0 ]; then
    echo "The number is Negative."
else
    echo "The number is Zero."
fi
```

## File Check

```bash
#!/bin/bash

read -p "Enter filename: " FILE

if [ -f "$FILE" ]; then
    echo "File exists."
else
    echo "File does not exist."
fi
```

---

# Task 5 – Server Check

## Code

```bash
#!/bin/bash

SERVICE="ssh"

read -p "Do you want to check the status of $SERVICE? (y/n): " CHOICE

if [ "$CHOICE" = "y" ]; then
    if systemctl is-active --quiet "$SERVICE"; then
        echo "$SERVICE is active."
    else
        echo "$SERVICE is not active."
    fi
else
    echo "Skipped."
fi
```

### Example Output

```
Do you want to check the status of ssh? (y/n): y

ssh is active.
```

---

# What I Learned

1. The shebang (`#!/bin/bash`) specifies the interpreter that executes the script.
2. Variables and `read` make scripts dynamic by allowing data storage and user input.
3. If-else statements help scripts make decisions based on conditions.

---

# Conclusion

This exercise introduced the fundamentals of shell scripting and demonstrated how to create interactive scripts using variables, user input, conditions, and executable permissions.
<img width="1920" height="1080" alt="Screenshot (296)" src="https://github.com/user-attachments/assets/e332c83f-332c-4eff-84fc-a80df626fc1a" />
<img width="1920" height="1080" alt="Screenshot (297)" src="https://github.com/user-attachments/assets/5ad95759-bfff-4fe9-a535-44a764496d3b" />
