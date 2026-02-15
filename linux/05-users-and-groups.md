# 👥 Linux Users, Groups, and Authentication

## 📋 Table of Contents
- [1️⃣ User & Group Basics](#1️⃣-user--group-basics)
- [2️⃣ Internal Mechanics: The 4 Critical Files](#2️⃣-internal-mechanics-the-4-critical-files)
- [3️⃣ User Management (CLI)](#3️⃣-user-management-cli)
- [4️⃣ Sudo & Privilege Escalation](#4️⃣-sudo--privilege-escalation)
- [5️⃣ Deep Dive: PAM (Pluggable Authentication Modules)](#5️⃣-deep-dive-pam-pluggable-authentication-modules)
- [6️⃣ System Limits (ulimit)](#6️⃣-system-limits-ulimit)
- [7️⃣ Advanced: User Namespaces (Container Basics)](#7️⃣-advanced-user-namespaces-container-basics)

---

## 1️⃣ User & Group Basics
Linux Permissions are based on **UIDs** (User IDs) and **GIDs** (Group IDs).
- **Root (0)**: The Superuser.
- **System Users (1-999)**: Service accounts (e.g., `www-data`, `mysql`). They usually have `/usr/sbin/nologin` shell.
- **Regular Users (1000+)**: Human accounts.

---

## 2️⃣ Internal Mechanics: The 4 Critical Files

When you run `adduser steve`, Linux edits these files:

| File | Permission | Purpose | Structure |
|:-----|:-----------|:--------|:----------|
| `/etc/passwd` | World Readable | User Info | `root:x:0:0:root:/root:/bin/bash` |
| `/etc/shadow` | **Root Only** | Password Hashes | `root:$6$Kj...:18500:0:99999:7:::` |
| `/etc/group` | World Readable | Group Info | `sudo:x:27:steve` |
| `/etc/gshadow`| **Root Only** | Group Passwords | (Rarely used today) |

> **Hackers love `/etc/passwd`** because it lists every user on the system, helping them target brute-force attacks.

---

## 3️⃣ User Management (CLI)

| Action | Command | Note |
|:-------|:--------|:-----|
| **Create** | `useradd -m -s /bin/bash steve` | `-m` creates `/home/steve`. |
| **Password**| `passwd steve` | Interactive prompt. |
| **Modify** | `usermod -aG sudo steve` | Add to 'sudo' group. |
| **Delete** | `userdel -r steve` | `-r` removes Home folder. |
| **Lock** | `usermod -L steve` | Disables login immediately. |

---

## 4️⃣ Sudo & Privilege Escalation

**Sudo** (SuperUser DO) allows a user to borrow Root privileges.
Config is in `/etc/sudoers`. **NEVER edit this manually.** Always use `visudo`.

**Secure Sudoers Config:**
```bash
# Allow 'steve' to restart Nginx WITHOUT a password
steve ALL=(root) NOPASSWD: /usr/sbin/systemctl restart nginx
```

---

## 5️⃣ Deep Dive: PAM (Pluggable Authentication Modules)

How does Linux know your password is correct? **PAM**.
Located in `/etc/pam.d/`.

**The Flow:**
1.  **Login** program asks for password.
2.  **PAM** checks `/etc/shadow`.
3.  **PAM** checks policies (e.g., `pam_tally2` for failed attempts).
4.  **PAM** enforces restrictions (e.g., `pam_time` limits login hours).

**Scenario**: You want to block a user after 3 failed password attempts.
*   Edit `/etc/pam.d/common-auth` and add `pam_tally2.so deny=3`.

---

## 6️⃣ System Limits (ulimit)

What if a user runs a script that spawns 1,000,000 processes? The server crashes.
**Solution**: Limits.

**View current limits:**
```bash
ulimit -a
```

**Permanent Configuration (`/etc/security/limits.conf`):**
```ini
# <domain> <type>  <item>  <value>
steve      hard    nproc   50      # Steve can max run 50 processes
@student   hard    fsize   100000  # 'student' group max file size 100MB
```

---

## 7️⃣ Advanced: User Namespaces (Container Basics)

This is how **Docker** works.
In a Container, you might be **Root** (UID 0).
But on the Host, you map to **Nobody** (UID 65534).

**User Namespaces** allow a process to have a different set of UIDs than the host system, providing isolation.

```bash
# Enter a new user namespace (requires kernel support)
unshare --user --map-root-user
whoami
# Output: root (But you are faked!)
```

---

> 🧭 **Next:** [Process Management Deep Dive →](./06-process-management.md)