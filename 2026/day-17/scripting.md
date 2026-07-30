# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

## Objective

The objective of this task was to learn shell scripting concepts such as loops, command-line arguments, package installation, and basic error handling.

---

# Task 1 – For Loop

## for_loop.sh

```bash
#!/bin/bash

FRUITS=("Apple" "Banana" "Orange" "Mango" "Grapes")

echo "List of Fruits:"

for fruit in "${FRUITS[@]}"
do
    echo "$fruit"
done
```

### Output

```
Apple
Banana
Orange
Mango
Grapes
```

---

## count.sh

```bash
#!/bin/bash

for i in {1..10}
do
    echo "$i"
done
```

### Output

```
1
2
3
4
5
6
7
8
9
10
```

---

# Task 2 – While Loop

## countdown.sh

```bash
#!/bin/bash

read -p "Enter a number: " NUM

while [ "$NUM" -ge 0 ]
do
    echo "$NUM"
    NUM=$((NUM-1))
done

echo "Done!"
```

### Example Output

```
Enter a number: 5

5
4
3
2
1
0
Done!
```

---

# Task 3 – Command-Line Arguments

## greet.sh

```bash
#!/bin/bash

if [ $# -eq 0 ]
then
    echo "Usage: ./greet.sh <name>"
else
    echo "Hello, $1!"
fi
```

### Example Output

```
Hello, Komal!
```

---

## args_demo.sh

```bash
#!/bin/bash

echo "Script Name: $0"
echo "Total Arguments: $#"
echo "Arguments: $@"
```

### Example Output

```
Script Name: ./args_demo.sh
Total Arguments: 3
Arguments: AWS Docker Linux
```

---

# Task 4 – Install Packages

## install_packages.sh

```bash
#!/bin/bash

if [ "$EUID" -ne 0 ]; then
    echo "Please run this script as root."
    exit 1
fi

PACKAGES=("nginx" "curl" "wget")

for pkg in "${PACKAGES[@]}"
do
    if dpkg -s "$pkg" &> /dev/null
    then
        echo "$pkg is already installed."
    else
        echo "Installing $pkg..."
        apt update
        apt install -y "$pkg"
    fi
done
```

### Output

```
nginx is already installed.
curl is already installed.
wget is already installed.
```

---

# Task 5 – Error Handling

## safe_script.sh

```bash
#!/bin/bash

set -e

mkdir /tmp/devops-test || echo "Directory already exists"

cd /tmp/devops-test || {
    echo "Unable to enter directory."
    exit 1
}

touch sample.txt

echo "Script completed successfully."
```

### Output

```
Script completed successfully.
```

---

# What I Learned

1. **For** and **while** loops automate repetitive tasks in shell scripts.
2. Command-line arguments (`$1`, `$#`, `$@`) make scripts reusable and flexible.
3. Error handling with `set -e`, `||`, and checking for root privileges makes scripts safer and more reliable.

---

# Conclusion

This exercise helped me understand how to use loops, process command-line arguments, automate package installation, and handle errors effectively in Bash scripts. These skills are essential for automating system administration and DevOps tasks.
<img width="1920" height="476" alt="Screenshot (300)" src="https://github.com/user-attachments/assets/9bfb5094-bf83-4dbf-81ac-7429b47d3c1e" />
<img width="1920" height="948" alt="Screenshot (301)" src="https://github.com/user-attachments/assets/5c645095-3b5b-401e-b0e6-bd9376ca3218" />
