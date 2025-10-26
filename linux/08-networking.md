# 🌐 Linux Networking Basics

Networking is one of the most critical parts of Linux system administration and development.  
Whether you’re deploying applications, troubleshooting servers, or using remote connections — networking skills are essential.

This document covers all **basic to intermediate networking concepts and commands** for developers.

---

## 🧭 Table of Contents

1. [Introduction](#1-introduction)
2. [Network Configuration Files](#2-network-configuration-files)
3. [Checking Network Interfaces](#3-checking-network-interfaces)
4. [IP Address Management](#4-ip-address-management)
5. [Testing Network Connectivity](#5-testing-network-connectivity)
6. [DNS and Hostname Commands](#6-dns-and-hostname-commands)
7. [Network Statistics and Monitoring](#7-network-statistics-and-monitoring)
8. [Port and Service Scanning](#8-port-and-service-scanning)
9. [SSH and Secure Remote Access](#9-ssh-and-secure-remote-access)
10. [Transferring Files Over Network](#10-transferring-files-over-network)
11. [Network Troubleshooting Tips](#11-network-troubleshooting-tips)
12. [Best Practices](#12-best-practices)
13. [Summary](#13-summary)

---

## 1️⃣ Introduction

Every Linux system connects to networks using interfaces (physical or virtual).  
Understanding how to manage, test, and troubleshoot these connections is vital.

Common network interface names:
- `eth0`, `eth1` → Wired Ethernet  
- `wlan0` → Wireless  
- `lo` → Loopback (local system, 127.0.0.1)

---

## 2️⃣ Network Configuration Files

| File | Description |
|-------|-------------|
| `/etc/network/interfaces` | Old-style network configuration (Debian) |
| `/etc/sysconfig/network-scripts/ifcfg-*` | Network scripts (CentOS/RHEL) |
| `/etc/resolv.conf` | DNS servers configuration |
| `/etc/hosts` | Static hostname to IP mappings |
| `/etc/hostname` | System hostname |
| `/etc/nsswitch.conf` | Name service switch configuration |

---

## 3️⃣ Checking Network Interfaces

### 🔹 Show Active Interfaces

```bash
ip link show
````

or older command:

```bash
ifconfig
```

Example output:

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 192.168.1.10/24 brd 192.168.1.255 scope global eth0
```

### 🔹 Show IP Addresses Only

```bash
ip addr show
```

or:

```bash
hostname -I
```

---

## 4️⃣ IP Address Management

### 🔹 Assign IP Temporarily

```bash
sudo ip addr add 192.168.1.20/24 dev eth0
```

### 🔹 Remove IP Address

```bash
sudo ip addr del 192.168.1.20/24 dev eth0
```

### 🔹 Bring Interface Up or Down

```bash
sudo ip link set eth0 up
sudo ip link set eth0 down
```

---

## 5️⃣ Testing Network Connectivity

### 🔹 Ping

Test if a host is reachable:

```bash
ping google.com
ping -c 4 8.8.8.8
```

### 🔹 Traceroute

Check the route packets take:

```bash
traceroute google.com
```

Install if missing:

```bash
sudo apt install traceroute
```

### 🔹 Trace Path (alternative)

```bash
tracepath google.com
```

---

## 6️⃣ DNS and Hostname Commands

### 🔹 Check DNS Resolution

```bash
nslookup openai.com
```

or:

```bash
dig openai.com
```

### 🔹 Reverse DNS Lookup

```bash
dig -x 8.8.8.8
```

### 🔹 Check Current Hostname

```bash
hostname
```

### 🔹 Change Hostname Temporarily

```bash
sudo hostname newhostname
```

### 🔹 Change Hostname Permanently

On modern systems:

```bash
sudo hostnamectl set-hostname myserver
```

---

## 7️⃣ Network Statistics and Monitoring

### 🔹 Display Interface Statistics

```bash
netstat -i
```

or:

```bash
ip -s link
```

### 🔹 Display Open Connections

```bash
netstat -tulnp
```

| Option | Meaning                |
| ------ | ---------------------- |
| `-t`   | TCP connections        |
| `-u`   | UDP connections        |
| `-l`   | Listening sockets      |
| `-n`   | Show numeric addresses |
| `-p`   | Show process ID/name   |

Example:

```
tcp  0  0 0.0.0.0:22  0.0.0.0:*  LISTEN  1001/sshd
```

### 🔹 Modern Equivalent of netstat

```bash
ss -tuln
```

### 🔹 Network Traffic Monitor

Install `iftop` or `nload`:

```bash
sudo apt install iftop nload
sudo iftop -i eth0
```

---

## 8️⃣ Port and Service Scanning

### 🔹 Check if a Port is Listening

```bash
sudo netstat -tuln | grep 8080
# OR
sudo ss -tuln | grep 8080
```

### 🔹 Test Port Connectivity

```bash
telnet <host> <port>
# Example
telnet localhost 3306
```

If telnet is not available:

```bash
nc -zv localhost 3306
```

---

## 9️⃣ SSH and Secure Remote Access

SSH (Secure Shell) is the standard protocol for **remote administration** and **file transfer** on Linux systems.

### 🔹 Connect to a Remote Server

```bash
ssh user@hostname
```

### 🔹 Specify a Custom Port

```bash
ssh -p 2222 user@hostname
```

### 🔹 Copy Files via SCP

```bash
scp file.txt user@remote_host:/home/user/
```

### 🔹 Copy Directories Recursively

```bash
scp -r /var/www/ user@192.168.1.50:/backup/
```

### 🔹 Using SSH Key Authentication

1. Generate key pair:

   ```bash
   ssh-keygen -t rsa -b 4096
   ```
2. Copy key to remote host:

   ```bash
   ssh-copy-id user@hostname
   ```

Now you can log in passwordlessly.

---

## 🔟 Transferring Files Over Network

### 🔹 Using SCP (Secure Copy)

```bash
scp myfile.txt user@remote:/home/user/
```

### 🔹 Using rsync (Recommended)

Efficient file synchronization tool:

```bash
rsync -avz /source/dir/ user@remote:/destination/
```

Options:

* `-a` → archive mode (preserve attributes)
* `-v` → verbose
* `-z` → compress data
* `--delete` → remove deleted files on destination

---

## 11️⃣ Network Troubleshooting Tips

| Command                           | Purpose                       |
| --------------------------------- | ----------------------------- |
| `ping`                            | Test connectivity             |
| `traceroute`                      | Identify route to destination |
| `netstat` / `ss`                  | Check open ports and services |
| `ifconfig` / `ip`                 | Check interface configuration |
| `nslookup` / `dig`                | DNS lookup                    |
| `nload` / `iftop`                 | Monitor live traffic          |
| `route -n`                        | Display routing table         |
| `systemctl status NetworkManager` | Check network service status  |

---

### 🔹 Restart Networking Services

```bash
sudo systemctl restart NetworkManager
```

or on older systems:

```bash
sudo service networking restart
```

---

### 🔹 Flush DNS Cache

```bash
sudo systemd-resolve --flush-caches
```

---

### 🔹 Reset Network Interface

```bash
sudo ip link set eth0 down
sudo ip link set eth0 up
```

---

## 12️⃣ Best Practices

✅ Keep network configurations documented (`/etc/network/interfaces`, `/etc/hosts`)
✅ Use static IPs for servers, dynamic (DHCP) for clients
✅ Always secure remote access using **SSH keys**, not passwords
✅ Regularly check open ports to avoid unnecessary exposure
✅ Use firewalls (`ufw`, `firewalld`) to control traffic
✅ Monitor uptime and traffic regularly using `iftop`, `nload`, or `vnstat`

---

## 13️⃣ Summary

* **Networking** in Linux is managed using tools like `ip`, `ping`, `netstat`, and `ss`.
* **SSH** allows secure remote access and file transfer.
* Tools like `rsync` and `scp` make moving files fast and secure.
* DNS and routing configuration files define how your system communicates.
* Good network hygiene = stable, secure, and reliable systems.

---

> 🧭 **Next Topic:** [09-shell-scripting.md → Automating Tasks with Bash Scripts](./09-shell-scripting.md)