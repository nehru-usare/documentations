# 📁 Linux File System Structure

## 📋 Table of Contents
- [1️⃣ The File System Hierarchy (FHS)](#1️⃣-the-file-system-hierarchy-fhs)
- [2️⃣ Internal Mechanics: Inodes & Metadata](#2️⃣-internal-mechanics-inodes--metadata)
- [3️⃣ Virtual File System (VFS)](#3️⃣-virtual-file-system-vfs)
- [4️⃣ Device Files (/dev)](#4️⃣-device-files-dev)
- [5️⃣ Mounting & Persistence (/etc/fstab)](#5️⃣-mounting--persistence-etcfstab)
- [6️⃣ Advanced: Logical Volume Manager (LVM)](#6️⃣-advanced-logical-volume-manager-lvm)

---

## 1️⃣ The File System Hierarchy (FHS)

Linux organizes everything into a **Single Hierarchical Tree** starting at `/`.

| Directory | Description | "Windows Equivalent" |
|:----------|:------------|:---------------------|
| `/` | **Root**. The beginning of everything. | `This PC` |
| `/bin` & `/sbin` | **Binaries**. Essential programs (`ls`, `reboot`). | `C:\Windows\System32` |
| `/etc` | **Etcetera**. Configuration files. | Registry / `AppData` |
| `/home` | **Home**. Users' personal files. | `C:\Users` |
| `/lib` | **Libraries**. Shared code for binaries. | `C:\Windows\System32\*.dll` |
| `/dev` | **Devices**. Hardware represented as files. | Device Manager |
| `/proc` & `/sys` | **Virtual**. Kernel & Hardware info in RAM. | Task Manager details |
| `/var` | **Variable**. Logs, Databases, Mail. | `C:\ProgramData` |

---

## 2️⃣ Internal Mechanics: Inodes & Metadata

When you save a file, Linux stores it in two parts:
1.  **The Data Block**: The actual content ("Hello World").
2.  **The Inode (Index Node)**: The metadata (Owner, Permissions, Size, Location on disk).

**The Filename** is just a label pointing to an Inode number.

### 🧠 The "Inode Full" Problem
You can run out of disk space in two ways:
1.  **Blocks Full**: Disk is actually full of data (`df -h`).
2.  **Inodes Full**: Too many small files used up all ID numbers (`df -i`).

---

## 3️⃣ Virtual File System (VFS)

Linux can read many file systems (ext4, xfs, ntfs, vfat) transparently.
The **VFS** is a kernel layer that translates standard commands (`ls`, `cp`) into specific driver instructions.

`User` -> `cp file.txt` -> `VFS` -> `ext4 driver` -> `Hard Drive`

---

## 4️⃣ Device Files (/dev)

In Linux, "Everything is a file", even hardware.

| Type | Symbol | Description | Example |
|:-----|:-------|:------------|:--------|
| **Block Device** | `b` | Buffered data access (Storage). | `/dev/sda` (Hard Drive) |
| **Character Device**| `c` | Unbuffered, direct stream (Input). | `/dev/console`, Mouse |

**Cool Trick:**
Send text directly to a terminal device:
```bash
echo "Hello!" > /dev/pts/0
```

---

## 5️⃣ Mounting & Persistence (/etc/fstab)

**Mounting** attaches a storage device to a folder.

### Manual Mount
```bash
sudo mount /dev/sdb1 /mnt/usb
```

### Persistent Mount (/etc/fstab)
To survive a reboot, add it to `/etc/fstab`.

```ini
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/sdb1       /data           ext4    defaults        0       0
```

> **⚠️ Critical Warning:** A syntax error in `/etc/fstab` can cause your system to **fail to boot**. Always test with `sudo mount -a` before rebooting.

---

## 6️⃣ Advanced: Logical Volume Manager (LVM)

Traditional Partitions are fixed size. **LVM** allows flexible resizing.

**Structure:**
1.  **Physical Volume (PV)**: The actual disk (`/dev/sda`).
2.  **Volume Group (VG)**: A pool of combined disks.
3.  **Logical Volume (LV)**: A "virtual partition" you can resize on the fly.

**Scenario:** Database filling up?
1.  Plug in new drive.
2.  Add it to the **Volume Group**.
3.  Extend the **Logical Volume**.
4.  Zero downtime.

---

> 🧭 **Next:** [Permissions & Ownership Deep Dive →](./04-permissions.md)
