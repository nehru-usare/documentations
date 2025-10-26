# 🤖 Linux Automation and Scheduling

Automation in Linux helps you **save time, reduce errors**, and **ensure consistency**.  
By mastering scheduling tools like `cron`, `at`, `anacron`, and `systemd timers`, you can run tasks automatically — daily, weekly, or even once in the future.

---

## 🧭 Table of Contents

1. [Introduction](#1-introduction)
2. [The Need for Automation](#2-the-need-for-automation)
3. [Cron — Periodic Task Scheduler](#3-cron--periodic-task-scheduler)
4. [Cron Syntax and Examples](#4-cron-syntax-and-examples)
5. [Managing Cron Jobs](#5-managing-cron-jobs)
6. [System-Wide Cron Jobs](#6-system-wide-cron-jobs)
7. [Anacron — Run Missed Jobs](#7-anacron--run-missed-jobs)
8. [at — One-Time Scheduling](#8-at--one-time-scheduling)
9. [systemd Timers — Modern Scheduling](#9-systemd-timers--modern-scheduling)
10. [Practical Automation Examples](#10-practical-automation-examples)
11. [Best Practices](#11-best-practices)
12. [Summary](#12-summary)

---

## 1️⃣ Introduction

Linux offers multiple tools for **task scheduling and automation**:
- `cron` → Repetitive tasks (minute, daily, weekly, etc.)
- `anacron` → Tasks that must run even if system was off
- `at` → One-time scheduled tasks
- `systemd timers` → Modern cron replacement with better logging and control

---

## 2️⃣ The Need for Automation

Automation reduces manual work and ensures reliability in:
- Backups  
- Log rotations  
- System updates  
- Health checks  
- File cleanup  
- Application restarts  

---

## 3️⃣ Cron — Periodic Task Scheduler

`cron` is a daemon that runs tasks automatically at defined intervals.  
Each user (including root) has their own crontab (cron table).

### 🔹 Check if cron service is running:
```bash
sudo systemctl status cron
````

### 🔹 Start / Enable cron:

```bash
sudo systemctl enable --now cron
```

---

## 4️⃣ Cron Syntax and Examples

A cron job has **five time fields** followed by the command to execute:

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of the week (0 - 7)
│ │ │ └──── Month (1 - 12)
│ │ └────── Day of the month (1 - 31)
│ └──────── Hour (0 - 23)
└────────── Minute (0 - 59)
```

### 🧩 Examples:

| Schedule            | Cron Expression | Description              |
| ------------------- | --------------- | ------------------------ |
| Every minute        | `* * * * *`     | Run command every minute |
| Every 5 minutes     | `*/5 * * * *`   | Run every 5 minutes      |
| Every day at 2AM    | `0 2 * * *`     | Daily at 2:00 AM         |
| Every Monday at 3PM | `0 15 * * 1`    | Weekly job               |
| Every 1st of month  | `0 0 1 * *`     | Monthly job              |

### 🔹 Example cron job:

```bash
0 3 * * * /home/alex/scripts/backup.sh >> /var/log/backup.log 2>&1
```

This runs at **3:00 AM daily** and logs output to `/var/log/backup.log`.

---

## 5️⃣ Managing Cron Jobs

### 🔹 View current cron jobs

```bash
crontab -l
```

### 🔹 Edit cron jobs

```bash
crontab -e
```

### 🔹 Remove all cron jobs

```bash
crontab -r
```

### 🔹 Check cron log

```bash
grep CRON /var/log/syslog
```

---

## 6️⃣ System-Wide Cron Jobs

System-wide cron jobs are defined in `/etc`:

| Location             | Description               |
| -------------------- | ------------------------- |
| `/etc/crontab`       | Global cron configuration |
| `/etc/cron.hourly/`  | Runs every hour           |
| `/etc/cron.daily/`   | Runs once daily           |
| `/etc/cron.weekly/`  | Runs weekly               |
| `/etc/cron.monthly/` | Runs monthly              |

Example `/etc/crontab` entry:

```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
0 4 * * * root /usr/local/bin/cleanup.sh
```

---

## 7️⃣ Anacron — Run Missed Jobs

`anacron` ensures periodic jobs **run even if the system was off** at the scheduled time.

Perfect for laptops or servers that reboot periodically.

### 🔹 Configuration File:

`/etc/anacrontab`

Example:

```
# period delay job-identifier command
1 5 daily-cleanup /usr/local/bin/cleanup.sh
7 10 weekly-report /usr/local/bin/report.sh
```

| Field                       | Description               |
| --------------------------- | ------------------------- |
| 1                           | Run every 1 day           |
| 5                           | Wait 5 minutes after boot |
| daily-cleanup               | Job name                  |
| `/usr/local/bin/cleanup.sh` | Command                   |

### 🔹 Run anacron manually:

```bash
sudo anacron -n
```

---

## 8️⃣ at — One-Time Scheduling

`at` lets you schedule **one-time commands or scripts** to run in the future.

### 🔹 Install `at` (if missing):

```bash
sudo apt install at
sudo systemctl enable --now atd
```

### 🔹 Schedule a job:

```bash
echo "/home/alex/scripts/backup.sh" | at 02:00
```

### 🔹 Schedule using natural time:

```bash
at now + 5 minutes
at midnight
at 10:30 AM tomorrow
```

### 🔹 View pending jobs:

```bash
atq
```

### 🔹 Remove a job:

```bash
atrm <job_id>
```

---

## 9️⃣ systemd Timers — Modern Scheduling

`systemd timers` are a **more powerful and reliable alternative** to cron.
They integrate directly with systemd for better logging, dependencies, and recovery.

### 🔹 Create Service Unit

`/etc/systemd/system/backup.service`

```ini
[Unit]
Description=Run daily backup

[Service]
ExecStart=/home/alex/scripts/backup.sh
```

### 🔹 Create Timer Unit

`/etc/systemd/system/backup.timer`

```ini
[Unit]
Description=Run backup daily at 2AM

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### 🔹 Enable and Start Timer

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
```

### 🔹 View Timers

```bash
systemctl list-timers
```

### 🔹 Logs

```bash
journalctl -u backup.service
```

---

## 🔟 Practical Automation Examples

### 🧩 Auto Update Packages Daily

```bash
0 1 * * * sudo apt update && sudo apt -y upgrade
```

### 🧩 Auto Backup Database Every Night

```bash
0 3 * * * /usr/local/bin/db-backup.sh >> /var/log/db-backup.log 2>&1
```

### 🧩 Cleanup Temporary Files Weekly

```bash
0 0 * * 0 rm -rf /tmp/*
```

### 🧩 Restart Service on Crash (systemd)

Create `/etc/systemd/system/myapp.service`:

```ini
[Service]
Restart=always
RestartSec=10
ExecStart=/usr/bin/java -jar /opt/myapp.jar
```

---

## 11️⃣ Best Practices

✅ Use **absolute paths** in cron jobs
✅ Redirect output and errors to log files (`>> file.log 2>&1`)
✅ Use `anacron` for laptops / systems that aren’t always running
✅ Prefer `systemd timers` for critical production automation
✅ Use version control for scripts (`git`)
✅ Regularly check `cron` logs (`grep CRON /var/log/syslog`)
✅ Don’t schedule heavy jobs at the same time as backups or updates

---

## 12️⃣ Summary

* **cron** — schedules recurring jobs
* **anacron** — runs missed jobs
* **at** — one-time job scheduling
* **systemd timers** — modern cron replacement
* Automating tasks improves **reliability, efficiency, and consistency**

Mastering these tools transforms your Linux system into a **self-managing, intelligent environment**.

---

> 🧭 **Next Topic:** [13-linux-administration-tips.md → System Administration & Optimization Tips](./13-linux-administration-tips.md)