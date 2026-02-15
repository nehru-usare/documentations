# 05. Distributed Caching Strategy

> **Part 4: Data & Consistency**  
> **Difficulty:** ⭐⭐⭐⭐ (Architect)  
> **Status:** Performance Critical

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand that Cache = RAM = Fast. |
| **Developer** | Use `@Cacheable` in Spring Boot with Redis. |
| **Architect** | Solve Cache Invalidation and Cache Stampede problems. |

---

## 1. Why This Topic Exists

### The Latency Gap
*   **Disk (DB)**: 5 - 100 ms.
*   **RAM (Redis)**: 0.1 - 1 ms.
*   **Goal**: Serve 90% of requests from RAM to protect the Database.

---

## 3. Core Concepts (🟢 Beginner Level)

### Local vs Distributed Cache
1.  **Local (Caffeine/HashMap)**: Inside the JVM. Super fast. But inconsistent across instances (Node A has old data, Node B has new).
2.  **Distributed (Redis)**: External server. Shared by all instances. Consistent but slower (Network hop).

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Strategies

**1. Cache-Aside (Lazy Loading)** - *Most Common*
*   *Read*: Check Redis. If Miss -> Read DB -> Write Redis -> Return.
*   *Write*: Write DB -> Delete Redis Key (Invalidate).
*   *Why Delete?*: Updating cache is race-condition prone. Deleting forces a clean reload next time.

**2. Write-Through**
*   App writes to Cache. Cache writes to DB. (Slow writes, fast reads).

**3. Write-Behind (Write-Back)**
*   App writes to Cache. Cache queues write to DB async. (Super fast, but risk of data loss if Cache dies).

---

## 5. Internal Mechanics (🔴 Architect Level)

### The Thundering Herd (Cache Stampede)
*   **Event**: A "Hot Key" (e.g., Homepage Config) expires.
*   **Impact**: 1000 requests miss cache simultaneously. 1000 requests hit DB. DB dies.
*   **Fix**: **Probabilistic Early Expiration** or **Locking** (only 1 thread rebuilds cache).

---

## 14. Summary & Architect Takeaways

*   **There are only 2 hard things in CS**: Cache Invalidation and Naming things.
*   **Staleness**: Caching means serving old data. How old is acceptable? 1 second? 1 hour? Define SLAs.
