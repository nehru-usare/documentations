# 🐚 Linux Shell Scripting — Automation with Bash

Shell scripting allows you to **automate repetitive tasks**, perform **system operations**, and build **powerful CLI tools** using the Linux terminal.  
It’s an essential skill for developers, DevOps engineers, and system administrators.

This document provides a full introduction to **Bash scripting**, including syntax, variables, loops, conditionals, and practical examples.

---

## 🧭 Table of Contents

1. [What is a Shell Script?](#1-what-is-a-shell-script)
2. [Creating Your First Script](#2-creating-your-first-script)
3. [Making a Script Executable](#3-making-a-script-executable)
4. [Variables and User Input](#4-variables-and-user-input)
5. [Command Substitution](#5-command-substitution)
6. [Conditionals (if / else)](#6-conditionals-if--else)
7. [Loops (for, while, until)](#7-loops-for-while-until)
8. [Functions](#8-functions)
9. [Positional Parameters ($1, $2, ...)](#9-positional-parameters-1-2-)
10. [Exit Status and Error Handling](#10-exit-status-and-error-handling)
11. [Useful Script Examples](#11-useful-script-examples)
12. [Best Practices](#12-best-practices)
13. [Summary](#13-summary)

---

## 1️⃣ What is a Shell Script?

A **shell script** is a text file that contains a series of **commands** executed by the shell.  
It’s like automating typing commands manually into the terminal.

- Default shell in Linux: **bash** (Bourne Again Shell)
- Other shells: `zsh`, `sh`, `ksh`, `fish`

---

## 2️⃣ Creating Your First Script

1. Open your terminal and create a new file:
   ```bash
   nano hello.sh
````

2. Add the following lines:

   ```bash
   #!/bin/bash
   echo "Hello, Linux World!"
  ````

   The first line (`#!/bin/bash`) is called a **shebang** — it tells the system which interpreter to use.

3. Save and exit (`Ctrl + O`, `Enter`, `Ctrl + X`).

---

## 3️⃣ Making a Script Executable

Before running a script, you must make it executable.

```bash
chmod +x hello.sh
```

Run the script:

```bash
./hello.sh
```

Output:

```
Hello, Linux World!
```

---

## 4️⃣ Variables and User Input

### 🔹 Declaring Variables

```bash
#!/bin/bash
name="Alex"
echo "Welcome, $name!"
```

> No spaces around `=` in Bash!

### 🔹 Reading User Input

```bash
#!/bin/bash
echo "Enter your name:"
read username
echo "Hello, $username!"
```

You can also read input silently (e.g., password):

```bash
read -s password
```

---

## 5️⃣ Command Substitution

You can assign command output to variables:

```bash
current_date=$(date)
echo "Today's date is: $current_date"
```

Other example:

```bash
user_count=$(who | wc -l)
echo "There are $user_count users logged in."
```

---

## 6️⃣ Conditionals (if / else)

Conditionals let you execute code **based on conditions**.

```bash
#!/bin/bash
echo "Enter a number:"
read num

if [ $num -gt 10 ]; then
  echo "Number is greater than 10"
elif [ $num -eq 10 ]; then
  echo "Number equals 10"
else
  echo "Number is less than 10"
fi
```

### 🧩 Comparison Operators

| Operator | Description      |
| -------- | ---------------- |
| `-eq`    | Equal            |
| `-ne`    | Not equal        |
| `-gt`    | Greater than     |
| `-lt`    | Less than        |
| `-ge`    | Greater or equal |
| `-le`    | Less or equal    |

### File test operators

| Operator  | Meaning                  |
| --------- | ------------------------ |
| `-f file` | True if file exists      |
| `-d dir`  | True if directory exists |
| `-r file` | True if readable         |
| `-w file` | True if writable         |
| `-x file` | True if executable       |

Example:

```bash
if [ -d /home/$USER ]; then
  echo "Home directory exists"
fi
```

---

## 7️⃣ Loops (for, while, until)

### 🔹 For Loop

```bash
for i in 1 2 3 4 5
do
  echo "Number: $i"
done
```

### 🔹 For Loop (with range)

```bash
for i in {1..5}
do
  echo "Iteration $i"
done
```

### 🔹 While Loop

```bash
count=1
while [ $count -le 5 ]
do
  echo "Count: $count"
  ((count++))
done
```

### 🔹 Until Loop

```bash
num=1
until [ $num -gt 5 ]
do
  echo "Num: $num"
  ((num++))
done
```

---

## 8️⃣ Functions

Functions make your script modular and reusable.

```bash
#!/bin/bash

say_hello() {
  echo "Hello, $1!"
}

say_hello "Alice"
say_hello "Bob"
```

You can also return values:

```bash
get_sum() {
  local sum=$(( $1 + $2 ))
  echo $sum
}

result=$(get_sum 5 10)
echo "Result: $result"
```

---

## 9️⃣ Positional Parameters ($1, $2, ...)

When you run a script with arguments:

```bash
./myscript.sh arg1 arg2
```

You can access them like this:

```bash
echo "First argument: $1"
echo "Second argument: $2"
```

| Variable        | Description                 |
| --------------- | --------------------------- |
| `$0`            | Script name                 |
| `$1`, `$2`, ... | Command-line arguments      |
| `$#`            | Number of arguments         |
| `$@`            | All arguments as list       |
| `$?`            | Exit status of last command |

---

## 🔟 Exit Status and Error Handling

Every command returns an **exit status**:

* `0` → success
* Non-zero → error

Example:

```bash
cp file.txt /backup/
if [ $? -eq 0 ]; then
  echo "Backup successful!"
else
  echo "Backup failed!"
fi
```

You can exit manually:

```bash
exit 1
```

Or use error handling with `set`:

```bash
set -e   # Exit immediately if a command fails
```

---

## 11️⃣ Useful Script Examples

### 🧩 Example 1: Backup Script

```bash
#!/bin/bash
SOURCE="/home/$USER/Documents"
DEST="/home/$USER/backup"
DATE=$(date +%Y-%m-%d)

mkdir -p $DEST/$DATE
cp -r $SOURCE/* $DEST/$DATE/

echo "Backup completed successfully on $DATE."
```

---

### 🧩 Example 2: Disk Usage Alert

```bash
#!/bin/bash
THRESHOLD=80
USAGE=$(df / | grep / | awk '{print $5}' | sed 's/%//g')

if [ $USAGE -gt $THRESHOLD ]; then
  echo "Disk usage is above ${THRESHOLD}% - current: ${USAGE}%"
else
  echo "Disk usage is normal: ${USAGE}%"
fi
```

---

### 🧩 Example 3: Simple Menu Script

```bash
#!/bin/bash
echo "1. Show Date"
echo "2. Show Uptime"
echo "3. Show Disk Usage"
read -p "Choose an option: " choice

case $choice in
  1) date ;;
  2) uptime ;;
  3) df -h ;;
  *) echo "Invalid option" ;;
esac
```

---

### 🧩 Example 4: Monitor Running Process

```bash
#!/bin/bash
read -p "Enter process name: " process
if pgrep $process > /dev/null
then
  echo "$process is running"
else
  echo "$process is not running"
fi
```

---

## 12️⃣ Best Practices

✅ Always include `#!/bin/bash` at the top of scripts
✅ Use comments (`#`) to describe logic
✅ Quote variables: `"$var"` (avoids word-splitting issues)
✅ Use `set -e` to stop on errors
✅ Use functions for reusable logic
✅ Test scripts in a safe environment before deploying
✅ Log output using redirection (`>> logfile.txt`)

---

## 13️⃣ Summary

* Shell scripts are **powerful automation tools** in Linux.
* Use variables, conditionals, loops, and functions to build flexible scripts.
* Always test and handle errors gracefully.
* Scripts can automate **backups, monitoring, deployments, and reports**.
* Mastering shell scripting saves time and boosts productivity.

---

> 🧭 **Next Topic:** [10-advanced-topics.md → Advanced Linux Administration (Processes, Cron Jobs, Environment)](./10-advanced-topics.md)