# Docker Security (Rootless, Capabilities)

> **Part 2: Best Practices**  
> **Difficulty:** ⭐⭐⭐⭐ (Security)  
> **Status:** Hardened

---

## 0. Learning Objectives
*   **Beginner**: Why running as `root` inside a container is dangerous.
*   **Developer**: Using `Trivy` to find CVEs.
*   **Architect**: Designing a Rootless production environment.

---

## 1. Core Concepts (🟢 Beginner Level)

### The Attack Vector
*   If I am `root` inside a container -> I break out (Kernel Exploit) -> I am `root` on the Host.
*   **Game Over**.
*   **Goal**: Ensure container root != Host root.

---

## 2. Architecture Breakdown (Defense Depth)

### 1. Rootless Mode
*   Runs the Docker Daemon itself as a non-root user.
*   Uses **User Namespaces** to map `Container Root (0)` to `Host User (1001)`.
*   Even if attacker breaks out, they are just a normal user on Host.

### 2. Capabilities (`--cap-add` / `--cap-drop`)
*   Root has "All Powers" (Kill processes, Bind port < 1024, Mount disks).
*   **Principle of Least Privilege**:
    *   `docker run --cap-drop ALL --cap-add NET_BIND_SERVICE ...`
    *   Give the container *only* what it needs.

### 3. Seccomp (Secure Computing)
*   Kernel has 300+ Syscalls (`open`, `read`, `write`, `reboot`...).
*   Docker blocks ~44 dangerous syscalls by default.
*   You can whitelist strictly needed syscalls (e.g., Nginx needs `epoll`).

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. The Supply Chain Attack
*   Attacker publishes `ubuntu-optimized` image. You pull it.
*   It contains a crypto miner.
*   **Fix**:
    1.  **Trust**: Use Docker Official Images or Verified Publisher.
    2.  **Verify**: Docker Content Trust (Notary) signs images.

---

## 4. Scaling Considerations

### Image Scanning
*   Integrate **Trivy** or **Clair** in CI/CD.
*   `trivy image my-app:latest`.
*   Block deployment if HIGH/CRITICAL CVEs are found.

---

## 5. Real Production Lessons

### "The Dirty COW"
*   Kernel vulnerability (Copy On Write).
*   Allowed escalation from Container to Host.
*   **Lesson**: Container Security depends on **Host Kernel Security**. Patch your host!

---

## 6. Interview Questions

### Basic
1.  Why not run as root?
2.  What is a CVE?
3.  How to scan an image?

### Intermediate
1.  Explain Capabilities. Which one allows `ping`? (`NET_RAW`).
2.  What is "Privileged" mode? (`--privileged`). Why is it bad? (Gives access to all devices).
3.  Difference between `USER` in Dockerfile and User Namespaces.

### Advanced
1.  How does Rootless Docker work internally? (uses `newuidmap`/`newgidmap`).
2.  Write a Seccomp profile to block `chmod`.
3.  Explain the risks of mounting `/var/run/docker.sock`. (Total Host Takeover).

---

## 7. Summary & Architect Takeaways

1.  **Drop Capabilities**: Default security is good, but `DROP ALL` is better.
2.  **Read Only Root**: `docker run --read-only`. Attackers can't download malware.
3.  **Scan Daily**: New CVEs are discovered every day.
