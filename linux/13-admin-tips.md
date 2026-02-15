# 🛠️ Linux Admin Tips & Disaster Recovery

## 📋 Table of Contents
- [1️⃣ Disaster Recovery: Resetting Root Password](#1️⃣-disaster-recovery-resetting-root-password)
- [2️⃣ Boot Issues: Analyzing Journal](#2️⃣-boot-issues-analyzing-journal)
- [3️⃣ Kernel Tuning (sysctl)](#3️⃣-kernel-tuning-sysctl)
- [4️⃣ Security: SSH Hardening](#4️⃣-security-ssh-hardening)
- [5️⃣ Hardware Debugging](#5️⃣-hardware-debugging)
- [6️⃣ The "Magic" SysRq Key](#6️⃣-the-magic-sysrq-key)

---

## 1️⃣ Disaster Recovery: Resetting Root Password

Forgot the Root password? Physical access is required.

1.  **Reboot** the machine.
2.  At the **GRUB Menu**, select your kernel and press `e` (Edit).
3.  Find the line starting with `linux ...`.
4.  Go to the end of that line and append:
    ```
    init=/bin/bash
    ```
5.  Press `Ctrl + X` (or F10) to boot.
6.  You are now in a root shell. The disk is Read-Only.
    ```bash
    mount -o remount,rw /
    passwd root
    # (Enter new password)
    exec /sbin/init
    ```

---

## 2️⃣ Boot Issues: Analyzing Journal

Server booted but services failed?
`journalctl` is your best friend.

```bash
# Show logs from the current boot only
journalctl -b

# Show only Errors (Critical)
journalctl -p 3 -xb

# Show boot timing (Who is slow?)
systemd-analyze blame
```

---

## 3️⃣ Kernel Tuning (sysctl)

Optimize your server without rebooting.
Config: `/etc/sysctl.conf`

**Common Tunes:**
```ini
# 1. Reduce Swap usage (Default 60. 10 = only swap if RAM absolute full)
vm.swappiness=10

# 2. Increase Open Files Limit (For High-Load Web Servers)
fs.file-max=100000

# 3. Allow more connections (Backlog)
net.core.somaxconn=1024

# 4. Enable IP Forwarding (Router/VPN mode)
net.ipv4.ip_forward=1
```

**Apply Changes:**
```bash
sudo sysctl -p
```

---

## 4️⃣ Security: SSH Hardening

Default SSH is vulnerable to brute force.

**Edit `/etc/ssh/sshd_config`:**
```ssh
PermitRootLogin no          # Never log in as root directly
PasswordAuthentication no   # Use SSH Keys only!
Port 2222                   # Change default port (reduces noise)
```

**Install Fail2Ban:**
Automatically blocks IPs that fail login 3 times.
```bash
sudo apt install fail2ban
```

---

## 5️⃣ Hardware Debugging

Is it the OS or the Hardware?

| Tool | Purpose |
|:-----|:--------|
| `lspci` | List PCI devices (Graphics card, Network card). |
| `lsusb` | List USB devices. |
| `dmidecode` | **Bios & Motherboard Info** (Serial Numbers, RAM slots). |
| `lshw` | List Hardware (Detailed tree). |
| `smartctl` | **Check Disk Health** (S.M.A.R.T data). |

**Scenario**: "What RAM stick is in Slot 2?"
```bash
sudo dmidecode -t memory
```

---

## 6️⃣ The "Magic" SysRq Key

System frozen? SSH dead? Keyboard Unresponsive?
Before pulling the plug, try the **Magic SysRq**.

1.  Hold `Alt + SysRq (PrintScreen)`.
2.  Type `R E I S U B` (slowly).

| Letter | Action |
|:-------|:-------|
| **R** | Switch keyboard from Raw to XLATE (Control). |
| **E** | Send SIGTERM to all (Nice kill). |
| **I** | Send SIGKILL to all (Force kill). |
| **S** | Sync (Flush data to disk). |
| **U** | Unmount (Remount read-only). |
| **B** | Reboot. |

Mnemonic: "**R**eboot **E**ven **I**f **S**ystem **U**tterly **B**roken".

---
