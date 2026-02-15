# 📊 Linux System Monitoring & Performance

## 📋 Table of Contents
- [1️⃣ The 4 Golden Signals (SRE)](#1️⃣-the-4-golden-signals-sre)
- [2️⃣ CPU & Load Analysis](#2️⃣-cpu--load-analysis)
- [3️⃣ Memory Analysis (RAM & Swap)](#3️⃣-memory-analysis-ram--swap)
- [4️⃣ Disk I/O Analysis](#4️⃣-disk-io-analysis)
- [5️⃣ Network Traffic Analysis](#5️⃣-network-traffic-analysis)
- [6️⃣ Deep Dive: strace (The X-Ray)](#6️⃣-deep-dive-strace-the-x-ray)
- [7️⃣ Deep Dive: tcpdump (The MRI)](#7️⃣-deep-dive-tcpdump-the-mri)
- [8️⃣ The Future: eBPF](#8️⃣-the-future-ebpf)

---

## 1️⃣ The 4 Golden Signals (SRE)

Google SREs measure system health using these 4 metrics:

| Signal | Linux Tool | What to look for? |
|:-------|:-----------|:------------------|
| **Latency** | `curl`, `ping` | How long does it take to serve a request? |
| **Traffic** | `nload`, `iftop` | Bandwidth usage (Mb/s). |
| **Errors** | `journalctl`, `grep 500` | HTTP 500s, specific logs. |
| **Saturation** | `top`, `uptime` | "Fullness". High Load, Full Disk, 100% CPU. |

---

## 2️⃣ CPU & Load Analysis

**Load Average** (`uptime`):
- `1.0` Load = 1 CPU Core is 100% busy.
- `4.0` Load on a 4-core machine = 100% Saturation.
- **Rule of Thumb**: If Load > (Cores * 2), system is stuck.

**Tools:**
- `htop`: Visual.
- `mpstat -P ALL 1`: Per-core usage.

---

## 3️⃣ Memory Analysis (RAM & Swap)

**Key Concept: Cache is Good.**
Linux borrows unused RAM to cache files.
`free -m` might show "Free: 100MB", but "Available: 8GB".
**Trust "Available", not "Free".**

**OOM Killer**:
If you run out of RAM + Swap, the Kernel **kills** the biggest process to save the system.
Check logs:
```bash
dmesg | grep -i "out of memory"
```

---

## 4️⃣ Disk I/O Analysis

Is the database slow because of CPU or Disk?

**Command**: `iostat -xz 1`
Look at **`%iowait`**:
- High CPU + Low iowait = Application Logic (Code) issue.
- Low CPU + **High iowait** = Disk cannot keep up (Hardware/DB) issue.

**Command**: `iotop`
Who is writing to disk?

---

## 5️⃣ Network Traffic Analysis

**Command**: `ss -tulnp`
See who is listening.

**Command**: `iftop`
"Who is downloading 5GB of data right now?"
Shows real-time connections and bandwidth.

---

## 6️⃣ Deep Dive: strace (The X-Ray)

`strace` logs every **System Call** (Syscall) a process makes.
"Why is Nginx failing to start? (No logs produced)."

```bash
# Attach to a running process
sudo strace -p 1234

# Run a command and trace it
strace -e openat,connect,write cat /etc/passwd
```

**Output**:
```
openat("/etc/passwd") = 3
read(3, "root:x:0...", 1024)
write(1, "root:x:0...", ...)
```
If you see `EACCES (Permission denied)`, you found the bug.

---

## 7️⃣ Deep Dive: tcpdump (The MRI)

Captures actual packets off the wire.

```bash
# Capture web traffic on eth0
sudo tcpdump -i eth0 port 80 -n -A
```

**Use Case**:
- "Is the API sending the JSON body?"
- "Is the firewall dropping the ACK packet?"

---

## 8️⃣ The Future: eBPF

**eBPF** (Extended Berkeley Packet Filter) allows us to run sandboxed programs inside the kernel without recompiling it.

**Tools**:
- `bcc-tools`: `opensnoop` (Show all files opened live), `execsnoop` (Show all new processes).
- `bpftrace`: Scriptable tracing.

Example (`execsnoop`):
```bash
# Watch every command running on the system LIVE
sudo execsnoop
```

---

> 🧭 **Next:** [Automation: Cron & Systemd Timers →](./12-automation.md)