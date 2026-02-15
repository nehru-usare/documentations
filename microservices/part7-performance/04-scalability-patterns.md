# 04. Scalability Patterns (AKF Scale Cube, Statelessness)

> **Part 7: Performance & Scalability**  
> **Difficulty:** ⭐⭐⭐⭐ (Architect)  
> **Status:** The Blueprint

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand Vertical vs Horizontal Scaling. |
| **Developer** | Write Stateless Microservices (No HttpSession). |
| **Architect** | Apply the AKF Scale Cube to reach 100M users. |

---

## 1. Why This Topic Exists

### The Wall
You added more CPU (Vertical), but the server is still slow.
You added more Servers (Horizontal), but the DB is locked.
**Scalability Patterns** tell you *how* to split the workload.

---

## 3. Core Concepts (🟢 Beginner Level)

### The AKF Scale Cube (The Holy Trinity)

1.  **X-Axis (Horizontal Duplication)**
    *   Clone the service. Run 10 copies behind a Load Balancer.
    *   *Requirement*: Statelessness.
    *   *Limit*: DB becomes bottleneck.

2.  **Y-Axis (Functional Decomposition)**
    *   Split Monolith into Microservices.
    *   "Checkout" services scales independently of "Login" service.
    *   *Limit*: Complexity of distributed systems.

3.  **Z-Axis (Data Partitioning / Sharding)**
    *   Split by Customer ID.
    *   Customers 1-1M -> Pod A.
    *   Customers 1M-2M -> Pod B.
    *   *Idea*: Each pod has a slice of data (Uber uses this for Cities).

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Stateless Architecture
**Rule**: Never store "User Session" in Service RAM.
*   *Bad*: `HttpSession.put("cart", items)`.
    *   If Request 2 hits a different server, the Cart is empty.
*   *Fix*: Store Cart in **Redis** or Client (JWT/Cookie).

### Sticky Sessions (Session Affinity)
Load Balancer ensures Client A always hits Server 1.
*   *Why Bad?*: If Server 1 dies, Client A loses everything. Uneven load distribution.
*   *Fix*: **Statelessness**. Any request can go to any server.

---

## 5. Internal Mechanics (🔴 Architect Level)

### Caching at the Edge (CDN)
Don't serve `logo.png` from your Java App.
Use Cloudflare / AWS CloudFront.
*   Offloads 90% of traffic.
*   Reduces latency (Server is in Users' city).

---

## 14. Summary & Architect Takeaways

*   **Scale Out (X)**: Easy, just add money.
*   **Scale Functional (Y)**: Hard, requires refactoring.
*   **Scale Data (Z)**: Very Hard, requires smart routing.
