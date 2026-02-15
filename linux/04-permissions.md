# 🔐 Linux File Permissions & Security

## 📋 Table of Contents
- [1️⃣ Standard Permissions (rwx)](#1️⃣-standard-permissions-rwx)
- [2️⃣ Ownership (User & Group)](#2️⃣-ownership-user--group)
- [3️⃣ Advanced: Default Permissions (Umask)](#3️⃣-advanced-default-permissions-umask)
- [4️⃣ Granular Access: ACLs (Access Control Lists)](#4️⃣-granular-access-acls-access-control-lists)
- [5️⃣ Immutability: File Attributes (chattr)](#5️⃣-immutability-file-attributes-chattr)
- [6️⃣ Special Permissions (SUID, SGID, Sticky Bit)](#6️⃣-special-permissions-suid-sgid-sticky-bit)
- [7️⃣ Security Best Practices](#7️⃣-security-best-practices)

---

## 1️⃣ Standard Permissions (rwx)

Linux uses a **Bitmask** system.
- **Read (r)** = 4
- **Write (w)** = 2
- **Execute (x)** = 1

| Permission | Code | Description |
|:-----------|:-----|:------------|
| `rwx` | 7 | Full Access (4+2+1) |
| `rw-` | 6 | Read & Write (4+2) |
| `r-x` | 5 | Read & Execute (4+1) |
| `r--` | 4 | Read Only |

**Command:**
```bash
chmod 755 script.sh
# User: 7 (rwx), Group: 5 (r-x), Others: 5 (r-x)
```

---

## 2️⃣ Ownership (User & Group)

Every file belongs to a **User** and a **Group**.

```bash
# Change Owner to 'steve'
sudo chown steve file.txt

# Change Group to 'developers'
sudo chgrp developers file.txt

# Change Both (and recursive)
sudo chown -R steve:developers /var/www/html
```

---

## 3️⃣ Advanced: Default Permissions (Umask)

When you create a file, it gets default permissions (usually `644`). Why?
It involves **Subtraction** from a base score.

- **Base Limit**: `666` (Files), `777` (Folders).
- **Umask**: The bits to *REMOVE*.

**Calculation:**
`666` (Base) - `022` (Default Umask) = `644` (`rw-r--r--`).

**Scenario**: You want all new files in a folder to be private.
```bash
umask 077
touch secret.txt
ls -l secret.txt
# Output: -rw------- (600)
```

---

## 4️⃣ Granular Access: ACLs (Access Control Lists)

**Problem**: You want *Steve* to read-write, *Alice* to read-only, and *Bob* to have no access. Standard groups cannot do this three-way split.
**Solution**: Access Control Lists (ACL).

```bash
# 1. Give Steve RW access
setfacl -m u:steve:rw file.txt

# 2. Give Alice Read access
setfacl -m u:alice:r file.txt

# 3. View ACLs
getfacl file.txt
```

---

## 5️⃣ Immutability: File Attributes (chattr)

Even **Root** can accidentally delete files.
`chattr` (Change Attribute) adds a layer of protection above permissions.

| Attribute | Description |
|:----------|:------------|
| **`+i`** | **Immutable**. Cannot be modified, deleted, or renamed. Even by Root! |
| **`+a`** | **Append Only**. Can only add data (logs), cannot overwrite/delete. |

```bash
# Lock the file
sudo chattr +i production_config.conf

# Try to delete it
rm production_config.conf
# Output: Operation not permitted

# Unlock it
sudo chattr -i production_config.conf
```

---

## 6️⃣ Special Permissions (SUID, SGID, Sticky Bit)

Dangerous but necessary tools.

| Bit | Code | Function | Example |
|:----|:-----|:---------|:--------|
| **SUID** | `4000` | Run as **File Owner** (usually Root). | `passwd` (needs to edit /etc/shadow) |
| **SGID** | `2000` | Inherit **Folder Group**. | Shared Team Folders. |
| **Sticky**| `1000` | Only **Owner** can delete. | `/tmp` |

**Security Risk**: If a script has SUID and is owned by Root, anyone running it becomes Root for that split second.

---

## 7️⃣ Security Best Practices

1.  **Never use 777**: It allows anyone to inject malicious code.
2.  **Lock Audit Logs**: Use `chattr +a` on `/var/log` files so hackers can't wipe their tracks.
3.  **SSH Keys**: Must be `600` (`-rw-------`). If too open, SSH refuses to work.
4.  **Find SUID Files**: Audit your system for backdoors.
    ```bash
    find / -perm -4000 2>/dev/null
    ```

---

> 🧭 **Next:** [User & Group Management Deep Dive →](./05-users-and-groups.md)