# 01. Security Fundamentals (CIA Triad, Zero Trust)

> **Part 6: Security**  
> **Difficulty:** ⭐⭐⭐ (Architect)  
> **Status:** Mandatory

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand the CIA Triad. |
| **Developer** | Apply Principle of Least Privilege. |
| **Architect** | Design a Zero Trust Architecture. |

---

## 1. Why This Topic Exists

### The Old Castle Model
Old security was "Build a big firewall (Moat) and trust everyone inside (Castle)".
**Problem**: Once a hacker gets inside (Phishing), they can access everything.

### The New Model: Zero Trust
"Never Trust, Always Verify."
Even if the request comes from `10.0.0.5` (Internal), we verify its Identity (JWT).

---

## 3. Core Concepts (🟢 Beginner Level)

### The CIA Triad
1.  **Confidentiality**: Only authorized people can see data (Encryption).
2.  **Integrity**: Data hasn't been tampered with (Signing / Hashing).
3.  **Availability**: Data is accessible when needed (DDoS protection).

### Principle of Least Privilege
A Service should only have permissions to do its specific job.
*   *Bad*: `OrderService` has `root` access to DB.
*   *Good*: `OrderService` has `SELECT, INSERT` on `orders_schema`.

---

## 5. Internal Mechanics (🔴 Architect Level)

### Zero Trust Architecture (NIST 800-207)
1.  **Identity is the Perimeter**: We don't care about IPs. We care about **Who** you are (Service Account / User).
2.  **Micro-Segmentation**: Firewall rules preventing `Frontend` from talking to `Database` directly.
3.  **Encryption Everywhere**: mTLS between all services.

---

## 14. Summary & Architect Takeaways

*   **Security is everyone's job**: Not just the "Security Team".
*   **Layered Defense**: If the Firewall fails, the Gateway should catch it. If Gateway fails, the Service should catch it.
