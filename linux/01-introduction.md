# 🐧 Introduction to Linux

## 📋 Table of Contents
- [🔹 What is Linux?](#-what-is-linux)
- [🧱 Linux Architecture](#-linux-architecture)
- [🧩 Linux Distributions](#-linux-distributions)
- [⚙️ Kernel vs OS](#️-kernel-vs-os)
- [💡 Why Developers Use Linux](#-why-developers-use-linux)
- [🧠 Practice Setup Options](#-practice-setup-options)
- [🚀 Minimal Working Example (Hands-on)](#-minimal-working-example-hands-on)
- [🏗️ Use Cases: When to use what?](#️-use-cases-when-to-use-what)
- [🔧 Internal Mechanics: The Kernel](#-internal-mechanics-the-kernel)
- [🏁 The Boot Process (How Linux Starts)](#-the-boot-process-how-linux-starts)
- [🖥️ Terminal vs Shell vs Console](#️-terminal-vs-shell-vs-console)

---

## 🔹 What is Linux?
Linux is an **open-source, Unix-like operating system** that powers servers, desktops, mobile devices, and embedded systems across the world.  
It’s known for its **stability**, **security**, and **performance** — making it the backbone of the **internet and cloud computing**.

> **Note:** Linux is actually just the *kernel*. The OS you use (like Ubuntu) is a collection of utilities, libraries, and the kernel packed together.

---

## 🧱 Linux Architecture

Think of Linux as a layered cake.

| Layer | Description | Example Components |
|:-------|:-------------|:-------------------|
| **User Space** | Where your applications run. | Web Browser, Python Script, Text Editor |
| **Shell** | The interface interpreting your commands. | Bash, Zsh, Fish |
| **System Libraries** | Functions used by applications to talk to the kernel. | glibc, libsystemd |
| **Kernel Space** | The core managing hardware resources. | Process Scheduler, Memory Manager, Drivers |
| **Hardware** | Physical machinery. | CPU, RAM, Disk, Network Card |

---

## 🧩 Linux Distributions

Linux comes in many “flavors” called **distributions (distros)** — each with its own package manager and philosophy.

| Distribution | Package Manager | Primary Use Case | When to Choose? |
|:-------------|:----------------|:-----------------|:----------------|
| **Ubuntu** | `apt` (Debian) | Desktop & General Usage | **Beginners & Developers.** Excellent community support, easy to install proprietary drivers. |
| **Debian** | `apt` (Debian) | Servers & Stability | **SysAdmins.** When you need rock-solid stability and don't care about having the "newest" versions of software. |
| **CentOS / RHEL** | `yum` / `dnf` (RPM) | Enterprise Servers | **Corporate Environments.** Standard in many big companies. (Note: CentOS Stream is the new upstream). |
| **Fedora** | `dnf` (RPM) | Workstation / Cutting Edge | **Linux Enthusiasts.** If you want the latest kernel and features before they hit RHEL. |
| **Alpine** | `apk` | Containers (Docker) | **Cloud Native / Docker.** Extremely lightweight (5MB base image). |

---

## ⚙️ Kernel vs OS

- **Kernel** → The "Engine" of the car. It handles CPU scheduling, memory management (RAM), and talks to hardware drivers.
- **Operating System** → The "Car" itself. It includes the engine (kernel), but also the steering wheel (shell), the dashboard (GUI), and the wheels (utilities).

---

## 💡 Why Developers Use Linux

1.  **The Server Standard**: 90%+ of the world's cloud servers run Linux. If you write backend code, it will likely run on Linux.
2.  **Tooling Support**: Tools like Docker, Kubernetes, and Redis are "Linux-native". They run best (or only) on Linux.
3.  **Scripting Power**: Bash/Shell scripting allows you to automate complex tasks that are difficult in a GUI.

---

## 🧠 Practice Setup Options

| Option | Pros | Cons |
|:-------|:-----|:-----|
| **WSL 2 (recommended)** | Runs inside Windows. Fast, easy file sharing. | No native GUI (though WSLg exists). |
| **Virtual Machine** | Complete isolation. Snapshots. | Heavy on RAM/CPU. Slower integration. |
| **Dual Boot** | 100% Hardware performance. | Annoying to reboot to switch OS. Risk of breaking Windows bootloader. |
| **Docker Container** | Instant startup. Ephemeral. | Not a full persistent OS. Data lost on exit (unless mounted). |

---

## 🚀 Minimal Working Example (Hands-on)

Open your terminal and try these commands to identify your system.

### 1. Check Kernel Version
```bash
uname -r
# Output Example: 5.15.0-1031-aws
# Meaning: You are running Linux Kernel version 5.15
```

### 2. Check Distribution Info
```bash
cat /etc/os-release
# Output Example:
# PRETTY_NAME="Ubuntu 22.04.2 LTS"
# NAME="Ubuntu"
# VERSION_ID="22.04"
```

### 3. Check System Architecture
```bash
lscpu | head -n 5
# Output Example:
# Architecture:            x86_64
# CPU(s):                  4
```

---

## 🏗️ Use Cases: When to use what?

### Use Case 1: The Personal Developer Laptop
**Scenario:** You are a frontend developer building React apps but need a backend API.
**Recommendation:** Use **WSL 2 (Ubuntu)** on Windows or **macOS** (Unix-based).
**Why?** You get the full Linux terminal toolchain (Node, NPM, Git) without giving up Photoshop or Office.

### Use Case 2: The Enterprise Database Server
**Scenario:** You are deploying a massive PostgreSQL database for a bank.
**Recommendation:** Use **RHEL (Red Hat Enterprise Linux)** or **Debian Stable**.
**Why?** Stability is king. You don't want "bleeding edge" updates breaking your database driver. You want 10-year support cycles.

### Use Case 3: The Microservice Container
**Scenario:** You are building a Java Spring Boot app effectively to deploy on Kubernetes.
**Recommendation:** Use **Alpine Linux** or **Distroless** base images.
**Why?** Security and Size. A 5MB OS image is faster to pull and has fewer security vulnerabilities than a 700MB full OS.

---

## 🔧 Internal Mechanics: The Kernel

The Linux Kernel is **Monolithic**, meaning the entire OS (drivers, file system management, network stack) runs in the same high-privilege memory space (**Kernel Space**).

### Key Responsibilities
1.  **Process Scheduler**: Decides which app gets the CPU right now (Completely Fair Scheduler - CFS).
2.  **Memory Management (OOM)**: Allocates RAM. If RAM runs out, the **OOM Killer** (Out of Memory Killer) sacrifices a process to save the system.
3.  **System Calls (Syscalls)**: The API that apps use to talk to the kernel (e.g., `open()`, `read()`, `fork()`).

### Kernel Versioning Semantics
Example: `5.15.0-76`
- **5**: Major Version (Huge changes).
- **15**: Minor Version (New features).
- **0**: Patch Level (Bug/Security fixes).
- **-76**: Distro-specific build number.

---

## 🏁 The Boot Process (How Linux Starts)

Understanding this helps you debug "My server won't start" issues.

1.  **BIOS / UEFI**: The motherboard initializes hardware and looks for a bootable device (Part of the hardware, not Linux).
2.  **Boot Loader (GRUB)**: Grand Unified Bootloader. It loads the Kernel into memory. You see this as the menu where you pick "Ubuntu" or "Windows".
3.  **Kernel**: Mounts the Root File System (`/`) as read-only.
4.  **Init Process (Systemd)**: The Kernel starts the **first** process, PID 1.
    - Historically this was `SysVinit` (scripts).
    - Modern Linux uses `Systemd` (parallel, improved dependency management).
5.  **User Space**: Systemd starts background services (Network, Bluetooth, GUI Login).

---

## 🖥️ Terminal vs Shell vs Console

Developers often confuse these terms.

| Term | Definition | Real-world Analogy |
|:-----|:-----------|:-------------------|
| **Terminal** | The program window that accepts input and shows output (e.g., Gnome Terminal, iTerm2, Putty). | The **Monitor/Screen** and Keyboard. |
| **Shell** | The program interpreting your commands (e.g., Bash, Zsh). It runs *inside* the terminal. | The **Brain** processing the words. |
| **Console** | The physical hardware terminal (mostly historical) or the direct text-mode access to the server. | The physical machine itself. |

> **Pro Tip:** When you SSH into a server, your **SSH Client** is the Terminal, and the remote server runs the **Shell**.

---

> 🧭 **Next:** [Basic Linux Commands →](./02-commands.md)
