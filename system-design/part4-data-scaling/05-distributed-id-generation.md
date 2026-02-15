# Distributed ID Generation

> **Part 4: Data Scaling**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Uniquely Yours

---

## 0. Learning Objectives
*   **Beginner**: Why Auto-Increment (1, 2, 3) fails in distributed systems.
*   **Developer**: Use UUIDs vs Snowflakes.
*   **Architect**: Implementing a custom ID generator server (Like Twitter Snowflake).

---

## 1. Problem Context
**Why does this exist?**
Single DB: `AUTO_INCREMENT`. Works fine.
Sharded DB:
*   Shard A generates ID 100.
*   Shard B generates ID 100.
*   **Collision**. Primary Key Violation.
You need a way to generate Unique IDs without a central coordinator (SPOF).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. UUID (Universally Unique Identifier)
*   128-bit number. (e.g., `123e4567-e89b-12d3-a456-426614174000`).
*   Generated randomly or using MAC address.
*   *Pros*: Zero coordination. Infinite scale.
*   *Cons*: Huge (16 bytes). Not sortable by time. Bad for B-Tree Performance.

### 2. Sortable IDs
*   Essential for performant DBs.
*   New IDs should be "bigger" than old IDs (Time-ordered).
*   Prevents Page Splitting in B-Trees.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Topologies

1.  **Ticket Server (Flickr Strategy)**:
    *   Central DB dedicated to handing out IDs. `REPLACE INTO Tickets (bit) VALUES ('a')`.
    *   *Pros*: Simple numeric IDs (64-bit).
    *   *Cons*: SPOF. Latency.

2.  **Twitter Snowflake**:
    *   64-bit Long.
    *   **Time (41 bits)** + **Node ID (10 bits)** + **Sequence (12 bits)**.
    *   *Pros*: K-Sortable. Distributed. High Performance.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Snowflake Bit Allocation
*   **Sign Bit**: 1 bit (Unused).
*   **Timestamp**: 41 bits (Milliseconds since epoch). Lasts 69 years.
*   **Datacenter ID**: 5 bits (32 DCs).
*   **Worker ID**: 5 bits (32 Machines per DC).
*   **Sequence**: 12 bits (4096 IDs per millisecond per machine).

### 2. Clock Synchronization (NTP)
*   Snowflake relies on System Clock.
*   **Clock Move Backwards**: If NTP resets clock back 1 second, you might generate duplicate IDs.
*   *Fix*: Refuse to generate IDs until clock catches up.

---

## 5. Trade-Off Analysis

| Strategy | Size | Sortable? | Coordination |
| :--- | :--- | :--- | :--- |
| **Auto-Increment** | Small (8B) | Yes | Central DB (Strict) |
| **UUID (v4)** | Huge (16B) | No | None |
| **UUID (v7)** | Huge (16B) | Yes | None |
| **Snowflake** | Small (8B) | Yes | Node Config |

---

## 6. Scaling Considerations

### Throughput
*   **Snowflake**: 4096 IDs / ms / node = 4 Million IDs / sec.
*   Infinite scale by adding more Worker Nodes.

---

## 7. Failure Scenarios & Recovery

### 1. Node ID Conflict
*   **Scenario**: Dev accidentally configures Machine A and Machine B with same Worker ID `5`.
*   **Result**: If they generate ID at same millisecond -> Collision.
*   **Fix**: Use Zookeeper/Etcd to dynamically assign Worker IDs on boot.

---

## 8. Security Considerations

### 1. Information Leakage
*   **UUID v1 / Snowflake**: Contains Timestamp and sometimes MAC address.
*   Competitors can calculate your growth rate ("User ID today - User ID yesterday").
*   *Fix*: Use random UUID (v4) for Public facing IDs. Use Snowflake for Internal DB Keys.

---

## 9. Performance Considerations

*   **B-Tree Fragmentation**: Random UUIDs cause inserts to jump around the B-Tree, loading random pages from disk.
*   **Sequential IDs**: Inserts always append to the right side of the B-Tree. Cache friendly.

---

## 10. Real Production Lessons

### Instagram's ID
*   **Req**: Sortable, 64-bit.
*   **Solution**: Modified Snowflake using Postgres stored function.
*   Shard ID is part of the ID. Helps with mapping `ID -> Shard`.

---

## 11. Interview Questions

### Basic
1.  Why not use UUIDs as Primary Keys? (Size, Fragmentation).
2.  What is a GUID? (MS terminology for UUID).
3.  How many bits in a Java Long? (64).
4.  Why do we need unique IDs in distributed systems?
5.  Can UUIDs collide? (Mathematically yes, practically no).

### Intermediate
1.  Explain the parts of a Snowflake ID.
2.  What is a Ticket Server?
3.  How do you handle clock drift in ID generation?
4.  What is "K-Sortable"? (Roughly sorted).
5.  Why is 128-bit bad for MySQL InnoDB? (Primary key is duplicated in every secondary index).

### Advanced
1.  Design a completely decentralized, sortable ID generator (UUID v7).
2.  How to upgrade from 32-bit INT to 64-bit BIGINT in a live database with 1B rows?
3.  Calculate the lifespan of Twitter Snowflake (69 years). What happens after?
4.  Implement a Snowflake algorithm that handles NTP backward jumps safely.
5.  Compare Zookeeper vs Etcd as a coordinator for Worker IDs.

### Architect-Level
1.  "We are merging with another company. They also use Snowflake. How do we merge DBs?" (Hope Datacenter IDs were different, or re-key everything).
2.  Design an ID system for an Offline-First mobile app syncing to cloud. (Client-generated UUIDs are mandatory).
3.  Critique the use of MongoDB `ObjectId` (it includes timestamp, machine, pid, counter).

---

## 12. Scenario-Based System Design Problems

### 1. Design TinyURL
*   **Req**: Short URL (7 chars).
*   **ID**: Base62 Encode(Snowflake ID).
*   **Length**: 64-bit integer -> ~11 chars Base62. Too long.
*   **Enhancement**: Custom shorter ID generator or pre-generated ranges.

### 2. Design Chat Message ID
*   **Req**: Ordering.
*   **Choice**: Snowflake.
*   **Why**: Messages must be sorted by time. Using Timestamp as ID allows easy `ORDER BY id`.

---

## 13. Summary & Architect Takeaways

1.  **Snowflake is Gold Standard**: For internal DB keys, use Snowflake (or ULID/UUIDv7).
2.  **Don't coordinate**: Ticket servers are a bottleneck. ID generation should be local to the node.
3.  **Hide the volume**: Don't expose sequential IDs to the public unless you want to leak your business metrics.
