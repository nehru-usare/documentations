# ⚙️ Advanced Linux Topics

This section dives into advanced Linux concepts — designed for developers, DevOps engineers, and system administrators.  
We’ll explore system automation, environment configuration, process scheduling, monitoring, and key administrative tools.

---

## 🧭 Table of Contents

1. [Environment Variables](#1-environment-variables)
2. [Process Scheduling with Cron Jobs](#2-process-scheduling-with-cron-jobs)
3. [System Monitoring and Logs](#3-system-monitoring-and-logs)
4. [Systemd Services](#4-systemd-services)
5. [File Compression and Archiving](#5-file-compression-and-archiving)
6. [Disk and Storage Management](#6-disk-and-storage-management)
7. [Networking and Firewalls](#7-networking-and-firewalls)
8. [System Security Essentials](#8-system-security-essentials)
9. [Performance Tuning Tips](#9-performance-tuning-tips)
10. [Summary](#10-summary)

---

## 1️⃣ Environment Variables

Environment variables store configuration values used by processes and applications.  
They define user settings, paths, and system behavior.

### 🔹 View Environment Variables

```bash
printenv
env
````

### 🔹 View a Specific Variable

```bash
echo $HOME
echo $PATH
echo $USER
```

### 🔹 Create Temporary Variable

```bash
MYVAR="HelloWorld"
echo $MYVAR
```

> Note: This variable is lost after you close the session.

### 🔹 Create Permanent Variable

Edit `~/.bashrc` or `~/.bash_profile`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
export PATH=$PATH:$JAVA_HOME/bin
```

Apply changes:

```bash
source ~/.bashrc
```

---

## 2️⃣ Process Scheduling with Cron Jobs

**Cron** is a time-based job scheduler in Unix/Linux that automates repetitive tasks.

### 🔹 Cron Syntax

```bash
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └─ Day of week (0 - 7)
│ │ │ └─── Month (1 - 12)
│ │ └───── Day of month (1 - 31)
│ └─────── Hour (0 - 23)
└───────── Minute (0 - 59)
```

### 🔹 View Cron Jobs

```bash
crontab -l
```

### 🔹 Edit Cron Jobs

```bash
crontab -e
```

### 🔹 Example Cron Jobs

| Task                        | Cron Schedule | Command                   |
| --------------------------- | ------------- | ------------------------- |
| Backup every day at 2AM     | `0 2 * * *`   | `/home/user/backup.sh`    |
| Clean temp files every hour | `0 * * * *`   | `rm -rf /tmp/*`           |
| Restart service weekly      | `0 3 * * 0`   | `systemctl restart nginx` |

### 🔹 System-Wide Cron Jobs

* `/etc/crontab` → system cron configuration
* `/etc/cron.hourly/`, `/etc/cron.daily/` → directory-based automation

---

## 3️⃣ System Monitoring and Logs

### 🔹 View System Load

```bash
uptime
```

Shows load averages for last 1, 5, and 15 minutes.

### 🔹 Real-Time Process Monitoring

```bash
top
htop
```

### 🔹 Disk and Memory Usage

```bash
df -h      # Disk space
du -sh *   # Directory size
free -m    # Memory usage
```

### 🔹 View System Logs

| Command                   | Description                |
| ------------------------- | -------------------------- |
| `journalctl`              | Show system logs (systemd) |
| `dmesg`                   | Kernel messages            |
| `tail -f /var/log/syslog` | Live system log            |
| `less /var/log/auth.log`  | Authentication logs        |

Example:

```bash
sudo journalctl -u ssh
```

---

## 4️⃣ Systemd Services

`systemd` is the modern init system that manages system startup and services.

### 🔹 Common Commands

| Command                       | Description         |
| ----------------------------- | ------------------- |
| `systemctl start <service>`   | Start a service     |
| `systemctl stop <service>`    | Stop a service      |
| `systemctl restart <service>` | Restart service     |
| `systemctl enable <service>`  | Start at boot       |
| `systemctl disable <service>` | Disable startup     |
| `systemctl status <service>`  | Show service status |

Example:

```bash
sudo systemctl status nginx
```

### 🔹 View Boot Performance

```bash
systemd-analyze
```

---

## 5️⃣ File Compression and Archiving

Linux supports multiple tools for compressing and packaging files.

| Tool    | Command Example               | Description     |
| ------- | ----------------------------- | --------------- |
| `tar`   | `tar -cvf archive.tar dir/`   | Create archive  |
|         | `tar -xvf archive.tar`        | Extract archive |
| `gzip`  | `gzip file.txt`               | Compress file   |
|         | `gunzip file.txt.gz`          | Decompress      |
| `zip`   | `zip archive.zip file1 file2` | Zip files       |
| `unzip` | `unzip archive.zip`           | Unzip files     |

### 🔹 Combine tar and gzip

```bash
tar -czvf archive.tar.gz /home/user/docs/
tar -xzvf archive.tar.gz
```

---

## 6️⃣ Disk and Storage Management

### 🔹 List Disks

```bash
lsblk
```

### 🔹 View Partitions

```bash
fdisk -l
```

### 🔹 Mount and Unmount Drives

```bash
sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb
```

### 🔹 Persistent Mount (auto-mount at boot)

Edit `/etc/fstab`:

```
/dev/sdb1  /mnt/usb  ext4  defaults  0 2
```

---

## 7️⃣ Networking and Firewalls

### 🔹 Enable/Disable Network Interface

```bash
sudo ip link set eth0 up
sudo ip link set eth0 down
```

### 🔹 View Routing Table

```bash
route -n
```

### 🔹 Firewall Management (UFW)

```bash
sudo ufw status
sudo ufw enable
sudo ufw allow 22/tcp
sudo ufw deny 80/tcp
```

### 🔹 Firewalld (CentOS/RHEL)

```bash
sudo firewall-cmd --list-all
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

---

## 8️⃣ System Security Essentials

### 🔹 Manage Users and Password Policies

```bash
sudo passwd -l user      # Lock user account
sudo passwd -u user      # Unlock user
```

### 🔹 Secure SSH Access

Edit `/etc/ssh/sshd_config`:

```
PermitRootLogin no
PasswordAuthentication no
```

Restart SSH service:

```bash
sudo systemctl restart ssh
```

### 🔹 Check Sudo Privileges

```bash
sudo -l
```

### 🔹 File Integrity Check

```bash
sha256sum file.iso
```

---

## 9️⃣ Performance Tuning Tips

| Area      | Command / Tool           | Description            |
| --------- | ------------------------ | ---------------------- |
| CPU       | `top`, `htop`, `mpstat`  | Monitor CPU usage      |
| Memory    | `free -h`, `vmstat`      | Memory usage overview  |
| Disk I/O  | `iostat`, `iotop`        | Disk read/write stats  |
| Network   | `iftop`, `ss`, `netstat` | Network utilization    |
| Boot time | `systemd-analyze blame`  | Identify slow services |

### 🔹 Kill High CPU Process

```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head
kill -9 <PID>
```

### 🔹 Tune Kernel Parameters

Edit `/etc/sysctl.conf`:

```
vm.swappiness=10
fs.file-max=2097152
net.ipv4.ip_forward=1
```

Apply changes:

```bash
sudo sysctl -p
```

---

## 🔟 Summary

You’ve now reached the **advanced level** of Linux knowledge 🎯

✅ Manage environment variables for development setups
✅ Automate system tasks using cron jobs
✅ Monitor system performance and logs
✅ Control and debug services with `systemd`
✅ Manage disks, networks, and security effectively
✅ Tune system performance for production workloads

---

> 🧭 **Next Topic:** [11-system-monitoring.md → Tools and Commands for Linux System Monitoring](./11-system-monitoring.md)