# 05. Platform Security (Network Policies, Pod Security)

> **Part 6: Security**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (SRE)  
> **Status:** Advanced

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand that internal networks are dangerous. |
| **Developer** | Configure `runAsNonRoot: true`. |
| **Architect** | Enforce Default Deny network policies. |

---

## 1. Why This Topic Exists

### The Flat Network
By default, in K8s, **any pod can talk to any pod**.
*   Hacked Frontend -> calls Database directly.
*   Hacked Frontend -> scans internal network.

---

## 3. Core Concepts (🟢 Beginner Level)

### Network Policy
A YAML firewall rule.
*   "Only `Frontend` pods can call `Backend` pods on port 8080".
*   "Database rejects everything except `Backend`".

### Rootless Containers
Running as `root` (UID 0) inside container = `root` on Host (conceptually).
If attacker breaks out (Container Escape), they own the Node.
*   *Fix*: `securityContext: runAsUser: 1000`.

---

## 14. Summary & Architect Takeaways

*   **Least Privilege**: Applies to Network too.
*   **Defense in Depth**: Even if they have a Password, the Network Policy should block the connection.
