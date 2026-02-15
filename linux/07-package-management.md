# 📦 Linux Package Management & Architecture

## 📋 Table of Contents
- [1️⃣ Package Managers (High-Level vs Low-Level)](#1️⃣-package-managers-high-level-vs-low-level)
- [2️⃣ Internal Mechanics: Dependencies & Repositories](#2️⃣-internal-mechanics-dependencies--repositories)
- [3️⃣ The "Big Two" (APT & RPM)](#3️⃣-the-big-two-apt--rpm)
- [4️⃣ Compilation: Installing from Source](#4️⃣-compilation-installing-from-source)
- [5️⃣ Modern Packaging: Snap & Flatpak](#5️⃣-modern-packaging-snap--flatpak)
- [6️⃣ Language Managers (pip, npm, cargo)](#6️⃣-language-managers-pip-npm-cargo)

---

## 1️⃣ Package Managers (High-Level vs Low-Level)

Linux package management works in two layers:

| Layer | Debian/Ubuntu | RHEL/CentOS | Function |
|:------|:--------------|:------------|:---------|
| **Frontend** | `apt` | `dnf` / `yum` | Resolves dependencies, downloads files, updates repos. |
| **Backend** | `dpkg` | `rpm` | Just installs the file you gave it. No internet, no dependency solving. |

**Scenario**:
- You run `apt install vlc`.
- `apt` calculates dependencies (`libvideo`, `qt-core`).
- `apt` downloads 5 `.deb` files.
- `apt` hands them to `dpkg` to install one by one.

---

## 2️⃣ Internal Mechanics: Dependencies & Repositories

### Dependency Resolution
A package is a node in a graph.
`VLC` -> depends on -> `FFmpeg` -> depends on -> `Codecs`.
If `Codecs` conflicts with `System-Library-v2`, `apt` will refuse to install. This is "Dependency Hell".

### Repositories & Priorities
Your OS looks at `/etc/apt/sources.list`.
What if `Nginx 1.18` is in the **Main** repo, but `Nginx 1.20` is in a **Custom PPA**?
- **APT Pinning** allows you to prioritize specific versions.
- Default priority is usually 500.

---

## 3️⃣ The "Big Two" (APT & RPM)

### 🔹 APT (Debian Family)
Stored in `/var/lib/apt/lists/`.
```bash
# Update local map of internet repos
sudo apt update

# Upgrade all packages
sudo apt upgrade
```

### 🔹 RPM/DNF (Red Hat Family)
Stored in `/var/cache/dnf/`.
```bash
# Search for which package provides a specific file
dnf provides /usr/bin/python3
```

---

## 4️⃣ Compilation: Installing from Source

When `apt` is too old, you compile.

**The Holy Trinity of compilation:**

1.  **`./configure`**:
    - A script that scans your system.
    - "Do you have GCC? Do you have RAM? Do you have the SSL library?"
    - Creates a `Makefile`.
2.  **`make`**:
    - Reads the `Makefile`.
    - Compiles Source Code (`.c`) into Binary (`.o`).
    - CPU intensive.
3.  **`sudo make install`**:
    - Copies the binary to `/usr/local/bin/`.

**Downside**: No `apt update` for this. You must update manually.

---

## 5️⃣ Modern Packaging: Snap & Flatpak

Traditional packages use **Shared Libraries** (Small size, but dependency issues).
Modern packages use **Containerization** (Bundled libraries, Sandbox).

| Feature | **APT/RPM** | **Snap/Flatpak** |
|:--------|:------------|:-----------------|
| **Format** | Archive of files. | Disk Image mounted as loop device. |
| **Deps** | Uses System Libs. | Bundled inside. |
| **Isolation**| None. | Sandboxed (cannot read `/home` by default). |
| **Size** | Small. | Huge. |

---

## 6️⃣ Language Managers (pip, npm, cargo)

**Developer Warning:**
- `apt` manages system python (`/usr/lib/python3`).
- `pip` manages system python libraries.

If you run `sudo pip install requests`, you might break `apt` tools that rely on an older version of `requests`.
**Solution**: ALWAYS use Virtual Environments (`venv`).

---

> 🧭 **Next:** [Networking Deep Dive →](./08-networking.md)