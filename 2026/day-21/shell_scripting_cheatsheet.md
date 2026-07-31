# Day 21 – Shell Scripting Cheat Sheet

A quick reference guide for Bash Shell Scripting.

---

# Quick Reference Table

| Topic | Key Syntax | Example |
|--------|------------|---------|
| Variable | `VAR="value"` | `NAME="DevOps"` |
| Argument | `$1`, `$2` | `./script.sh John` |
| If | `if [ condition ]; then` | `if [ -f file ]; then` |
| For Loop | `for i in list; do` | `for i in 1 2 3; do` |
| While Loop | `while [ condition ]` | `while [ $n -gt 0 ]` |
| Function | `name(){}` | `greet(){ echo Hi; }` |
| Grep | `grep pattern file` | `grep -i error app.log` |
| Awk | `awk '{print $1}' file` | `awk -F: '{print $1}' /etc/passwd` |
| Sed | `sed 's/old/new/g' file` | `sed -i 's/foo/bar/g' file.txt` |

---

# 1. Basics

## Shebang

Specifies the interpreter used to execute the script.

```bash
#!/bin/bash
```

---

## Run a Script

```bash
chmod +x script.sh
./script.sh
```

or

```bash
bash script.sh
```

---

## Comments

```bash
# Single-line comment

echo "Hello" # Inline comment
```

---

## Variables

```bash
NAME="DevOps"

echo $NAME

echo "$NAME"

echo '$NAME'
```

Difference:

- Double quotes expand variables.
- Single quotes treat variables as plain text.

---

## User Input

```bash
read -p "Enter Name: " NAME

echo "Hello $NAME"
```

---

## Command Line Arguments

| Variable | Meaning |
|----------|----------|
| `$0` | Script name |
| `$1` | First argument |
| `$2` | Second argument |
| `$#` | Number of arguments |
| `$@` | All arguments |
| `$?` | Exit status of previous command |

Example

```bash
./script.sh DevOps AWS
```

---

# 2. Operators & Conditionals

## String Comparison

```bash
[ "$A" = "$B" ]

[ "$A" != "$B" ]

[ -z "$VAR" ]

[ -n "$VAR" ]
```

---

## Integer Comparison

```bash
-eq
-ne
-lt
-gt
-le
-ge
```

Example

```bash
if [ $A -gt $B ]
```

---

## File Test Operators

| Operator | Description |
|----------|-------------|
| -f | File exists |
| -d | Directory exists |
| -e | Exists |
| -r | Readable |
| -w | Writable |
| -x | Executable |
| -s | Not empty |

---

## If Else

```bash
if [ condition ]
then
    echo "True"

elif [ condition ]
then
    echo "Else If"

else
    echo "False"
fi
```

---

## Logical Operators

```bash
&&

||

!
```

Example

```bash
[ -f file ] && echo "Exists"
```

---

## Case Statement

```bash
case $choice in

1)
echo "One"
;;

2)
echo "Two"
;;

*)
echo "Invalid"
;;

esac
```

---

# 3. Loops

## For Loop

```bash
for fruit in Apple Mango Orange
do
    echo $fruit
done
```

---

## C Style For Loop

```bash
for ((i=1;i<=5;i++))
do
echo $i
done
```

---

## While Loop

```bash
count=1

while [ $count -le 5 ]
do
echo $count
((count++))
done
```

---

## Until Loop

```bash
count=1

until [ $count -gt 5 ]
do
echo $count
((count++))
done
```

---

## Break & Continue

```bash
break

continue
```

---

## Loop Through Files

```bash
for file in *.log
do
echo $file
done
```

---

## Read File Line by Line

```bash
while read line
do
echo $line
done < file.txt
```

---

# 4. Functions

## Define Function

```bash
greet(){

echo "Hello"

}
```

---

## Call Function

```bash
greet
```

---

## Function Arguments

```bash
greet(){

echo "Hello $1"

}

greet DevOps
```

---

## Return

```bash
return 0
```

Return strings using

```bash
echo
```

---

## Local Variables

```bash
demo(){

local NAME="Linux"

}
```

---

# 5. Text Processing Commands

## grep

```bash
grep error log.txt

grep -i error log.txt

grep -r error .

grep -n error log.txt

grep -c error log.txt

grep -v error log.txt

grep -E "error|failed" log.txt
```

---

## awk

```bash
awk '{print $1}' file

awk -F: '{print $1}' /etc/passwd

awk 'BEGIN{print "Start"}'

awk 'END{print "Done"}'
```

---

## sed

Replace

```bash
sed 's/old/new/g' file
```

Delete line

```bash
sed '3d' file
```

Edit file

```bash
sed -i 's/foo/bar/g' file
```

---

## cut

```bash
cut -d: -f1 /etc/passwd
```

---

## sort

```bash
sort file

sort -n file

sort -r file

sort -u file
```

---

## uniq

```bash
uniq file

uniq -c file
```

---

## tr

Uppercase

```bash
echo devops | tr a-z A-Z
```

Delete spaces

```bash
tr -d ' '
```

---

## wc

```bash
wc file

wc -l file

wc -w file

wc -c file
```

---

## head & tail

```bash
head file

head -5 file

tail file

tail -10 file

tail -f logfile.log
```

---

# 6. Useful One-Liners

Delete files older than 7 days

```bash
find . -type f -mtime +7 -delete
```

Count log lines

```bash
wc -l *.log
```

Replace text in all files

```bash
sed -i 's/http/https/g' *.conf
```

Check service

```bash
systemctl is-active nginx
```

Monitor disk usage

```bash
df -h
```

Watch errors in real time

```bash
tail -f app.log | grep ERROR
```

Find largest files

```bash
du -sh * | sort -hr | head
```

---

# 7. Error Handling

Exit Status

```bash
echo $?
```

Successful exit

```bash
exit 0
```

Failure

```bash
exit 1
```

---

## Strict Mode

```bash
set -e
```

Exit on error.

```bash
set -u
```

Treat undefined variables as errors.

```bash
set -o pipefail
```

Fails pipeline if any command fails.

```bash
set -x
```

Debug mode.

---

## Trap

```bash
trap 'echo "Cleaning up..."' EXIT
```

Runs command before exiting.

---

# 8. Common DevOps Commands

Disk Usage

```bash
df -h
```

Memory

```bash
free -h
```

CPU

```bash
top
```

Running Services

```bash
systemctl status nginx
```

Listening Ports

```bash
ss -tulpn
```

IP Address

```bash
ip addr
```

Network

```bash
ping google.com
```

Download File

```bash
wget URL
```

HTTP Request

```bash
curl -I https://google.com
```

---

# Interview Tips

- Always start scripts with `#!/bin/bash`.
- Use `set -euo pipefail` in production scripts.
- Quote variables: `"$VAR"` to avoid word splitting.
- Check exit codes using `$?`.
- Use functions to make scripts reusable.
- Prefer `grep`, `awk`, and `sed` for text processing.
- Test scripts before scheduling them with cron.

---

# Key Learnings

1. Shell scripting automates repetitive Linux and DevOps tasks.
2. Functions, loops, and conditionals make scripts reusable and maintainable.
3. Text-processing tools like `grep`, `awk`, and `sed` are essential for log analysis and automation.
