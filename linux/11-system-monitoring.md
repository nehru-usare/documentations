# 📊 Linux System Monitoring

System monitoring is essential for maintaining **performance, stability, and security** in Linux systems.  
This guide covers tools and commands to track **CPU, memory, disk, processes, and network usage** — both in real time and historically.

---

## 🧭 Table of Contents

1. [Introduction](#1-introduction)
2. [System Resource Overview](#2-system-resource-overview)
3. [CPU Monitoring](#3-cpu-monitoring)
4. [Memory Monitoring](#4-memory-monitoring)
5. [Disk Usage and I/O Monitoring](#5-disk-usage-and-io-monitoring)
6. [Process Monitoring Tools](#6-process-monitoring-tools)
7. [Network Monitoring](#7-network-monitoring)
8. [System Logging and Audit](#8-system-logging-and-audit)
9. [Performance Monitoring Tools](#9-performance-monitoring-tools)
10. [Automated Health Checks](#10-automated-health-checks)
11. [Best Practices](#11-best-practices)
12. [Summary](#12-summary)

---

## 1️⃣ Introduction

Monitoring Linux systems ensures that:
- You can **detect issues early**
- **Optimize performance**
- **Ensure availability** and **reliability**

Monitoring can be:
- **Manual** (CLI tools)
- **Automated** (scripts, daemons)
- **Centralized** (Prometheus, Grafana, Nagios, etc.)

---

## 2️⃣ System Resource Overview

Start by checking system performance using simple built-in tools.

| Command | Description |
|----------|-------------|
| `top` | Real-time view of system processes |
| `htop` | Interactive process viewer (colorful top) |
| `vmstat` | CPU, memory, I/O statistics |
| `free -h` | Display memory usage |
| `df -h` | Show disk space usage |
| `iostat` | Display CPU and I/O statistics |
| `uptime` | Show system uptime and load averages |

---

## 3️⃣ CPU Monitoring

### 🔹 View CPU Load

```bash
uptime
````

Example:

```
10:40:52 up 3 days, 4:12, 2 users, load average: 0.25, 0.30, 0.28
```

| Field  | Description     |
| ------ | --------------- |
| 1 min  | Short-term load |
| 5 min  | Mid-term load   |
| 15 min | Long-term load  |

If the load average exceeds the number of CPU cores, the system is **CPU-bound**.

---

### 🔹 Detailed CPU Stats

```bash
mpstat 1
```

Install (if missing):

```bash
sudo apt install sysstat
```

### 🔹 View CPU Usage per Process

```bash
top
# or
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head
```

---

## 4️⃣ Memory Monitoring

### 🔹 View Overall Memory Usage

```bash
free -h
```

Output:

```
              total        used        free      shared  buff/cache   available
Mem:           7.6G        2.3G        3.2G        125M        2.1G        5.1G
Swap:          2.0G          0B        2.0G
```

### 🔹 Check Memory Usage by Process

```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head
```

### 🔹 Memory Pressure Statistics (Kernel)

```bash
cat /proc/meminfo
```

---

## 5️⃣ Disk Usage and I/O Monitoring

### 🔹 Disk Space Overview

```bash
df -h
```

Output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       100G   60G   35G  65% /
```

### 🔹 Directory-Level Usage

```bash
du -sh *
```

### 🔹 Monitor Disk I/O

```bash
iostat -xz 1
```

| Metric       | Description                      |
| ------------ | -------------------------------- |
| `%util`      | How busy the disk is             |
| `r/s`, `w/s` | Read/write operations per second |
| `await`      | Average wait time                |

### 🔹 Real-Time Disk Activity

```bash
iotop
```

> Install via: `sudo apt install iotop`

---

## 6️⃣ Process Monitoring Tools

### 🔹 top

Real-time interactive process viewer:

```bash
top
```

Useful keys:

* `P` → Sort by CPU
* `M` → Sort by memory
* `k` → Kill process
* `1` → Show per-CPU stats
* `q` → Quit

---

### 🔹 htop

```bash
sudo apt install htop
htop
```

Provides:

* Tree view of processes
* Mouse support
* Colorful CPU/memory bars

---

### 🔹 ps (Process Status)

```bash
ps aux | grep nginx
```

| Option | Description                  |
| ------ | ---------------------------- |
| `a`    | All users                    |
| `u`    | Display user/owner           |
| `x`    | Include background processes |

---

## 7️⃣ Network Monitoring

### 🔹 Display Network Interfaces

```bash
ip link show
```

### 🔹 Network Statistics

```bash
netstat -s
ss -s
```

### 🔹 Show Listening Ports

```bash
ss -tuln
```

### 🔹 Monitor Bandwidth (Live)

```bash
sudo apt install iftop nload
sudo iftop -i eth0
sudo nload
```

### 🔹 Check Network Connectivity

```bash
ping -c 4 google.com
traceroute openai.com
```

---

## 8️⃣ System Logging and Audit

Linux logs everything in `/var/log`.

| File                | Purpose                   |
| ------------------- | ------------------------- |
| `/var/log/syslog`   | System messages           |
| `/var/log/auth.log` | Authentication events     |
| `/var/log/dmesg`    | Kernel messages           |
| `/var/log/cron.log` | Cron job logs             |
| `/var/log/apache2/` | Web server logs           |
| `/var/log/messages` | General logs (RHEL-based) |

### 🔹 View Logs in Real Time

```bash
tail -f /var/log/syslog
```

### 🔹 Using journalctl (systemd logs)

```bash
sudo journalctl
sudo journalctl -u ssh
sudo journalctl --since "2 hours ago"
```

---

## 9️⃣ Performance Monitoring Tools

### 🔹 vmstat

Reports CPU, memory, and I/O stats.

```bash
vmstat 2 5
```

Output columns:

* `r`: processes waiting for CPU
* `b`: processes in uninterruptible sleep
* `si/so`: swap in/out
* `us`: user CPU usage

---

### 🔹 sar

Collects and reports performance metrics.

```bash
sudo apt install sysstat
sar -u 5 5      # CPU stats
sar -r 5 5      # Memory stats
sar -n DEV 5 5  # Network stats
```

---

### 🔹 dstat

Combines vmstat, iostat, netstat in one tool.

```bash
sudo apt install dstat
dstat -cdngy 2
```

---

### 🔹 atop

Advanced real-time performance monitor.

```bash
sudo apt install atop
sudo atop
```

Displays CPU, memory, disk, and process metrics in detail.

---

## 🔟 Automated Health Checks

You can automate monitoring tasks using **scripts** and **cron jobs**.

Example: Disk usage alert script

```bash
#!/bin/bash
THRESHOLD=80
USAGE=$(df / | grep / | awk '{print $5}' | sed 's/%//g')

if [ $USAGE -gt $THRESHOLD ]; then
  echo "⚠️ Disk usage above ${THRESHOLD}%: ${USAGE}%" | mail -s "Disk Alert" admin@example.com
fi
```

Add to cron:

```bash
0 * * * * /home/user/scripts/disk-alert.sh
```

---

## 11️⃣ Best Practices

✅ Use `top` or `htop` for quick monitoring
✅ Regularly review `/var/log/` for errors
✅ Automate health checks using cron scripts
✅ Monitor disk usage weekly (`du -sh /`)
✅ Use centralized monitoring (e.g., Prometheus + Grafana)
✅ Check system load before deploying new workloads
✅ Keep system updated to ensure stability and security

---

## 12️⃣ Summary

* Monitoring ensures **system performance and reliability**.
* Use tools like `top`, `htop`, `vmstat`, `iostat`, and `sar`.
* Monitor **CPU**, **memory**, **disk**, and **network** usage regularly.
* Logs and system metrics help diagnose issues quickly.
* Automate periodic health checks for proactive maintenance.

---

> 🧭 **Next Topic:** [12-automation-and-scheduling.md → Automating Tasks & Scheduling in Linux](./12-automation-and-scheduling.md)