# ⚙️ Linux Process Management

In Linux, every running command, program, or service is known as a **process**.  
Understanding process management is crucial for **monitoring performance**, **troubleshooting**, and **controlling applications**.

---

## 🧭 Table of Contents
1. [What is a Process?](#1-what-is-a-process)
2. [Process States](#2-process-states)
3. [Types of Processes](#3-types-of-processes)
4. [Viewing Processes](#4-viewing-processes)
5. [Foreground and Background Processes](#5-foreground-and-background-processes)
6. [Managing and Killing Processes](#6-managing-and-killing-processes)
7. [Job Control](#7-job-control)
8. [Process Priorities (nice & renice)](#8-process-priorities-nice--renice)
9. [Monitoring System Performance](#9-monitoring-system-performance)
10. [Zombie and Orphan Processes](#10-zombie-and-orphan-processes)
11. [Best Practices](#11-best-practices)
12. [Summary](#12-summary)

---

## 1️⃣ What is a Process?

A **process** is a program or command that is currently running in memory.

Each process has a unique:
- **PID (Process ID)** → Identifies the process  
- **PPID (Parent Process ID)** → The process that started it  
- **UID** → The user running the process  

You can view all processes on your system using:
```bash
ps -ef
````

Example output:

```
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Oct25 ?        00:01:25 systemd
alex      2048     1  1 09:10 ?        00:00:05 java -jar app.jar
```

---

## 2️⃣ Process States

| State                   | Description                        |
| ----------------------- | ---------------------------------- |
| **R (Running)**         | Currently running or ready to run  |
| **S (Sleeping)**        | Waiting for an event (I/O, etc.)   |
| **T (Stopped)**         | Paused or stopped                  |
| **Z (Zombie)**          | Finished but not removed by parent |
| **D (Uninterruptible)** | Waiting for hardware I/O           |

You can view process states using:

```bash
ps -eo pid,stat,comm
```

---

## 3️⃣ Types of Processes

| Type           | Description                                 |
| -------------- | ------------------------------------------- |
| **Foreground** | Tied to terminal, requires user interaction |
| **Background** | Runs without terminal interaction           |
| **Daemon**     | Background service (e.g., `sshd`, `crond`)  |
| **Parent**     | Process that spawns another                 |
| **Child**      | Created by parent process                   |
| **Zombie**     | Completed but not cleaned up                |
| **Orphan**     | Parent process exited before child finished |

---

## 4️⃣ Viewing Processes

### 🔹 `ps` — Process Status

```bash
ps aux       # Show all processes with details
ps -ef       # Standard full listing
ps -u user   # Show processes by a specific user
```

Example output:

```
USER   PID %CPU %MEM  VSZ  RSS TTY  STAT START   TIME COMMAND
root     1  0.0  0.2 22564 3456 ?    Ss   09:00   0:01 systemd
alex  2050  2.5  3.0 95444 8096 pts/0 S+   09:10   0:03 java -jar app.jar
```

---

### 🔹 `top` — Real-Time Process Viewer

```bash
top
```

Press:

* `P` → Sort by CPU usage
* `M` → Sort by memory usage
* `k` → Kill a process (enter PID)
* `q` → Quit

To exit, press **`q`**.

---

### 🔹 `htop` — Enhanced top (colorful, interactive)

Install first:

```bash
sudo apt install htop
```

Run:

```bash
htop
```

---

## 5️⃣ Foreground and Background Processes

By default, commands run in the **foreground** — blocking the terminal until they finish.

### Run a process in background

Add `&` at the end:

```bash
python3 script.py &
```

Check background jobs:

```bash
jobs
```

Bring it to foreground:

```bash
fg %1
```

Send a running process to background (pause first):

```bash
Ctrl + Z
bg
```

---

## 6️⃣ Managing and Killing Processes

### 🔹 Kill by PID

```bash
kill <PID>
```

### 🔹 Force Kill

```bash
kill -9 <PID>
```

### 🔹 Kill by Process Name

```bash
pkill <name>
```

### 🔹 Kill All Matching Processes

```bash
killall <process_name>
```

Example:

```bash
pkill java       # Kill all Java processes
killall firefox  # Kill all Firefox instances
```

---

## 7️⃣ Job Control

Job control helps you manage processes running in your terminal session.

| Command    | Description                |
| ---------- | -------------------------- |
| `jobs`     | List all background jobs   |
| `bg`       | Resume a job in background |
| `fg`       | Bring a job to foreground  |
| `Ctrl + Z` | Suspend foreground job     |
| `Ctrl + C` | Terminate foreground job   |

Example:

```bash
sleep 300 &
jobs
fg %1
```

---

## 8️⃣ Process Priorities (nice & renice)

Linux allows you to assign **priorities** to processes using **nice values**.
Lower nice values = higher priority.

| Range | Meaning          |
| ----- | ---------------- |
| -20   | Highest priority |
| 0     | Default          |
| +19   | Lowest priority  |

### 🔹 Start a process with priority

```bash
nice -n 10 myscript.sh
```

### 🔹 Change priority of a running process

```bash
renice -n 5 -p 1234
```

Check:

```bash
ps -o pid,ni,comm -p 1234
```

---

## 9️⃣ Monitoring System Performance

### 🔹 View System Load and Uptime

```bash
uptime
```

Output:

```
10:23:14 up 2 days, 3:40, 2 users, load average: 0.45, 0.38, 0.25
```

### 🔹 Memory and CPU Usage

```bash
free -h
```

### 🔹 Disk I/O and CPU stats

```bash
iostat
```

### 🔹 Virtual Memory Stats

```bash
vmstat 2 5
```

### 🔹 Real-Time System Monitoring

```bash
top
htop
```

---

## 🔟 Zombie and Orphan Processes

### 🧟 Zombie Process

A process that has completed execution but still appears in process table because its **parent didn’t clean it up**.

Identify zombies:

```bash
ps aux | grep Z
```

Remove by killing parent:

```bash
kill -HUP <parent_PID>
```

---

### 👶 Orphan Process

Occurs when a parent process terminates before its child.

The **init/systemd process** (PID 1) automatically adopts orphans.

Check with:

```bash
ps -ef | grep <child_pid>
```

---

## 11️⃣ Best Practices

✅ Regularly monitor system processes using `top` or `htop`
✅ Avoid using `kill -9` unless absolutely necessary
✅ Use `nice` for CPU-intensive jobs to maintain system responsiveness
✅ Log and audit long-running processes
✅ Investigate high CPU or memory usage (`ps aux --sort=-%cpu | head`)
✅ Automate monitoring using cron + log tools

---

## 12️⃣ Summary

* Each program running in Linux is a **process** identified by a **PID**.
* Use `ps`, `top`, `htop` to view and manage them.
* Use `kill`, `pkill`, and `killall` to terminate processes.
* Manage process priorities using `nice` and `renice`.
* Understand **zombie** and **orphan** processes to maintain system health.
* Efficient process management ensures better **performance, stability, and resource usage**.

---

> 🧭 **Next Topic:** [07-package-management.md → Installing and Managing Software Packages](./07-package-management.md)