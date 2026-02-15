# Database Sharding

> **Part 4: Data Scaling**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** Breaking the Monolith

---

## 0. Learning Objectives
*   **Beginner**: Understand why we split one big database into many small ones.
*   **Developer**: Implement "Application-Side Sharding" using a Routing logic.
*   **Architect**: Design a Resharding strategy that doesn't require downtime.

---

## 1. Problem Context
**Why does this exist?**
Your Postgres DB is 5TB.
Queries are slow. Indexes don't fit in RAM.
You buy the biggest server AWS offers (`x1e.32xlarge`).
It's still too slow.
**Solution**: Split the data across 10 servers.
*   Server 1: Users A-C.
*   Server 2: Users D-F.
...

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Vertical Scaling (Scale Up)
*   Buying a bigger CPU/RAM.
*   *Limit*: Physics and Cost.

### 2. Horizontal Scaling (Sharding)
*   Adding more servers.
*   *Limit*: Complexity.

### 3. Shard Key (Partition Key)
*   The column determines which server the data goes to.
*   *Example*: `user_id`, `region`, `date`.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Sharding Strategies

1.  **Key Based (Hash)**: `ShardID = Hash(UserID) % TotalShards`.
    *   *Pros*: Uniform distribution.
    *   *Cons*: **Resharding** is painful. If you go from 10 to 11 shards, nearly ALL keys move.

2.  **Range Based**: `IDs 1-1000` -> Shard 1. `IDs 1001-2000` -> Shard 2.
    *   *Pros*: Easy range queries (`WHERE id BETWEEN...`).
    *   *Cons*: **Hot Spots**. If Shard 1 is "Old Users" and Shard 10 is "New Users", Shard 10 takes all the write traffic.

3.  **Directory Based (Lookup Table)**:
    *   A separate DB holds a table: `UserID -> ShardID`.
    *   *Pros*: Total flexibility. Can move individual users.
    *   *Cons*: Single Point of Failure (The Lookup DB). Added Latency.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. The Join Problem
*   Query: `SELECT * FROM User u JOIN Orders o ON u.id = o.user_id`.
*   If User is on Shard 1 and Order is on Shard 2, **Database cannot Join**.
*   *Fix*: **De-normalization** or App-Side Joins.

### 2. Global Unique IDs
*   Auto-Increment `IDs` (1, 2, 3) don't work across shards (Collision).
*   *Fix*: **Snowflake IDs** (Time + MachineID + Sequence).

---

## 5. Trade-Off Analysis

| Strategy | Complexity | Performance | Write Scale |
| :--- | :--- | :--- | :--- |
| **Monolith DB** | Low | Low | Low |
| **Read Replicas** | Medium | High (Read) | Low (Write) |
| **Sharding** | **Very High** | **Infinite** | **Infinite** |

---

## 6. Scaling Considerations

### Resharding (The Nightmare)
*   You have 10 Shards. You need 20.
*   **Hash Sharding**: You must migrate 50-90% of data.
*   **Strategy**: Prefer **Linear Splitting**.
    *   Shard 1 splits into 1A and 1B.
    *   Shard 2 splits into 2A and 2B.
    *   Only data on Shard 1 moves to 1B. Shard 2 is untouched.

---

## 7. Failure Scenarios & Recovery

### 1. The "Celebrity" Hot Key
*   **Scenario**: Shard Key = `user_id`. Katy Perry (`user_id=100`) is on Shard 5.
*   **Event**: She tweets. 10M people comment.
*   **Result**: Shard 5 melts. Shards 1-4 and 6-10 are idle.
*   **Fix**: Add "Salt" to the Shard Key for high-volume rows. Spread the load.

---

## 8. Security Considerations

### 1. Cross-Shard Leakage
*   In Multi-Tenant SaaS, verify that Shard Logic *never* routes Tenant A's query to Tenant B's shard.

---

## 9. Performance Considerations

*   **Scatter-Gather**: Querying "All Users" requires hitting ALL shards.
*   **Latency**: The response time is determined by the *slowest* shard.
*   *Rule*: Avoid Scatter-Gather queries in hot paths. Use a separate Search Index (Elasticsearch) for searching.

---

## 10. Real Production Lessons

### Foursquare's Outage
*   **Event**: Hit Shard limit.
*   **Action**: 17 hour downtime to move data to new shards.
*   **Lesson**: Virtual Shards (Logical Shards). Map 1000 Logical Shards to 10 Physical Servers. Moving Logical Shards is easier.

---

## 11. Interview Questions

### Basic
1.  What is Sharding?
2.  Difference between Sharding and Partitioning. (Often used interchangeably, but Partitioning is usually local to 1 server).
3.  Why can't I use Auto-Increment IDs in sharding?
4.  What is a Hot Key?
5.  How do you perform a Join across shards?

### Intermediate
1.  Explain Hash vs Range sharding.
2.  What is a "Routing Tier"?
3.  How does Consistent Hashing help sharding?
4.  What is "Data Locality"? (Keeping related data on the same shard).
5.  How do you handle Transactions across shards? (2PC - Avoid it!).

### Advanced
1.  Design a Resharding process with Zero Downtime.
2.  Architect a "Hierarchical Sharding" strategy (Geo > Hash).
3.  Analyze the impact of Sharding on Connection Pooling.
4.  Explain Vitess (YouTube's MySQL sharding middleware).
5.  How to shard a Many-to-Many relationship (Followers)?

### Architect-Level
1.  "We have 100TB data. Postgres or Cassandra?" (Cassandra handles sharding natively. Postgres requires manual/Citus setup).
2.  Design the data layer for a Global Multi-Tenant SaaS (Data Residency Laws + Sharding).
3.  Critique the "Directory Based" sharding approach.

---

## 12. Scenario-Based System Design Problems

### 1. Design Uber Trip History
*   **Shard Key**: `city_id`?
    *   *Problem*: New York is huge. Kansas is small. Unbalanced.
*   **Shard Key**: `trip_id` (Random Hash)?
    *   *Problem*: Good distribution, but querying "My Trips" requires Scatter-Gather.
*   **Verdict**: `user_id`. All my trips live on my shard.

### 2. Design Discord Messages
*   **Shard Key**: `channel_id`.
*   *Why*: All messages for a channel must be time-ordered and fetched together.

---

## 13. Summary & Architect Takeaways

1.  **Don't Shard until you must**: It introduces massive complexity. Try Read Replicas first.
2.  **Pick the Key carefully**: You effectively cannot change it later.
3.  **Avoid Distributed Transactions**: Design your schema so transactions stay within a single shard.
