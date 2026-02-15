# 🌐 Linux Networking & Architecture

## 📋 Table of Contents
- [1️⃣ Networking Basics (Interfaces & IPs)](#1️⃣-networking-basics-interfaces--ips)
- [2️⃣ Architectural Theory: The OSI Model](#2️⃣-architectural-theory-the-osi-model)
- [3️⃣ Internal Mechanics: DNS Resolution Flow](#3️⃣-internal-mechanics-dns-resolution-flow)
- [4️⃣ IP Addressing & Subnetting (CIDR)](#4️⃣-ip-addressing--subnetting-cidr)
- [5️⃣ Routing & Gateways](#5️⃣-routing--gateways)
- [6️⃣ Connectivity Troubleshooting Tools](#6️⃣-connectivity-troubleshooting-tools)
- [7️⃣ Security: Ports & Firewalls](#7️⃣-security-ports--firewalls)
- [8️⃣ File Transfer (SCP, Rsync)](#8️⃣-file-transfer-scp-rsync)

---

## 1️⃣ Networking Basics (Interfaces & IPs)

| Command | New (iproute2) | Old (net-tools) | Description |
|:--------|:---------------|:----------------|:------------|
| **Show IP** | `ip a` | `ifconfig` | Show interfaces and IP addresses. |
| **Up/Down** | `ip link set eth0 up` | `ifconfig eth0 up` | Enable/Disable interface. |
| **Gateway** | `ip route` | `route -n` | Show routing table. |
| **Stats** | `ip -s link` | `netstat -i` | Packet counts / Errors. |

---

## 2️⃣ Architectural Theory: The OSI Model

Understanding which layer a tool tests is crucial for debugging.

| Layer | Name | Protocol | Linux Tool | Failure Symptom |
|:------|:-----|:---------|:-----------|:----------------|
| **L7** | Application | HTTP, SSH, DNS | `curl`, `wget` | "404 Not Found", "Connection Refused" |
| **L4** | Transport | TCP, UDP | `netstat`, `telnet`, `nc` | Timeout, Port Closed |
| **L3** | Network | IP, ICMP | `ping`, `traceroute` | "Destination Host Unreachable" |
| **L2** | Data Link | Ethernet, MAC | `ip link`, `arp` | No Link light, Packet loss |
| **L1** | Physical | Cables, WiFi | `ethtool` | "No carrier" |

**Troubleshooting Workflow**: Start at **L1** (Is cable plugged?) -> **L3** (Can I ping?) -> **L4** (Is port open?) -> **L7** (Is app crashing?).

---

## 3️⃣ Internal Mechanics: DNS Resolution Flow

When you type `ping google.com`, Linux follows this exact order:

1.  **Local Cache**: (If `systemd-resolved` is running).
2.  **`/etc/hosts`**: Static overrides.
    ```
    127.0.0.1 localhost
    192.168.1.50 dev-server
    ```
3.  **`/etc/nsswitch.conf`**: Tells Linux "Check 'files' (hosts) first, then 'dns'".
    ```
    hosts: files dns
    ```
4.  **`/etc/resolv.conf`**: The actual DNS servers.
    ```
    nameserver 8.8.8.8
    ```

---

## 4️⃣ IP Addressing & Subnetting (CIDR)

We use **CIDR** (Classless Inter-Domain Routing) notation.

| CIDR | Subnet Mask | Total IPs | Use Case |
|:-----|:------------|:----------|:---------|
| **/32** | 255.255.255.255 | 1 | A single specific PC. |
| **/24** | 255.255.255.0 | 256 | Standard Home/Office LAN. |
| **/16** | 255.255.0.0 | 65,536 | Large AWS VPC. |
| **/0** | 0.0.0.0 | All | "The Internet" (Everything). |

---

## 5️⃣ Routing & Gateways

Packets need a map to leave your computer.
**Command**: `ip route`

```
default via 192.168.1.1 dev eth0  <-- The Gateway (Router)
192.168.1.0/24 dev eth0           <-- Local LAN (No Gateway needed)
```

**Logic**:
- "Is dest in `192.168.1.0/24`?" -> Yes? Send directly on `eth0`.
- "No?" -> Send to Default Gateway (`192.168.1.1`).

---

## 6️⃣ Connectivity Troubleshooting Tools

| Tool | Command | Purpose |
|:-----|:--------|:--------|
| **Ping** | `ping 8.8.8.8` | Check L3 connectivity (ICMP). |
| **Netcat** | `nc -vz google.com 443` | Check if **TCP Port** is open (L4). |
| **Traceroute**| `traceroute google.com` | See every router hop along the path. |
| **MTR** | `mtr google.com` | Ping + Traceroute combined (Real-time). |
| **Dig** | `dig google.com` | Debug DNS records. |

---

## 7️⃣ Security: Ports & Firewalls

### Open Ports
Hackers scan for listening applications.
```bash
# Show listening ports
sudo ss -tulnp
# -t (TCP), -u (UDP), -l (Listening), -n (Numeric), -p (Process)
```

### Firewalls
1.  **UFW (Uncomplicated Firewall)** - Ubuntu/Debian.
    ```bash
    sudo ufw allow 22/tcp
    sudo ufw enable
    ```
2.  **Firewalld** - RHEL/CentOS.
    ```bash
    sudo firewall-cmd --add-port=80/tcp --permanent
    sudo firewall-cmd --reload
    ```

---

## 8️⃣ File Transfer (SCP, Rsync)

**SCP (Secure Copy)**:
Simple file copy over SSH.
```bash
scp file.txt user@server:/home/user/
```

**Rsync (Remote Sync)**:
Smarter. Only copies *changed* parts. Resume support.
```bash
# -a (Archive), -v (Verbose), -z (Compress), -P (Progress bar)
rsync -avzP source_folder/ user@server:/dest/
```

---

> 🧭 **Next:** [Shell Scripting & Automation →](./09-shell-scripting.md)