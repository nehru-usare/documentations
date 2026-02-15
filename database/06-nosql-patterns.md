# 🌐 NoSQL Patterns & Distributed Systems

> **Document Level:** Senior → Architect  
> **Last Updated:** February 2026 | **Prerequisites:** [05-storage-internals.md](./05-storage-internals.md)

---

## 📋 Table of Contents

1.  [The 4 NoSQL Data Models](#the-4-nosql-data-models)
2.  [CAP Theorem Visualized](#cap-theorem-visualized)
3.  [Data Distribution: Consistent Hashing](#data-distribution-consistent-hashing)
4.  [Tunable Consistency (Quorums)](#tunable-consistency-quorums)

---

## The 4 NoSQL Data Models

Not all NoSQL is the same. Choose the right tool.

| Type | Examples | Best For | Data Model |
|:---|:---|:---|:---|
| **Key-Value** | Redis, DynamoDB | Caching, User Sessions | HashMap<K, V> |
| **Document** | MongoDB, Couchbase | CMS, Catalogs, User Profiles | JSON / BSON |
| **Wide-Column** | Cassandra, HBase | High-Velocity Writes, Logs, IoT | 2D Map (RowKey -> ColFamily -> Col -> Val) |
| **Graph** | Neo4j, Amazon Neptune | Social Networks, Fraud Detection | Nodes & Edges |

### Architect Decision Matrix
*   **Need ACID?** -> Stay with SQL (Postgres).
*   **Need flexible schema + fast dev?** -> Document (Mongo).
*   **Need 1M writes/sec?** -> Wide-Column (Cassandra).
*   **Need recursive relationship traversal?** -> Graph (Neo4j).

---

## CAP Theorem Visualized

In a Distributed System (Network Partition is inevitable), you can only pick two: **C** or **A**.

*   **Consistency (CP)**: Every read receives the most recent write or an error. (e.g., Banking).
    *   *Trade-off*: If a node is down, the system stops to prevent bad data.
    *   *Examples*: HBase, MongoDB (default), Redis (single).
*   **Availability (AP)**: Every request receives a (non-error) response, without the guarantee that it contains the most recent write. (e.g., Facebook Likes).
    *   *Trade-off*: You might see old data (Eventual Consistency).
    *   *Examples*: Cassandra, DynamoDB, DNS.

---

## Data Distribution: Consistent Hashing

How do we split 100TB data across 10 nodes?

### Simple Hashing (Modulo)
`Node = Hash(Key) % 10`
*   **Problem**: If you add Node 11, *everything* moves. `Hash(Key) % 11` changes almost all locations. Massive reshuffling.

### Consistent Hashing (The Ring)
Used by **Cassandra, DynamoDB, Discord**.
1.  Imagine a Ring (0 to 2^32).
2.  Place Nodes on the Ring at random points.
3.  Place Data Key on the Ring.
4.  Walk clockwise to find the first Node. That node owns the data.
*   **Benefit**: Adding a node only steals keys from its neighbor. Minimal data movement.

---

## Tunable Consistency (Quorums)

In Leaderless systems (Cassandra), you can choose your consistency level per query.

**Formula**: `R + W > N`
*   **N**: Replication Factor (Total copies, e.g., 3).
*   **W**: Write Quorum (How many must acknowledge write).
*   **R**: Read Quorum (How many must respond to read).

### Scenarios (N=3)
1.  **Strong Consistency**:
    *   **Write**: `ALL` (Wait for 3). **Read**: `ONE` (Wait for 1).
    *   *Slow Writes, Fast Reads*.
2.  **Balanced (Quorum)**:
    *   **Write**: `QUORUM` (2). **Read**: `QUORUM` (2).
    *   `2 + 2 > 3`. We are mathematically guaranteed to see the latest data.
3.  **High Availability (Eventual)**:
    *   **Write**: `ONE` (Fire and forget). **Read**: `ONE`.
    *   *Blazing fast, but high risk of stale data*.
