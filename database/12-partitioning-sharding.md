# Database Partitioning & Sharding Strategies

> **Part 3: Scaling**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Architect)  
> **Status:** Splitting...

---

## 0. Learning Objectives
*   **Beginner**: Why splitting a 1TB table into 10 chunks makes queries faster.
*   **Developer**: Coding for a sharded database (routing queries).
*   **Architect**: Choosing between Client-Side Sharding and Proxy Sharding (Vitess/Citus).

---

## 1. Core Concepts (🟢 Beginner Level)

### Partitioning vs Sharding
*   **Partitioning**: Splitting a table *within the same database instance*.
    *   Example: `orders_2024`, `orders_2025`.
*   **Sharding**: Splitting a table *across multiple database servers*.
    *   Example: `User A-M` on Server 1. `User N-Z` on Server 2.

---

## 2. Architecture Breakdown (Partitioning)

### 1. Range Partitioning
*   Split by value range. (Date, ID).
*   *Pros*: Easy to drop old data (`DROP TABLE orders_2020`).
*   *Cons*: Hotspots if everyone writes to "Today".

### 2. List Partitioning
*   Split by category. (Region: US, EU, ASIA).

### 3. Hash Partitioning
*   `Partition = Hash(Key) % N`.
*   Ensures even distribution. Hard to query ranges.

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. Partition Pruning
*   Query: `SELECT * FROM orders WHERE date = '2025-01-01'`.
*   DB Optimizer knows this date is ONLY in `orders_2025`.
*   It **skips** scanning all other partitions. *Massive speedup*.

### 2. Sharding Architectures
*   **Client-Side**: App Logic knows "User 1 is on DB1".
    *   *Cons*: Complex code. connection pool explosion.
*   **Proxy-Side (Vitess/ProxySQL/Citus)**: App talks to Proxy. Proxy routes query to correct Shard.
    *   *Pros*: Transparent to App.

---

## 4. Trade-Off Analysis

| Strategy | Complexity | Scale | Joins? |
| :--- | :--- | :--- | :--- |
| **Partitioning** | Low (Built-in) | Verified (Single Node limit) | Yes |
| **Sharding** | **Very High** | Infinite | **No** (Cross-shard joins are impossible/slow) |

---

## 5. Scaling Considerations

### The Resharding Nightmare
*   You have 10 Shards. You need 20.
*   **Consistent Hashing**: Minimizes data movement.
*   **Migration**:
    1.  Start Dual Write (Old + New).
    2.  Backfill Data.
    3.  Switch Reads.
    4.  Stop Old.

---

## 6. Failure Scenarios & Recovery

### 1. Cross-Shard Transactions
*   User 1 (Shard A) sends money to User 2 (Shard B).
*   **Problem**: If Shard A commits but Shard B fails -> Money lost.
*   **Fix**: 2-Phase Commit (2PC). Slow. Avoid it.

---

## 7. Security Considerations

### 1. Data Residency
*   Sharding by "Region" allows compliance with GDPR.
*   "EU Users" stay on "EU Shard" (physically in Frankfurt).

---

## 8. Performance Considerations

*   **Scatter-Gather**:
    *   Query: `SELECT * FROM users WHERE name = 'Bob'`.
    *   If you don't stick to the Shard Key (e.g., UserID), the Proxy must query **ALL** shards and aggregate results.
    *   *Performance Disaster*.

---

## 9. Real Production Lessons

### "Instagram ID"
*   Instagram generates IDs that contain the Shard ID inside them.
*   `ID = <Timestamp> + <ShardID> + <Sequence>`.
*   App knows exactly where to find the data just by looking at the ID.

---

## 10. Interview Questions

### Basic
1.  Partitioning vs Sharding.
2.  What is Partition Pruning?
3.  Why are joins hard in sharding?

### Intermediate
1.  Explain Consistent Hashing.
2.  How to handle Auto-Increment IDs in a sharded setup? (Snowflake).
3.  Range vs Hash Sharding.

### Advanced
1.  Design a Resharding workflow with zero downtime.
2.  How does Vitess/Citus handle cross-shard aggregations (`SUM(amount)`)?
3.  Architect a Multi-Tenant SaaS DB schema (Shared DB vs Schema-per-tenant vs DB-per-tenant).

---

## 11. Summary & Architect Takeaways

1.  **Don't Shard**: Sharding is the last resort. Optimize/Partition first.
2.  **Pick Key Carefully**: Changing the Shard Key later is nearly impossible.
3.  **No Joins**: If you shard, you lose SQL Joins. Prepare your app logic.
