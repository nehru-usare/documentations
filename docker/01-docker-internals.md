# Docker Internals (Namespaces, Cgroups)

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐⭐ (Deep Dive)  
> **Status:** Isolated

---

## 0. Learning Objectives
*   **Beginner**: Why Docker is not a Virtual Machine.
*   **Developer**: Why `ps aux` inside a container generally shows PID 1.
*   **Architect**: Understanding the security boundaries of containers.

---

## 1. Core Concepts (🟢 Beginner Level)

### "Docker is a Lie"
*   There is no such thing as a "Docker Container" in the Linux Kernel.
*   Docker is just a fancy user-space tool that uses two existing Kernel features:
    1.  **Namespaces**: What you can *see*.
    2.  **Cgroups**: What you can *use*.

---

## 2. Architecture Breakdown (The Kernel)

### 1. Namespaces (Isolation)
*   **PID Namespace**: Process ID 1 inside container is Process ID 45678 on Host. Container processes cannot see Host processes.
*   **NET Namespace**: Container has its own `eth0`, IP, and Loopback.
*   **MNT Namespace**: Container has its own `/` root filesystem.
*   **UTS Namespace**: Hostname.
*   **USER Namespace**: Root inside container != Root on Host (if enabled).

### 2. Cgroups (Control Groups)
*   Limits resource usage.
*   `cpu.shares`: "You get 50% of CPU".
*   `memory.limit_in_bytes`: "You get 512MB RAM. If you exceed, OOM Killer shoots you".

### 3. UnionFS (The Filesystem)
*   Images are Read-Only layers.
*   Container adds a thin Read-Write layer on top.
*   **Copy-on-Write (CoW)**: If you modify a file from the image, Docker copies it to the RW layer first.

---

## 3. Internal Mechanics (🔴 Architect Level)

### The Container Plumbing
1.  **Docker CLI**: Sends API request to Daemon.
2.  **Docker Daemon (`dockerd`)**: Manages images/volumes. Calls `containerd`.
3.  **Containerd**: High-level runtime. Manages lifecycle. Calls `runc`.
4.  **Runc**: Low-level runtime. Creates the Namespaces/Cgroups and exits.
5.  **Shim**: Keeps the container running (stdout/stderr) even if Daemon restarts.

---

## 4. Trade-Off Analysis

| Feature | VM | Container |
| :--- | :--- | :--- |
| **Boot Time** | Minutes (BIOS -> Kernel -> OS) | Millis (Start Process) |
| **Size** | GBs (Full OS) | MBs (App + Libs) |
| **Security** | Strong (Hardware Virt) | Weaker (Shared Kernel) |

---

## 5. Scaling Considerations

### The "Noisy Neighbor"
*   Even with Cgroups, containers share the Kernel Data Structures (File Descriptors, ARP Table).
*   One abusive container can crash the Host (e.g., Fork Bomb).
*   **Fix**: `pids-limit`.

---

## 6. Real Production Lessons

### "The PID 1 Problem"
*   Linux Kernel treats PID 1 specially (It must reap zombies).
*   If your App runs as PID 1 (default in Docker) and crashes, it might not handle `SIGTERM` correctly.
*   **Fix**: Use `tini` (`--init`) or entrypoint script to handle signals.

---

## 7. Interview Questions

### Basic
1.  VM vs Container.
2.  What is an Image Layer?
3.  Where does Docker store data? (`/var/lib/docker`).

### Intermediate
1.  Explain Copy-on-Write.
2.  What is a Cgroup?
3.  How to see processes running inside a container from the host? (`ps aux | grep ...`).

### Advanced
1.  If I delete `/bin/bash` in a running container, does it affect the Image? (No, Copy-on-Write).
2.  Explain the container boot sequence (`dockerd` -> `containerd` -> `runc`).
3.  How does `docker exec` work? (Join existing Namespaces).

---

## 8. Summary & Architect Takeaways

1.  **Shared Kernel**: Never assume perfect isolation.
2.  **Layers**: Optimize images by putting changing stuff (Code) at the end, static stuff (OS) at the start.
3.  **Ephemeral**: Containers are cattle, not pets.
