# ⚙️ Linux Process Management

## 📋 Table of Contents
- [1️⃣ Process Basics & Lifecycle](#1️⃣-process-basics--lifecycle)
- [2️⃣ Internal Mechanics: The Process Control Block (PCB)](#2️⃣-internal-mechanics-the-process-control-block-pcb)
- [3️⃣ Viewing Processes (ps, top)](#3️⃣-viewing-processes-ps-top)
- [4️⃣ Deep Dive: Signals (Kill is not just for killing)](#4️⃣-deep-dive-signals-kill-is-not-just-for-killing)
- [5️⃣ Job Control (Foreground/Background)](#5️⃣-job-control-foregroundbackground)
- [6️⃣ Performance: Load Averages Main Explained](#6️⃣-performance-load-averages-main-explained)
- [7️⃣ Process Priority (Nice & Renice)](#7️⃣-process-priority-nice--renice)
- [8️⃣ Zombies & Orphans](#8️⃣-zombies--orphans)

---

## 1️⃣ Process Basics & Lifecycle

In Linux, **Every Process** (except PID 1) is born from a parent process.

### The Lifecycle (Fork & Exec)
1.  **Fork()**: Parent creates a **Clone** of itself.
2.  **Exec()**: The clone replaces itself with a **New Program**.
3.  **Wait()**: Parent waits for the child to finish.
4.  **Exit()**: Child finishes and sends a status code back.

**Example**: When you type `ls`:
`Bash` (Parent) -> **Forks** -> `Bash Clone` -> **Execs** `ls` -> `ls` runs -> **Exits**.

---

## 2️⃣ Internal Mechanics: The Process Control Block (PCB)

The Kernel tracks every process in a structure called **PCB**. It stores:

| Field | Description |
|:------|:------------|
| **PID** | Unique ID (e.g., 1052). |
| **State** | Running, Sleeping, Stopped. |
| **Program Counter** | Which line of code is running *right now*. |
| **Open Files** | List of file descriptors (stdin, stdout, network sockets). |
| **Memory Map** | Which RAM addresses belong to this app. |

---

## 3️⃣ Viewing Processes (ps, top)

| Command | Status | Use Case |
|:--------|:-------|:---------|
| `ps -ef` | Snapshot | "Is Nginx running right now?" |
| `top` | Real-time | "Why is the fan spinning so loud?" |
| `htop` | Interactive | "I want to click on things to kill them." |

---

## 4️⃣ Deep Dive: Signals (Kill is not just for killing)

The `kill` command actually sends **Signals**.
There are 64 signals. Here are the big ones:

| Signal | ID | Name | Meaning | Can be Blocked? |
|:-------|:---|:-----|:--------|:----------------|
| **SIGHUP** | 1 | Hangup | "Reload config" (or User logged out). | ✅ Yes |
| **SIGINT** | 2 | Interrupt | `CTRL+C`. Polite "Stop now". | ✅ Yes |
| **SIGTERM**| 15 | Terminate | "Please save & quit". (Default). | ✅ Yes |
| **SIGKILL**| 9 | Kill | **"Die Instantly".** Kernel rips it from RAM. | ❌ NO |
| **SIGSTOP**| 19 | Stop | Pause execution (Freezes app). | ❌ NO |
| **SIGCONT**| 18 | Continue | Un-pause a stopped app. | ✅ Yes |

**Best Practice**: Always try `kill -15` (Term) first. Only use `kill -9` if it's stuck.

---

## 5️⃣ Job Control (Foreground/Background)

| Action | Command |
|:-------|:--------|
| **Run in background** | `./script.sh &` |
| **Pause current** | `CTRL+Z` |
| **Resume in background**| `bg` |
| **Bring to foreground** | `fg` |

---

## 6️⃣ Performance: Load Averages Main Explained

Run `uptime`:
`load average: 0.50, 1.20, 5.00`
(1 min, 5 min, 15 min averages).

**What does 1.00 mean?**
- It means **100% of 1 CPU Code is busy**.
- If you have **4 Cores**, a load of `1.00` means only 25% usage.
- **Danger Zone**: If Load > Number of Cores.

**High Load vs High CPU**:
- **High CPU**: Running math calculations.
- **High Load**: Waiting for Disk (I/O). A process is "Running" but stuck waiting for files.

---

## 7️⃣ Process Priority (Nice & Renice)

The Kernel decides who runs next based on "Niceness".
- **-20**: "I am not nice. I want CPU NOW." (High Priority).
- **+19**: "I am very nice. Take your time." (Low Priority).

```bash
# Start a backup quietly
nice -n 19 tar -czf backup.tar.gz /home

# Make a compiling job faster (Requires Root)
sudo renice -n -10 -p 4521
```

---

## 8️⃣ Zombies & Orphans

| Type | Definition | Solution |
|:-----|:-----------|:---------|
| **Zombie** | Dead process, waiting for Parent to read exit code. | Kill the **Parent**, or wait. |
| **Orphan** | Parent died, Child still running. | Systemd (PID 1) adopts it. |

**Command to find Zombies**:
```bash
ps aux | grep 'Z'
```

---

> 🧭 **Next:** [Package Management Deep Dive →](./07-package-management.md)