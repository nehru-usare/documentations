# Consistent Hashing

> **Part 4: Data Scaling**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** The Magic Ring

---

## 0. Learning Objectives
*   **Beginner**: Understand why `HashCode % N` is bad when `N` changes.
*   **Developer**: Implement a hash ring with a SortedMap.
*   **Architect**: Design a load balancer or cache cluster that minimizes remapping during scaling events.

---

## 1. Problem Context
**Why does this exist?**
You have 4 Cache Servers.
You map keys using `Hash(key) % 4`.
*   Server 1: 0-25
*   Server 2: 26-50
*   ...
**Server 3 Dies**.
Now you compute `Hash(key) % 3`.
*   **Disaster**: Almost EVERY key maps to a new server.
*   **Result**: 100% Cache Miss rate. Database melts.
**Consistent Hashing** ensures only $k/N$ keys move.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. The Ring
*   Imagine a circle with values 0 to $2^{32}-1$.
*   Servers are placed on the ring based on `Hash(ServerIP)`.
*   Keys are placed on the ring based on `Hash(Key)`.

### 2. The Rule
*   To find the server for a Key, go **Clockwise** on the ring until you find a Server.
*   If Server A dies, its keys fall to Server B (Next in line).
*   Keys on Server C and D are unaffected.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Virtual Nodes (Netfix)
*   **Problem**: If you have few servers on the ring, the gaps are uneven.
    *   Server A covers 10% of ring.
    *   Server B covers 90% of ring.
*   **Solution**: Virtual Nodes.
    *   Server A appears as A1, A2, A3... A100.
    *   Server B appears as B1, B2, B3... B100.
    *   Mix them thoroughly. Distribution becomes uniform.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Replication Strategy
*   In Dynamo/Cassandra, data is replicated to $N$ nodes.
*   **How?**
    *   Find the Coordinator (First node clockwise).
    *   Walk clockwise and pick the next $N-1$ distinct physical nodes.

### 2. Implementation
*   **Java**: `TreeMap` (Red-Black Tree).
    *   `Map<Integer, Node> ring.`
    *   `tailMap(hash).firstKey()` finds the next node.

---

## 5. Trade-Off Analysis

| Strategy | Rebalancing Cost | Complexity | Load Distribution |
| :--- | :--- | :--- | :--- |
| **Modulo Hashing** | High (100% keys move) | Extremely Low | Perfect (Uniform) |
| **Consistent Hashing** | Low (1/N keys move) | High (Ring logic) | Uneven (Needs VNodes) |

---

## 6. Scaling Considerations

### Adding a Node
*   You add Server E.
*   It places itself on the ring between A and B.
*   It "steals" some keys from B.
*   Only B loses data. A, C, D are untouched.
*   **Impact**: Cache miss rate increases by only $1/N$.

---

## 7. Failure Scenarios & Recovery

### 1. Cascading Failure
*   Server A dies.
*   All its load shifts to Server B.
*   Server B gets overloaded and dies.
*   All load shifts to Server C.
*   **Fix**: **Virtual Nodes** spread the load of a dead node across ALL surviving nodes, not just the neighbor.

---

## 8. Security Considerations

### 1. Hash Flooding
*   If attacker knows your Hash Function, they can generate keys that all map to Server A.
*   **Fix**: Use a cryptographic hash (SHA-256) or a secret seed (SipHash) so attackers can't predict ring positions.

---

## 9. Performance Considerations

*   **Lookup Speed**: Finding a node is $O(\log S)$ where S is number of servers (Binary Search on Ring coverage).
*   **Memory**: Storing millions of Virtual Nodes takes RAM.

---

## 10. Real Production Lessons

### Amazon Dynamo
*   The paper that introduced Consistent Hashing to the masses.
*   Used "Preference Lists" to handle temporary failures (Hinted Handoff).

---

## 11. Interview Questions

### Basic
1.  Why is Modulo hashing bad for Distributed Caching?
2.  What is the "Ring" in Consistent Hashing?
3.  What happens when a node is added?
4.  What is a Virtual Node?
5.  What hash functions are suitable? (MD5, SHA, MurmurHash).

### Intermediate
1.  Implement Consistent Hashing in pseudocode.
2.  How many Virtual Nodes should one server have? (~100-200).
3.  How does this relate to Partitioning?
4.  Explain "Hot Shards" in Consistent Hashing.
5.  Database vs Cache use cases for Consistent Hashing.

### Advanced
1.  Analyze the "Shed Logic" when a node is overloaded. (Bounded Load Consistent Hashing - Google).
2.  Design a load balancer using Consistent Hashing for long-lived WebSocket connections.
3.  Compare Rendezvous Hashing (HRW) vs Consistent Hashing.
4.  How to handle heterogenous servers (some big, some small)? (Assign more VNodes to big servers).
5.  Derive the standard deviation of load with X virtual nodes.

### Architect-Level
1.  "We are moving from Memcached (Modulo using client driver) to Redis Cluster (Hash Slots)." Plan the migration.
2.  Critique the use of Consistent Hashing in stateful TCP proxies.
3.  Design a Global Service Discovery using a specialized Ring.

---

## 12. Scenario-Based System Design Problems

### 1. Design a Distributed Key-Value Store
*   **Req**: Scale to 1000 nodes.
*   **Core**: Consistent Hashing with VNodes.
*   **Rep**: N=3 Preference List.

### 2. Design a CDN Request Router
*   **Req**: Route user to Cache Server based on URL.
*   **Core**: Consistent Hashing on URL.
*   **Benefit**: If a Cache Server is added, only minimal cache segments are invalidated.

---

## 13. Summary & Architect Takeaways

1.  **Standard for Distributed State**: If you hold state, you need Consistent Hashing.
2.  **Virtual Nodes are Mandatory**: Without them, load balancing is terrible.
3.  **Minimize Churn**: The goal is stability during change.
