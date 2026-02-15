# Ansible (Configuration Management)

> **Part 3: Infrastructure as Code**  
> **Difficulty:** ⭐⭐ (Scripting)  
> **Status:** Tasks [OK]

---

## 0. Learning Objectives
*   **Beginner**: Running an Ad-Hoc command against 100 servers.
*   **Developer**: Writing a Playbook to install Nginx.
*   **Architect**: Using Ansible for "Day 2 Operations" (Patching, Backup).

---

## 1. Context
**Ansible**: A radical shift from Chef/Puppet.
*   **Agentless**: No software to install on target servers.
*   **Push Model**: You run it from your laptop (or Jenkins), it pushes changes via SSH.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Inventory
*   A text file listing your servers.
    ```ini
    [web]
    192.168.1.10
    192.168.1.11

    [db]
    db.prod.local
    ```

### 2. Playbook
*   YAML file describing "Plays" (Actions).
    ```yaml
    - hosts: web
      tasks:
        - name: Install Nginx
          apt:
            name: nginx
            state: present
    ```

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Agentless via SSH
*   Ansible uses your local SSH keys.
*   It connects, uploads a small Python script, executes it, deletes it, and disconnects.
*   **Requirement**: Python must be installed on target (Found on 99% of Linux cloud instances).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Modules
*   Ansible doesn't just run shell commands. It uses **Modules**.
*   `shell: useradd bob` (Not idempotent. Fails if bob exists).
*   `user: name=bob state=present` (Idempotent. Checks if bob exists first).

### 2. Roles
*   Grouping Tasks, Files, Templates, and Variables into a reusable folder structure.
*   `roles/webserver/tasks/main.yml`.
*   Standard way to share Ansible code (Ansible Galaxy).

---

## 5. Trade-Off Analysis

| Feature | Ansible | Terraform |
| :--- | :--- | :--- |
| **Focus** | Config (Inside OS) | Infrastructure (Cloud API) |
| **Style** | Mutable | Immutable |
| **Agent** | No | No |
| **State** | Stateless (Real-time check) | Stateful (tfstate) |

---

## 6. Scaling Considerations

### Execution Speed
*   SSH is slow. Connecting to 1000 servers takes time.
*   **Forks**: Increase parallelism (default 5). `ansible-playbook -f 50`.
*   **Pull Mode**: For 10,000 nodes, use `ansible-pull` (Nodes fetch config from Git).

---

## 7. Failure Scenarios & Recovery

### 1. Half-Applied State
*   Playbook crashes halfway. Server 1-50 updated. 51-100 pending.
*   **Fix**: Rerun the playbook. Idempotency ensures 1-50 are skipped (No changes) and 51-100 are applied.

---

## 8. Security Considerations

### 1. Ansible Vault
*   Encrypts secrets inside the YAML file.
*   `ansible-vault encrypt secrets.yml`.
*   Decrypted at runtime using a password/key.

---

## 9. Performance Considerations

*   **Fact Gathering**: Ansible scans OS info (IP, RAM) at start. Slows down run.
*   Disable `gather_facts: false` if not needed.

---

## 10. Real Production Lessons

### "The Glue Code"
*   Even with Terraform & Kubernetes, Ansible is useful.
*   *Use Case*: "Update the SSL certificate on the Load Balancer".
*   *Use Case*: "Trigger a rolling restart of the legacy cluster".

---

## 11. Interview Questions

### Basic
1.  How does Ansible connect to servers? (SSH).
2.  What is a Playbook?
3.  Difference between `shell` and `command` modules.

### Intermediate
1.  Explain Idempotency in Ansible.
2.  What is Ansible Galaxy?
3.  How to encrypt passwords? (Vault).

### Advanced
1.  Architect an Ansible Tower / AWX setup for RBAC.
2.  Compare Ansible (Push) vs Puppet (Pull).
3.  Write a custom Ansible Plugin in Python.

---

## 12. Summary & Architect Takeaways

1.  **Simple is Better**: Ansible's learning curve is near zero.
2.  **Completes the Toolkit**: Terraform creates the server. Ansible configures the internals. Docker runs the app.
3.  **Automation**: If you type it in a terminal twice, put it in a Playbook.
