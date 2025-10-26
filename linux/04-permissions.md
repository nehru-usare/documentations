# 🔐 Linux File Permissions and Ownership

Linux is a **multi-user operating system**, meaning multiple people can use the system at the same time — each with specific access rights.  
File **permissions and ownership** determine **who can read, modify, or execute** files and directories.

---

## 🧭 Table of Contents
1. [What are Permissions?](#1-what-are-permissions)
2. [File Permission Format](#2-file-permission-format)
3. [Permission Types](#3-permission-types)
4. [Changing Permissions (chmod)](#4-changing-permissions-chmod)
5. [Ownership (chown and chgrp)](#5-ownership-chown-and-chgrp)
6. [Directory Permissions](#6-directory-permissions)
7. [Recursive Permissions](#7-recursive-permissions)
8. [Special Permissions (SUID, SGID, Sticky Bit)](#8-special-permissions-suid-sgid-sticky-bit)
9. [Viewing Permission Details](#9-viewing-permission-details)
10. [Common Permission Scenarios](#10-common-permission-scenarios)
11. [Best Practices](#11-best-practices)
12. [Cheatsheet](#12-cheatsheet)
13. [Summary](#13-summary)

---

## 1️⃣ What are Permissions?

Permissions are **rules that control file access**.  
They define what **actions a user or group can perform** on files and directories.

Every file or directory in Linux has:
- An **owner (user)**
- A **group**
- **Permission bits** (for user, group, and others)

---

## 2️⃣ File Permission Format

Command:
```bash
ls -l
````

Output:

```
-rwxr-xr--
```

### Breakdown:

| Position | Meaning                                       | Example |
| -------- | --------------------------------------------- | ------- |
| 1st      | File type (`-` file, `d` directory, `l` link) | `-`     |
| 2-4      | Owner (user) permissions                      | `rwx`   |
| 5-7      | Group permissions                             | `r-x`   |
| 8-10     | Others (world) permissions                    | `r--`   |

So,
`-rwxr-xr--` means:

* Owner → read, write, execute
* Group → read, execute
* Others → read only

---

## 3️⃣ Permission Types

| Symbol | Permission | Description                                   | Numeric Value |
| ------ | ---------- | --------------------------------------------- | ------------- |
| `r`    | Read       | View contents of file or list directory       | 4             |
| `w`    | Write      | Modify or delete file; add files to directory | 2             |
| `x`    | Execute    | Run as program or enter directory             | 1             |
| `-`    | None       | No permission                                 | 0             |

### Numeric Example:

`rwxr-xr--`
→ Owner = 7 (4+2+1)
→ Group = 5 (4+0+1)
→ Others = 4 (4+0+0)
**Equivalent numeric code:** `754`

---

## 4️⃣ Changing Permissions (chmod)

You can modify permissions in **symbolic** or **numeric** mode.

---

### 🔹 Symbolic Mode Examples

```bash
chmod u+x script.sh        # Add execute permission for user
chmod g-w file.txt         # Remove write permission for group
chmod o+r notes.txt        # Add read for others
chmod a-x run.sh           # Remove execute for all users
```

| Symbol | Meaning               |
| ------ | --------------------- |
| `u`    | User (owner)          |
| `g`    | Group                 |
| `o`    | Others                |
| `a`    | All (u+g+o)           |
| `+`    | Add permission        |
| `-`    | Remove permission     |
| `=`    | Set exact permissions |

---

### 🔹 Numeric Mode Examples

```bash
chmod 777 script.sh   # Everyone can read, write, execute
chmod 755 script.sh   # Owner full, others read & execute
chmod 644 file.txt    # Owner read/write, others read only
chmod 600 id_rsa      # Only owner can read/write (private)
```

| Octal | Permissions | Common Usage                           |
| ----- | ----------- | -------------------------------------- |
| 777   | `rwxrwxrwx` | Full access (avoid for sensitive data) |
| 755   | `rwxr-xr-x` | Executables or shared directories      |
| 700   | `rwx------` | Private scripts                        |
| 644   | `rw-r--r--` | Documents, configs                     |
| 600   | `rw-------` | Private keys, logs                     |

---

## 5️⃣ Ownership (chown and chgrp)

Each file has an **owner** and a **group** assigned.

Check ownership:

```bash
ls -l
```

Output:

```
-rw-r--r-- 1 alex devteam 1200 Oct 26 14:20 app.log
```

| Field   | Description |
| ------- | ----------- |
| alex    | File owner  |
| devteam | Group owner |

---

### 🔧 Changing Ownership

| Command                 | Description            |
| ----------------------- | ---------------------- |
| `chown user file`       | Change file owner      |
| `chown user:group file` | Change owner and group |
| `chgrp group file`      | Change group ownership |

Examples:

```bash
sudo chown john file.txt
sudo chown john:developers /opt/project/
sudo chgrp devteam report.pdf
```

### 🔁 Recursive Ownership

```bash
sudo chown -R user:group /var/www/
```

---

## 6️⃣ Directory Permissions

Permissions on directories behave differently:

| Permission | Directory Behavior                                          |
| ---------- | ----------------------------------------------------------- |
| **r**      | Allows listing directory contents (`ls`)                    |
| **w**      | Allows creating/deleting/renaming files                     |
| **x**      | Allows entering directory (`cd`) and accessing files inside |

### Example:

```bash
chmod 755 /var/www
```

Owner → full, Others → read and execute only

---

## 7️⃣ Recursive Permissions

To change permissions recursively for a directory and all its contents:

```bash
chmod -R 755 /var/www/
chown -R devuser:devgroup /var/www/
```

---

## 8️⃣ Special Permissions (SUID, SGID, Sticky Bit)

Linux supports **three advanced permission bits** for special scenarios.

| Special Bit    | Symbol | Octal Value | Description                            |
| -------------- | ------ | ----------- | -------------------------------------- |
| **SUID**       | `s`    | 4           | Execute file as the owner              |
| **SGID**       | `s`    | 2           | Inherit group ownership                |
| **Sticky Bit** | `t`    | 1           | Only owner can delete in shared folder |

---

### 🔹 SUID (Set User ID)

When applied to a file, it executes **as the file owner**, not the user who runs it.

Example:

```bash
chmod 4755 /usr/bin/passwd
```

Check:

```bash
ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 54256 Aug 10  /usr/bin/passwd
```

`rws` → SUID bit is active.
This allows normal users to run `passwd` with root privileges to change passwords.

---

### 🔹 SGID (Set Group ID)

For directories:
Files created inside will **inherit the group** of the parent directory.

```bash
chmod 2755 /shared/team
ls -ld /shared/team
drwxr-sr-x 2 john devgroup 4096 Oct 26 /shared/team
```

Now all new files inside `/shared/team` belong to group `devgroup`.

---

### 🔹 Sticky Bit

Used for **public directories** like `/tmp`
Prevents users from deleting other users’ files.

```bash
chmod 1777 /tmp
ls -ld /tmp
drwxrwxrwt 10 root root 4096 Oct 26 /tmp
```

`t` at the end → Sticky bit active.

---

## 9️⃣ Viewing Permission Details

Use the `stat` command for in-depth details:

```bash
stat example.txt
```

Output:

```
File: example.txt
Access: (0644/-rw-r--r--)  Uid: (1000/alex)   Gid: (1000/alex)
Access: 2025-10-26 12:20:00
Modify: 2025-10-26 12:18:00
Change: 2025-10-26 12:19:00
```

---

## 🔟 Common Permission Scenarios

| Scenario                         | Command                      |
| -------------------------------- | ---------------------------- |
| Make a script executable         | `chmod +x deploy.sh`         |
| Make directory publicly readable | `chmod 755 /var/www`         |
| Allow team write access          | `chmod g+w /opt/team`        |
| Secure SSH keys                  | `chmod 600 ~/.ssh/id_rsa`    |
| Set upload folder permissions    | `chmod 777 /var/www/uploads` |

---

## 11️⃣ Best Practices

✅ Always follow **least privilege principle** (give only needed access)
✅ Avoid using `777` (everyone full control)
✅ For shared folders, use **SGID** or **group ownership**
✅ Regularly audit permissions (`find / -perm 777`)
✅ Use version control for configuration files
✅ Never edit `/etc/sudoers` manually — use `visudo`

---

## 12️⃣ Cheatsheet

| Numeric | Permission  | Meaning                       |
| ------- | ----------- | ----------------------------- |
| 600     | `rw-------` | Only user can read/write      |
| 644     | `rw-r--r--` | Public readable file          |
| 700     | `rwx------` | Private executable            |
| 755     | `rwxr-xr-x` | Public executable             |
| 775     | `rwxrwxr-x` | Shared directory              |
| 777     | `rwxrwxrwx` | Everyone full access (unsafe) |

---

## 13️⃣ Summary

* Each file has **three permission sets**: user, group, and others.
* Permissions are represented symbolically (`rwx`) or numerically (e.g. `755`).
* Ownership defines **who controls the file**.
* Use `chmod`, `chown`, and `chgrp` to manage permissions.
* Advanced permissions (SUID, SGID, Sticky Bit) enable collaborative and secure environments.

---

> 🧭 **Next Topic:** [05-users-and-groups.md → User and Group Management](./05-users-and-groups.md)