# Caching Systems

> **Part 3: Core Components**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Speed Demon

---

## 0. Learning Objectives
*   **Beginner**: Understand why reading from RAM is 1000x faster than reading from Disk.
*   **Developer**: Implement Cache-Aside pattern in code.
*   **Architect**: Design a distributed caching strategy that handles strict consistency requirements.

---

## 1. Problem Context
**Why does this exist?**
Query: `SELECT * FROM Products WHERE category = 'Electronics'`.
*   **Database**: Scans disk. Parses SQL. Locks rows. Returns result in **50ms**.
*   **1M Users**: Database catches fire.
**Cache**: Stores the result in RAM (HashMap). Returns result in **0.5ms**.
*   **Goal**: Reduce Latency and Protect the Database.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Local Cache (In-Memory)
*   **Definition**: Storing data in the variable/heap of the application itself.
*   **Example**: `HashMap<String, Object>`.
*   **Speed**: Nanoseconds.
*   **Risk**: If server restarts, data is lost. Data is duplicated across servers.

### 2. Distributed Cache
*   **Definition**: A separate server (Redis/Memcached) dedicated to storing KV pairs in RAM.
*   **Speed**: Milliseconds (Network call).
*   **Benefit**: Shared across all App Servers.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Caching Strategies

1.  **Cache-Aside (Lazy Loading)**:
    *   App checks Cache.
    *   If Miss -> App checks DB -> App writes to Cache.
    *   *Pros*: Only requested data is cached.
    *   *Cons*: First request is slow (Miss). Data can get stale.

2.  **Write-Through**:
    *   App writes to Cache AND DB at the same time.
    *   *Pros*: Consistency. No stale data on read.
    *   *Cons*: High write latency (2 writes).

3.  **Write-Behind (Write-Back)**:
    *   App writes to Cache. Cache writes to DB later (Async).
    *   *Pros*: Ultra-fast writes.
    *   *Cons*: **Data Loss** if Cache crashes before syncing to DB.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Eviction Policies
RAM is expensive and finite. You must delete old data.
*   **LRU (Least Recently Used)**: Delete the item that hasn't been accessed for the longest time. (Standard).
*   **LFU (Least Frequently Used)**: Delete the item with fewest hits. (Good for stable popularity).
*   **TTL (Time To Live)**: Delete item after X seconds. (absolute expiration).

### 2. The Thundering Herd (Cache Stampede)
*   **Scenario**: 1000 concurrent users request Key "A".
*   **Event**: Key "A" expires.
*   **Result**: 1000 users get "Cache Miss". 1000 users hit the DB simultaneously. DB crashes.
*   **Fix**: **Probabilistic Early Expiration** (Refresh before TTL) or **Request Coalescing** (Single Flight).

---

## 5. Trade-Off Analysis

| Strategy | Speed | Consistency | Use Case |
| :--- | :--- | :--- | :--- |
| **Local Cache** | Highest | Lowest (Drift) | Static Configs, Tokens. |
| **Distributed Cache** | High | Medium | User Sessions, Product Data. |
| **Write-Back** | Highest Write | Low Durability | Likes, Counters, Analytics. |

---

## 6. Scaling Considerations

### Redis Cluster (Sharding)
*   Redis is single-threaded. It hits a CPU wall.
*   **solution**: Shard data across 10 Redis nodes.
*   **Hash Slot**: `CRC16(key) % 16384`. Determines which node holds the key.

---

## 7. Failure Scenarios & Recovery

### 1. Cache Avalanche
*   **Scenario**: You set TTL = 1 hour for ALL keys.
*   **Event**: At 1:00 PM, 1 Million keys expire simultaneously.
*   **Result**: Massive spike in DB load.
*   **Fix**: **Jitter**. Set TTL = 1 hour + Random(0-5 mins).

---

## 8. Security Considerations

### 1. No Encryption by Default
*   Redis protocol is often unencrypted.
*   If an attacker is inside the VPC, they can `FLUSHALL` or read user sessions.
*   **Fix**: Use Redis AUTH (ACLs) and TLS.

---

## 9. Performance Considerations

*   **Serialization**: Converting Objects to JSON/Bytes takes CPU.
*   **Network**: Large values (1MB HTML) block the Redis thread.
*   *Rule*: Keep keys short and values small (<100KB).

---

## 10. Real Production Lessons

### Facebook's "Lease" Mechanism
*   **Problem**: Stale Sets.
*   **Scenario**: Thread A reads DB. Thread B writes DB. Thread B updates Cache. Thread A overwrites Cache with old data.
*   **Fix**: Memcached "Lease". Only the thread that got the "Miss" is allowed to write.

---

## 11. Interview Questions

### Basic
1.  Why is Caching faster than DB?
2.  What is TTL?
3.  Difference between LRU and LFU.
4.  What is a Cache Hit?
5.  What is Memcached?

### Intermediate
1.  Explain Cache-Aside vs Write-Through.
2.  How do you handle Cache Invalidation? (The hard problem).
3.  What is a Cache Avalanche?
4.  Why is Redis Single-Threaded?
5.  How do you cache in a Microservices architecture? (Sidecar vs Central).

### Advanced
1.  Design a Distributed Rate Limiter using Redis.
2.  Analyze the consistency trade-offs of Write-Behind caching.
3.  How does Consistent Hashing minimize rebalancing in a Cache Cluster?
4.  Explain the "Hot Key" problem in a sharded cache and how to fix it (Local Cache replica).
5.  Critique the use of "Cache as a Database" (Persistence enabled).

### Architect-Level
1.  "We have 10TB of hot data." Architect a tiered caching solution (RAM + SSD/NVMe).
2.  Design a cache warm-up strategy for a new datacenter deployment.
3.  Evaluate the impact of False Sharing in CPU caches.

---

## 12. Scenario-Based System Design Problems

### 1. Design a Leaderboard
*   **Req**: Top 10 users.
*   **Solution**: Redis `Sorted Set` (ZSET). `ZADD user score`. `ZREVRANGE 0 9`. O(log N).

### 2. Design a Session Store
*   **Req**: 10M active users.
*   **Solution**: Redis Cluster. Key=`SessionID`. Value=`UserProfile`. TTL=30 mins sliding.

---

## 13. Summary & Architect Takeaways

1.  **There are only two hard things in CS**: Cache Invalidation and Naming Things.
2.  **Memory is volatile**: Never treat Cache as the Source of Truth.
3.  **Protect the DB**: The Cache is the DB's shield. If Cache dies, drop traffic, don't let it hit the DB.
