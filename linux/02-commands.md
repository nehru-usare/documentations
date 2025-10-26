# ⚙️ Basic Linux Commands

This guide introduces essential **Linux commands** for beginners.

---

## 📂 File and Directory Navigation

| Command | Description |
|----------|-------------|
| `pwd` | Print working directory |
| `ls` | List files and directories |
| `cd <dir>` | Change directory |
| `mkdir <dir>` | Create a new directory |
| `rmdir <dir>` | Remove empty directory |
| `touch <file>` | Create an empty file |
| `cp <src> <dest>` | Copy files |
| `mv <src> <dest>` | Move or rename files |
| `rm <file>` | Delete files |

---

## 📄 Viewing Files

| Command | Description |
|----------|-------------|
| `cat <file>` | Display file content |
| `less <file>` | View file page by page |
| `head -n 10 <file>` | Show first 10 lines |
| `tail -n 10 <file>` | Show last 10 lines |
| `grep "text" <file>` | Search for text inside files |
| `wc -l <file>` | Count lines in a file |

---

## ⚙️ System Information

| Command | Description |
|----------|-------------|
| `uname -a` | Kernel and system info |
| `df -h` | Disk space usage |
| `free -m` | Memory usage in MB |
| `top` | Real-time system processes |
| `uptime` | System uptime |
| `whoami` | Current username |

---

## 🔄 Working with History and Help

| Command | Description |
|----------|-------------|
| `history` | Show command history |
| `!!` | Run last command again |
| `man <command>` | Manual page for command |
| `<command> --help` | Quick help option |

---

## 🧰 File Search

| Command | Description |
|----------|-------------|
| `find /path -name "*.txt"` | Find text files |
| `locate <filename>` | Quickly find files (using index) |
| `which <command>` | Find command location |

---

> 🧭 **Next:** [Linux File System →](./03-file-system.md)
