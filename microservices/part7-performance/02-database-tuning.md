# 02. Database Tuning (Indexing, Connection Pooling, Sharding)

> **Part 7: Performance & Scalability**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (DBA / Architect)  
> **Status:** Critical Performance

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Learn to read `EXPLAIN ANALYZE`. |
| **Developer** | Tune HikariCP (`maximumPoolSize`). |
| **Architect** | Design a Sharding strategy for 10TB+ data. |

---

## 1. Why This Topic Exists

### The Slow Query
A single slow query (Full Table Scan) can lock the DB and kill the entire microservice.
*   **Goal**: Queries should run in < 10ms (Point lookup) or < 100ms (Range scan).

---

## 3. Core Concepts (🟢 Beginner Level)

### Indexing 101
*   **B-Tree**: Good for `=`, `>`, `<`. (Default).
*   **Hash**: Good for `=`. Bad for ranges.
*   **Composite Index**: `Index(A, B)`.
    *   Query `WHERE A=1 AND B=1` -> Fast.
    *   Query `WHERE A=1` -> Fast.
    *   Query `WHERE B=1` -> **Slow** (Prefix rule).

### Connection Pooling (HikariCP)
Opening a TCP connection to Postgres takes 50ms.
*   **Pool**: Keep 10 connections open and reuse them.
*   **Bottleneck**: If Pool is empty, threads wait.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### HikariCP Configuration
The most important config in Spring Boot.

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10  # Don't set to 100!
      minimum-idle: 10
      idle-timeout: 300000
      max-lifetime: 1800000 # 30 mins
      connection-timeout: 30000 # Wait 30s before throwing Exception
```

*   **Why small pool?**: `Pool Size = ((Core Count * 2) + Effective Spindle Count)`.
    *   For a 4-core CPU, pool size 10 is enough to saturate I/O. 100 connections just adds Context Switch overhead.

---

## 5. Internal Mechanics (🔴 Architect Level)

### Sharding (Horizontal Partitioning)
Splitting one big table across multiple servers.
*   **Sharding Key**: `UserId`.
*   **Algorithm**: `Hash(UserId) % 4`.
    *   User 1 -> Node A.
    *   User 2 -> Node B.
*   **Problem**: **Resharding**. If you add Node C, you have to move 33% of data. (Consistent Hashing helps).

---

## 9. Architect-Level Best Practices

1.  **Read Replicas**: Offload Reporting/Analytics to a Read Replica. Keep Primary for Writes.
2.  **No Logic in DB**: No Stored Procedures. Logic belongs in Java. ( easier to scale Java than Oracle).
3.  **Vacuum**: Ensure Postgres `VACUUM` is running to clean up dead tuples.

---

## 14. Summary & Architect Takeaways

*   **Index Everything?**: No. Indexes slow down Writes. Index only what you search by.
*   **Pooling**: "More connections" is usually the wrong answer. Fix the slow query holding the connection.
