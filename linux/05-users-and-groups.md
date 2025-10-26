# 👥 Linux Users and Groups

Linux is a **multi-user operating system**, meaning multiple users can work on the same system simultaneously.  
To manage access, Linux organizes users into **accounts** and **groups** — with **permissions** determining what each user can do.

Understanding how to manage users and groups is critical for:
- 🧩 Security  
- 👨‍💻 Collaboration  
- ⚙️ System Administration

---

## 🧭 Table of Contents

1. [Understanding Users and Groups](#1-understanding-users-and-groups)
2. [User Information and Files](#2-user-information-and-files)
3. [Group Information and Files](#3-group-information-and-files)
4. [User Management Commands](#4-user-management-commands)
5. [Group Management Commands](#5-group-management-commands)
6. [Managing Passwords and Security](#6-managing-passwords-and-security)
7. [Switching Users and sudo Access](#7-switching-users-and-sudo-access)
8. [User Profiles and Default Settings](#8-user-profiles-and-default-settings)
9. [Advanced User Administration](#9-advanced-user-administration)
10. [Best Practices](#10-best-practices)
11. [Summary](#11-summary)

---

## 1️⃣ Understanding Users and Groups

Linux uses two fundamental security entities:
- **Users** → Individual accounts (e.g., `root`, `john`, `developer`)
- **Groups** → Collections of users that share permissions

### 🧠 Key Concepts

| Concept | Description |
|----------|-------------|
| **User Account** | Identity used to log in and perform actions |
| **Root User** | Superuser with unrestricted access |
| **Normal User** | Limited permissions (for daily use) |
| **Group** | Logical collection of users sharing permissions |
| **UID** | User ID — unique identifier for a user |
| **GID** | Group ID — unique identifier for a group |

---

## 2️⃣ User Information and Files

Linux stores user information in system configuration files.

| File | Description |
|-------|--------------|
| `/etc/passwd` | Basic user account info |
| `/etc/shadow` | Encrypted passwords and aging info |
| `/etc/group` | Group definitions |
| `/etc/login.defs` | Default login settings (UID range, password policies) |

---

### 📄 Example: `/etc/passwd`

Each line represents one user:
```

john:x:1001:1001:John Smith:/home/john:/bin/bash

```

| Field | Meaning |
|--------|----------|
| `john` | Username |
| `x` | Password placeholder (actual stored in `/etc/shadow`) |
| `1001` | UID (User ID) |
| `1001` | GID (Primary Group ID) |
| `John Smith` | User description |
| `/home/john` | Home directory |
| `/bin/bash` | Default shell |

---

## 3️⃣ Group Information and Files

Groups allow permissions to be shared among multiple users.

### 📄 Example: `/etc/group`
```

devteam:x:1002:john,alice

````

| Field | Description |
|--------|-------------|
| `devteam` | Group name |
| `x` | Password placeholder |
| `1002` | GID (Group ID) |
| `john,alice` | Members of this group |

---

## 4️⃣ User Management Commands

### 👤 Creating Users

```bash
sudo adduser devuser
````

This command:

* Creates `/home/devuser/`
* Adds user entry in `/etc/passwd`
* Sets up shell and permissions

Alternative:

```bash
sudo useradd -m devuser
sudo passwd devuser
```

### ✏️ Modifying Users

| Command                      | Description                          |
| ---------------------------- | ------------------------------------ |
| `usermod -aG <group> <user>` | Add user to group                    |
| `usermod -d <dir> <user>`    | Change home directory                |
| `usermod -s /bin/zsh <user>` | Change shell                         |
| `usermod -l newname oldname` | Rename user                          |
| `chfn <user>`                | Change user info (name, phone, etc.) |

Example:

```bash
sudo usermod -aG sudo devuser
```

### 🗑️ Deleting Users

| Command             | Description                    |
| ------------------- | ------------------------------ |
| `userdel <user>`    | Delete user only               |
| `userdel -r <user>` | Delete user and home directory |

Example:

```bash
sudo userdel -r testuser
```

---

## 5️⃣ Group Management Commands

### 👪 Creating Groups

```bash
sudo groupadd developers
```

### Adding / Removing Users from Groups

```bash
sudo gpasswd -a john developers   # Add user to group
sudo gpasswd -d john developers   # Remove user from group
```

### Deleting a Group

```bash
sudo groupdel developers
```

### Viewing Group Memberships

```bash
groups john
id john
```

Example output:

```
uid=1001(john) gid=1001(john) groups=1001(john),1002(developers)
```

---

## 6️⃣ Managing Passwords and Security

### 🔑 Setting or Changing Passwords

```bash
passwd <username>
```

To enforce password change next login:

```bash
sudo passwd -e <username>
```

### 🧱 Password Aging

```bash
chage -l <username>   # View password policy
chage -M 90 <username>  # Expire password after 90 days
chage -d 0 <username>   # Force change on next login
```

---

## 7️⃣ Switching Users and sudo Access

### 👑 Root User

Root user (UID 0) can perform **any operation** on the system.

To become root:

```bash
sudo -i
```

### 🔄 Switching Between Users

| Command       | Description                  |
| ------------- | ---------------------------- |
| `su - <user>` | Switch user with environment |
| `su <user>`   | Switch user (no env change)  |
| `exit`        | Return to previous session   |

Example:

```bash
su - devuser
```

---

### ⚙️ sudo Command

`sudu` (SuperUser DO) allows normal users to run privileged commands.

#### To give sudo access:

```bash
sudo usermod -aG sudo devuser
```

#### Verify sudo access:

```bash
sudo whoami
```

Output:

```
root
```

#### Edit sudoers safely:

```bash
sudo visudo
```

Example entry:

```
devuser ALL=(ALL:ALL) ALL
```

---

## 8️⃣ User Profiles and Default Settings

When a new user is created, Linux uses **default templates** stored in `/etc/skel`.

### Default Files:

* `.bashrc` – Shell configuration
* `.profile` – Environment variables
* `.bash_logout` – Commands to run at logout

Copy defaults manually:

```bash
sudo cp /etc/skel/.bashrc /home/newuser/
```

---

## 9️⃣ Advanced User Administration

### 🧠 Locking / Unlocking Users

```bash
sudo usermod -L <user>   # Lock account
sudo usermod -U <user>   # Unlock account
```

### 🔍 Check Login History

```bash
last
```

### 🔍 Check Active Logins

```bash
who
w
```

### ⏱️ Restrict Login Time

Edit `/etc/security/time.conf` to limit when users can log in.

---

## 🔐 10️⃣ Best Practices

✅ Always use **non-root accounts** for regular work
✅ Use **sudo** for administrative tasks
✅ Set strong password policies (`/etc/login.defs`)
✅ Regularly audit inactive or test accounts
✅ Group users by **roles**, not by individuals (e.g., `dev`, `ops`, `qa`)
✅ Disable direct SSH access for root (`/etc/ssh/sshd_config` → `PermitRootLogin no`)
✅ Backup `/etc/passwd`, `/etc/group`, and `/etc/shadow` regularly

---

## 📘 11️⃣ Summary

* Every user in Linux has a **UID**, **GID**, and a home directory.
* Groups simplify **permission management**.
* Key files: `/etc/passwd`, `/etc/shadow`, `/etc/group`
* Use `adduser`, `usermod`, `userdel` for user lifecycle.
* Use `sudo` for safe privilege escalation.
* Proper user and group management ensures **security and collaboration**.

---

> 🧭 **Next Topic:** [06-process-management.md → Managing Processes in Linux](./06-process-management.md)