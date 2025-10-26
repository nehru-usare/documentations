# 📦 Linux Package Management

Linux systems use **package managers** to install, update, and remove software easily.  
A package is a bundle of software, configuration files, and metadata that can be installed as a single unit.

This document covers **how package management works** across major Linux distributions.

---

## 🧭 Table of Contents

1. [What is a Package?](#1-what-is-a-package)
2. [Package Formats](#2-package-formats)
3. [APT (Debian/Ubuntu)](#3-apt-debianubuntu)
4. [DPKG (Low-Level Debian Tool)](#4-dpkg-low-level-debian-tool)
5. [YUM and DNF (RedHat/CentOS/Fedora)](#5-yum-and-dnf-redhatcentosfedora)
6. [Repository Management](#6-repository-management)
7. [Package Search and Info Commands](#7-package-search-and-info-commands)
8. [Verifying and Cleaning Packages](#8-verifying-and-cleaning-packages)
9. [Installing from Source or External Files](#9-installing-from-source-or-external-files)
10. [Best Practices](#10-best-practices)
11. [Summary](#11-summary)

---

## 1️⃣ What is a Package?

A **package** in Linux is a compressed archive containing:
- Compiled binaries or libraries
- Configuration files
- Metadata (version, dependencies, description)

Package managers automate:
- Dependency resolution  
- Updates and upgrades  
- Uninstalling cleanly  

---

## 2️⃣ Package Formats

| Distribution | Package Type | Package Manager |
|---------------|---------------|-----------------|
| Debian, Ubuntu | `.deb` | `apt`, `dpkg` |
| Red Hat, CentOS, Fedora | `.rpm` | `yum`, `dnf`, `rpm` |
| Arch Linux | `.pkg.tar.zst` | `pacman` |
| SUSE Linux | `.rpm` | `zypper` |

---

## 3️⃣ APT (Debian/Ubuntu)

APT (Advanced Package Tool) is the most commonly used **high-level package manager** on Ubuntu and Debian-based systems.

### 🔹 Updating Repository Cache

```bash
sudo apt update
````

This refreshes the list of available packages and versions.

---

### 🔹 Installing Packages

```bash
sudo apt install nginx
sudo apt install openjdk-17-jdk
```

APT automatically downloads dependencies.

---

### 🔹 Removing Packages

```bash
sudo apt remove nginx
```

If you want to remove configuration files too:

```bash
sudo apt purge nginx
```

---

### 🔹 Upgrading Packages

| Command                 | Description                                 |
| ----------------------- | ------------------------------------------- |
| `sudo apt upgrade`      | Upgrade installed packages                  |
| `sudo apt full-upgrade` | Upgrade + handle dependencies intelligently |
| `sudo apt dist-upgrade` | Legacy equivalent of full-upgrade           |
| `sudo apt autoremove`   | Remove unused dependencies                  |

---

### 🔹 Listing and Searching Packages

```bash
apt list --installed
apt search nginx
```

---

### 🔹 Display Package Information

```bash
apt show nginx
```

Output example:

```
Package: nginx
Version: 1.18.0-6ubuntu14.4
Description: high performance web server
Depends: libc6, libpcre3
```

---

## 4️⃣ DPKG (Low-Level Debian Tool)

`dpkg` is a **backend package manager** for Debian systems.
APT uses it under the hood to install `.deb` packages.

### 🔹 Install a `.deb` file manually

```bash
sudo dpkg -i package.deb
```

If there are missing dependencies:

```bash
sudo apt --fix-broken install
```

---

### 🔹 Query Installed Packages

```bash
dpkg -l | grep nginx
```

### 🔹 Remove a Package

```bash
sudo dpkg -r nginx
```

### 🔹 List Files in a Package

```bash
dpkg -L nginx
```

### 🔹 Check Which Package Owns a File

```bash
dpkg -S /usr/bin/java
```

---

## 5️⃣ YUM and DNF (RedHat/CentOS/Fedora)

RPM-based distributions use **YUM** (Yellowdog Updater, Modified) or **DNF** (its modern replacement).

### 🔹 Update Repository

```bash
sudo yum update
# OR
sudo dnf update
```

### 🔹 Install Package

```bash
sudo yum install httpd
# OR
sudo dnf install httpd
```

### 🔹 Remove Package

```bash
sudo yum remove httpd
# OR
sudo dnf remove httpd
```

### 🔹 Search and Info

```bash
yum search nginx
yum info nginx
```

### 🔹 List Installed Packages

```bash
yum list installed
```

### 🔹 Clean Cache

```bash
sudo yum clean all
sudo dnf clean packages
```

---

## 6️⃣ Repository Management

Repositories (repos) are URLs or directories that contain packages.

### 🔹 Ubuntu/Debian

| File                       | Description          |
| -------------------------- | -------------------- |
| `/etc/apt/sources.list`    | Main repository list |
| `/etc/apt/sources.list.d/` | Custom repositories  |

Add new repository:

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
```

---

### 🔹 RHEL/CentOS

| Directory           | Description            |
| ------------------- | ---------------------- |
| `/etc/yum.repos.d/` | Stores `.repo` files   |
| `/etc/yum.conf`     | Main YUM configuration |

Example of repo file:

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://nginx.org/keys/nginx_signing.key
```

---

## 7️⃣ Package Search and Info Commands

| Command            | Purpose                        |
| ------------------ | ------------------------------ |
| `apt search <pkg>` | Search package (Debian)        |
| `yum search <pkg>` | Search package (RHEL)          |
| `apt show <pkg>`   | View package details           |
| `yum info <pkg>`   | View package details           |
| `dpkg -S <file>`   | Find which package owns a file |

---

## 8️⃣ Verifying and Cleaning Packages

### 🔹 Check Installed Packages

```bash
apt list --installed
yum list installed
```

### 🔹 Verify Package Integrity

```bash
debsums -s <package>
rpm -V <package>
```

### 🔹 Clean Cache and Unused Packages

```bash
sudo apt autoremove
sudo apt clean
sudo yum autoremove
```

---

## 9️⃣ Installing from Source or External Files

Sometimes software isn’t available via package managers.
You can install manually from **source** or **external repositories**.

### 🔹 Installing from .deb or .rpm

```bash
sudo dpkg -i example.deb
sudo rpm -ivh example.rpm
```

### 🔹 Installing from Source

```bash
tar -xvzf app.tar.gz
cd app
./configure
make
sudo make install
```

---

## 🔟 Best Practices

✅ Always update package index before installing (`sudo apt update`)
✅ Prefer official repositories — avoid random `.deb` or `.rpm` files
✅ Use `apt upgrade` regularly to stay secure
✅ Use `apt purge` or `yum remove` to clean config files
✅ Use `autoremove` to delete unused dependencies
✅ Check package info before installing unfamiliar software
✅ Keep backups of `/etc/apt/sources.list` and `/etc/yum.repos.d/`

---

## 🧾 Summary

* Linux packages simplify **installation and management of software**.
* **APT (Debian/Ubuntu)** and **YUM/DNF (RHEL/Fedora)** are the most common package managers.
* **dpkg** and **rpm** handle low-level package operations.
* Repositories store and distribute verified software packages.
* Regular package maintenance keeps your system **secure, stable, and efficient**.

---

> 🧭 **Next Topic:** [08-networking.md → Basic Networking Commands in Linux](./08-networking.md)