# 🚀 Advanced Linux Administration

## 📋 Table of Contents
- [1️⃣ Environment Variable Loading Order](#1️⃣-environment-variable-loading-order)
- [2️⃣ Cron vs Systemd Timers](#2️⃣-cron-vs-systemd-timers)
- [3️⃣ Managing Logs (Logrotate)](#3️⃣-managing-logs-logrotate)
- [4️⃣ SSH Tunnels & Port Forwarding](#4️⃣-ssh-tunnels--port-forwarding)
- [5️⃣ Disk Management (Mounts & Swap)](#5️⃣-disk-management-mounts--swap)
- [6️⃣ Background Jobs (nohup & disown)](#6️⃣-background-jobs-nohup--disown)

---

## 1️⃣ Environment Variable Loading Order

"Why is my `JAVA_HOME` not set?"
Because you put it in the wrong file.

| File | Loaded When? | Use Case |
|:-----|:-------------|:---------|
| `/etc/environment` | **Always** (System-wide). | Global variables (`HTTP_PROXY`). |
| `/etc/profile` | Login Shells (System-wide). | Global scripts. |
| `~/.bash_profile` | **Login Shell** (SSH, TTY). | User-specific config. |
| `~/.bashrc` | **Non-Login Shell** (New Terminal Tab). | Aliases, Prompt colors. |

**The Chain**:
1.  Login via SSH -> Loads `~/.bash_profile`.
2.  `~/.bash_profile` usually contains a line: `source ~/.bashrc`.
3.  So, putting everything in `~/.bashrc` is usually the safest bet for single-user dev machines.

---

## 2️⃣ Cron vs Systemd Timers

### 🔹 Classic Cron (`crontab -e`)
Good for simple scripts.
```bash
# Run backup at 3 AM daily
0 3 * * * /home/user/backup.sh
```

### 🔹 Systemd Timers (The Modern Way)
Better logging, dependency management, and "missed job" handling.

**1. Create Service (`backup.service`)**
```ini
[Service]
ExecStart=/home/user/backup.sh
```

**2. Create Timer (`backup.timer`)**
```ini
[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true  # Run immediately if computer was off at 3 AM
```

---

## 3️⃣ Managing Logs (Logrotate)

Logs grow forever until the disk fills up. **Logrotate** fixes this.
Config: `/etc/logrotate.d/nginx`

```nginx
/var/log/nginx/*.log {
    daily
    rotate 14       # Keep 14 days
    compress        # Gzip old logs
    delaycompress   # Don't gzip the most recent old log
    missingok       # Don't panic if log is missing
    notifempty      # Don't rotate empty logs
}
```

**Force run to test:**
```bash
sudo logrotate -f /etc/logrotate.conf
```

---

## 4️⃣ SSH Tunnels & Port Forwarding

Access services behind a firewall without VPN.

### 🔹 Local Port Forwarding (`-L`)
"Access Remote DB (Port 3306) on my Local Laptop (Port 5000)".

```bash
ssh -L 5000:localhost:3306 user@remote-server
```
Now point your local SQL Client to `localhost:5000`.

### 🔹 Remote Port Forwarding (`-R`)
"Expose my Local Web App (Port 3000) to the Public Server (Port 8080)".

```bash
ssh -R 8080:localhost:3000 user@public-server
```
Now anyone visiting `public-server:8080` sees your laptop's app.

### 🔹 Dynamic SOCKS Proxy (`-D`)
"Route ALL my browser traffic through the server".

```bash
ssh -D 9090 user@remote-server
```
Set Browser Proxy to `localhost:9090` (SOCKS5).

---

## 5️⃣ Disk Management (Mounts & Swap)

### 🔹 Fstab (Permanent Mounts)
File: `/etc/fstab`.
UUID is safer than `/dev/sdb1` because device names can change.

```bash
# Get UUID
blkid

# Add to fstab
UUID=550e8400-e29b-41d4 /mnt/data ext4 defaults 0 2
```

### 🔹 Swap File (Emergency RAM)
If RAM is full, Linux crashes. Swap prevents this (but is slow).

```bash
# Create 1GB file
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

## 6️⃣ Background Jobs (nohup & disown)

If you close SSH, your script dies.

**Solution 1: Nohup**
```bash
nohup python app.py &
# Output goes to 'nohup.out'
```

**Solution 2: Disown**
```bash
python app.py &
# [1] 12345
disown -h %1
```

**Solution 3: Screen / Tmux (Best)**
Use a terminal multiplexer to keep sessions alive forever.

---

> 🧭 **Next:** [System Monitoring & Performance Tuning →](./11-system-monitoring.md)