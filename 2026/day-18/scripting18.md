# Day 18 – Shell Scripting: Functions & Intermediate Concepts

## Objective

The objective of this task was to learn Bash functions, strict mode, local variables, and create a reusable system information script.

---

# Task 1 – Basic Functions

## functions.sh

```bash
#!/bin/bash

greet() {
    echo "Hello, $1!"
}

add() {
    SUM=$(( $1 + $2 ))
    echo "Sum = $SUM"
}

greet "Komal"
add 15 25
```

### Output

```
Hello, Komal!
Sum = 40
```

---

# Task 2 – Disk & Memory Check

## disk_check.sh

```bash
#!/bin/bash

check_disk() {
    echo "===== Disk Usage ====="
    df -h /
}

check_memory() {
    echo
    echo "===== Memory Usage ====="
    free -h
}

check_disk
check_memory
```

### Output

Displays disk usage and available memory of the system.

---

# Task 3 – Strict Mode

## strict_demo.sh

```bash
#!/bin/bash

set -euo pipefail

echo "Strict mode enabled"

echo "$MYVAR"
```

### Explanation

- **set -e** → Stops the script immediately if any command fails.
- **set -u** → Stops the script when an undefined variable is used.
- **set -o pipefail** → Makes a pipeline fail if any command in the pipeline fails.

---

# Task 4 – Local Variables

## local_demo.sh

```bash
#!/bin/bash

show_local() {
    local NAME="Komal"
    echo "Inside: $NAME"
}

show_global() {
    CITY="Pune"
}

show_local
echo "$NAME"

show_global
echo "$CITY"
```

### Observation

Local variables are available only inside the function, while global variables remain accessible after the function call.

---

# Task 5 – System Information Reporter

## system_info.sh

This script displays:

- Hostname and OS information
- System uptime
- Disk usage
- Memory usage
- Top CPU-consuming processes

All tasks are organized into reusable functions and executed from a `main()` function.

---

# What I Learned

1. Functions make scripts modular, reusable, and easier to maintain.
2. `set -euo pipefail` improves script reliability by stopping execution on common errors.
3. Local variables help avoid conflicts with variables used elsewhere in the script.

---

# Conclusion

This exercise strengthened my understanding of writing structured Bash scripts using functions, applying strict mode for safer execution, and organizing code into reusable components for system administration tasks.
<img width="1920" height="1080" alt="Screenshot (303)" src="https://github.com/user-attachments/assets/3e4aecfc-62fa-4cbf-b3e3-2bbb9e835ca3" />
<img width="1920" height="1080" alt="Screenshot (302)" src="https://github.com/user-attachments/assets/417e785b-47b0-4a01-a91a-a361a4841780" />

