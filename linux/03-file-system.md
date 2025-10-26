# 📁 Linux File System Structure

Linux organizes everything — programs, files, devices — into a **single hierarchical tree** starting at `/` (the root directory).

---

## 🧱 File System Hierarchy

| Directory | Description |
|------------|-------------|
| `/` | Root of the file system |
| `/home` | Home directories for users |
| `/root` | Home directory for root user |
| `/bin` | Essential binary commands |
| `/sbin` | System binaries (admin) |
| `/etc` | System configuration files |
| `/var` | Logs, caches, spools |
| `/usr` | User-installed programs and libraries |
| `/tmp` | Temporary files |
| `/dev` | Device files |
| `/mnt` | Mount point for temporary devices |
| `/opt` | Optional software |
| `/boot` | Boot loader files |

---

## 🧩 Paths

| Type | Example | Description |
|------|----------|-------------|
| **Absolute Path** | `/home/user/docs/file.txt` | Full path from `/` |
| **Relative Path** | `../docs/file.txt` | Relative to current directory |

---

## 🔗 File Types

| Type | Symbol | Example |
|------|---------|----------|
| Regular File | `-` | `-rw-r--r--` |
| Directory | `d` | `drwxr-xr-x` |
| Symbolic Link | `l` | `lrwxrwxrwx` |
| Block Device | `b` | `/dev/sda1` |
| Character Device | `c` | `/dev/tty0` |

---

## 🧠 Useful Commands

| Command | Description |
|----------|-------------|
| `ls -l` | Long listing (permissions, owner, size) |
| `du -h` | Disk usage per directory |
| `df -h` | Available disk space |
| `file <filename>` | Detect file type |
| `tree` | Display folder structure (install via `apt install tree`) |

---

## ⚡ Mounting and Unmounting

```bash
sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb
