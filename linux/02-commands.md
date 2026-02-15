# ⚙️ Basic & Advanced Linux Commands

## 📋 Table of Contents
- [1️⃣ Navigation & File Management](#1️⃣-navigation--file-management)
- [2️⃣ System Information](#2️⃣-system-information)
- [3️⃣ Search & Find](#3️⃣-search--find)
- [4️⃣ Internal Mechanics: Standard Streams (stdin, stdout, stderr)](#4️⃣-internal-mechanics-standard-streams-stdin-stdout-stderr)
- [5️⃣ Redirection & Piping](#5️⃣-redirection--piping)
- [6️⃣ Developer Deep Dive: Exit Codes](#6️⃣-developer-deep-dive-exit-codes)
- [7️⃣ Command Chaining (&&, ||, ;)](#7️⃣-command-chaining-----)
- [8️⃣ Productivity: Aliases](#8️⃣-productivity-aliases)

---

## 1️⃣ Navigation & File Management

| Command | Common Flags | Description | Example |
|:--------|:-------------|:------------|:--------|
| `pwd` | - | **P**rint **W**orking **D**irectory. | `pwd` → `/home/user` |
| `ls` | `-l` (long), `-a` (all), `-h` (human readable) | List contents. | `ls -lah` |
| `cd` | `-` (previous dir) | Change Directory. | `cd /var/log` |
| `cp` | `-r` (recursive) | Copy files/folders. | `cp -r src backup_src` |
| `mv` | - | Move or **Rename**. | `mv old.txt new.txt` |
| `rm` | `-r` (recursive), `-f` (force) | Remove. | `rm -rf node_modules` |

> **Pro Tip:** Use `cd -` to toggle between your current and previous directory.

---

## 2️⃣ System Information

| Command | Description | Use Case |
|:--------|:------------|:---------|
| `uname -r` | Kernel version. | checking if you need a reboot after update. |
| `df -h` | Disk usage. | "Why is the database failing? Oh, disk full." |
| `free -m` | RAM usage (MB). | Checking for memory leaks. |
| `top` | Process viewer. | "Which app is eating my CPU?" |

---

## 3️⃣ Search & Find

| Tool | Command | Description |
|:-----|:--------|:------------|
| **find** | `find /var/log -name "*.log"` | Search by **filename**. Slow but powerful. |
| **grep** | `grep -r "error" .` | Search by **content** inside files. |
| **which**| `which java` | Shows *where* the executable is (`/usr/bin/java`). |

---

## 4️⃣ Internal Mechanics: Standard Streams (stdin, stdout, stderr)

Every Linux command has 3 standard data streams connected to it:

| Stream | ID | Description | Default Target |
|:-------|:---|:------------|:---------------|
| **stdin** | 0 | **Input**: Data fed into the program. | Keyboard |
| **stdout** | 1 | **Output**: The normal result. | Screen |
| **stderr** | 2 | **Error**: Error messages. | Screen |

This separation allows us to separate "Real Data" from "Error Logs".

---

## 5️⃣ Redirection & Piping

### 🔹 Redirection (`>`)
Send output to a file instead of the screen.

```bash
# Save output to file (OVERWRITE)
echo "Hello" > file.txt

# Append to file
echo "World" >> file.txt

# Redirect ONLY Errors (2>)
grep "forbidden" /var/log/syslog 2> errors.txt

# Redirect Everything (stdout & stderr)
./script.sh > all_logs.txt 2>&1
```

### 🔹 Piping (`|`)
Take the **stdout** of Command A and push it into **stdin** of Command B.

```bash
# Count how many files are in the current folder
ls -1 | wc -l
# (ls lists files) -> (wc counts lines)
```

---

## 6️⃣ Developer Deep Dive: Exit Codes

Every command returns a number when it finishes: **The Exit Code**.
- **`0`**: Success.
- **`1-255`**: Failure.

You can check the last code with `$?`.

```bash
ls /existing-folder
echo $?
# Output: 0 (Success)

ls /non-existent-folder
echo $?
# Output: 2 (Error)
```

**Scripting Use Case**:
```bash
git pull
if [ $? -eq 0 ]; then
    echo "Update successful!"
else
    echo "Update failed!"
fi
```

---

## 7️⃣ Command Chaining (&&, ||, ;)

Control the flow based on success or failure.

| Operator | Logic | Example |
|:---------|:------|:--------|
| **`&&`** | Run next ONLY if previous **Succeeded**. | `mkdir build && cd build` |
| **`||`** | Run next ONLY if previous **Failed**. | `ping google.com || echo "No Internet"` |
| **`;`** | Run next **Always** (Ignore errors). | `date ; echo "Done"` |

**Scenario**: Deploying code.
```bash
# Stop server, AND IF that works, update code, AND IF that works, start server.
sudo systemctl stop app && git pull && sudo systemctl start app
```

---

## 8️⃣ Productivity: Aliases

Create shortcuts for long commands in your `~/.bashrc` or `~/.zshrc`.

```bash
# 1. Open config
nano ~/.bashrc

# 2. Add aliases
alias ll='ls -lah'
alias update='sudo apt update && sudo apt upgrade'
alias gs='git status'

# 3. Reload config
source ~/.bashrc
```

Now you just type `update` to upgrade your whole system!

---

> 🧭 **Next:** [File System Deep Dive →](./03-file-system.md)
