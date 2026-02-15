# 🤖 Linux Automation: Beyond Scripts

## 📋 Table of Contents
- [1️⃣ The Evolution of Automation](#1️⃣-the-evolution-of-automation)
- [2️⃣ Cron & Systemd Timers (Recap)](#2️⃣-cron--systemd-timers-recap)
- [3️⃣ The Golden Rule: Idempotency](#3️⃣-the-golden-rule-idempotency)
- [4️⃣ Cloud-Init (Bootstrapping Servers)](#4️⃣-cloud-init-bootstrapping-servers)
- [5️⃣ Deep Dive: Ansible (Agentless Automation)](#5️⃣-deep-dive-ansible-agentless-automation)
- [6️⃣ Ansible Playbook Example](#6️⃣-ansible-playbook-example)

---

## 1️⃣ The Evolution of Automation

| Level | Tool | Description | Scale |
|:------|:-----|:------------|:------|
| **Level 1** | Bash Scripts | "Do `apt install nginx`". Fails if run twice. | 1 Server |
| **Level 2** | Cron / Systemd | "Run script every day". Good for backups. | 1 Server |
| **Level 3** | **Cloud-Init** | "Configure me when I first boot". | Cloud VMs |
| **Level 4** | **Ansible** | "Make these 100 servers look like this YAML". | 1000+ Servers |

---

## 2️⃣ Cron & Systemd Timers (Recap)

We covered these in **Chapter 10**.
- **Cron**: Simple, time-based.
- **Systemd Timers**: Robust, logs to journal, handles missed events.

---

## 3️⃣ The Golden Rule: Idempotency

**Definition**: An operation is *idempotent* if the result is the same whether you run it once or 1000 times.

**Bad (Not Idempotent):**
```bash
# append_line.sh
echo "server=8.8.8.8" >> /etc/resolv.conf
```
*Run twice?* You get duplicate lines.

**Good (Idempotent):**
```bash
# Ansible approach
"Ensure line 'server=8.8.8.8' exists in /etc/resolv.conf"
```
*Run twice?* Nothing happens the second time.

---

## 4️⃣ Cloud-Init (Bootstrapping Servers)

When you launch an EC2 instance, how do you set the password? **Cloud-Init**.
It runs **once** at the very first boot.

**Example `user-data` script:**
```yaml
#cloud-config
users:
  - name: devops
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa AAAAB3Nza...

packages:
  - nginx
  - git

runcmd:
  - systemctl start nginx
```

---

## 5️⃣ Deep Dive: Ansible (Agentless Automation)

**Why Ansible?**
1.  **Agentless**: No software needed on the target server. Just SSH.
2.  **YAML**: Easy to read.
3.  **Idempotent**: Safe to run repeatedly.

### Core Concepts
1.  **Inventory**: A list of servers.
    ```ini
    [webservers]
    192.168.1.50
    192.168.1.51
    ```
2.  **Ad-Hoc Commands**:
    ```bash
    # "Ping all webservers"
    ansible webservers -i hosts.ini -m ping
    ```

---

## 6️⃣ Ansible Playbook Example

This single file installs Nginx, configures it, and starts it.

`site.yml`:
```yaml
---
- name: Configure Web Server
  hosts: webservers
  become: true  # Run as sudo

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Create index.html
      copy:
        content: "<h1>Automated by Ansible!</h1>"
        dest: /var/www/html/index.html
        mode: '0644'

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

**Run it:**
```bash
ansible-playbook -i hosts.ini site.yml
```

---

> 🧭 **Next:** [System Administration Tips (Rescue & Recovery) →](./13-admin-tips.md)
