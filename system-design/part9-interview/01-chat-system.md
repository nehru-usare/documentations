# Design a Chat System (WhatsApp/Slack)

> **Part 9: Interview Scenarios**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** The Classic

---

## 0. Learning Objectives
*   **Beginner**: Why HTTP is bad for Chat (Polling).
*   **Developer**: Managing WebSocket connections statefully.
*   **Architect**: Scaling to 1 Billion Users (CS10K problem, Storage sharding).

---

## 1. Problem Context
**The Ask**: Design WhatsApp.
*   **Scale**: 1 Billion DAU.
*   **Features**: 1-on-1 Text, Group Chat, Online Status, Sent/Delivered/Read Receipts.
*   **Constraint**: Low Latency (Real-time).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Connection Protocol
*   **HTTP**: Request-Response. Client asks "Any new messages?" (Polling). *Inefficient*.
*   **WebSocket**: Persistent bidirectional TCP connection. Server pushes messages to Client. *Efficient*.

### 2. Stateful vs Stateless
*   Web Services are usually Stateless.
*   Chat Servers are **Stateful**. They know which user is connected to which server.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Components
1.  **Chat Service** (Stateful): Holds WebSocket connections.
2.  **Presence Service**: Manages Online/Offline status.
3.  **Message Store**: Database for chat history.
4.  **Push Notification Service**: For users who are offline (FCM/APNS).
5.  **Service Discovery (Zookeeper)**: Tracks which Chat Server holds User A's connection.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. The Message Flow (User A -> User B)
1.  A sends msg to Chat Server 1.
2.  Chat Server 1 asks Service Discovery: "Where is User B?"
3.  **Scenario 1 (B is Online)**: Discovery says "CS-2". CS-1 forwards msg to CS-2 (via Redis Pub/Sub or RPC). CS-2 pushes to B via WebSocket.
4.  **Scenario 2 (B is Offline)**: Push to Message Store. Trigger Push Notification.

### 2. Group Chat Maxima
*   **Small Group (<100)**: Client sends 1 msg. Server loops 100 times and pushes to members.
*   **Mega Group (100k)**: Do not push. Client "Pulls" messages when they open the app. (Hybrid approach).

---

## 5. Trade-Off Analysis

| Feature | SQL (MySQL) | NoSQL (Cassandra/HBase) |
| :--- | :--- | :--- |
| **Write Throughput** | Low (B-Tree splits) | Extremely High (LSM Tree) |
| **Schema** | Rigid | Flexible |
| **Query Pattern** | Complex Joins | Key-Value (ChatID + Timestamp) |
| **Verdict** | **No**. Scale limits. | **Yes**. (Facebook uses HBase, Discord uses ScyllaDB). |

---

## 6. Scaling Considerations

### 1. Storage Scale
*   WhatsApp processes 60 Billion messages/day.
*   Sharding Key: `ChatID`.
*   Keep "Recent" messages in Redis (Cache). Archive old messages to HBase/S3.

### 2. Connection Limit
*   One server can handle ~100k concurrent WebSockets (with kernel tuning).
*   For 100M concurrent users -> 1000 Chat Servers.

---

## 7. Failure Scenarios & Recovery

### 1. Chat Server Failure
*   Server X dies. 100k users disconnect.
*   **Thundering Herd**: 100k users try to reconnect instantly.
*   **Fix**: **Jitter** (Random delay 0-30s) on client reconnect logic.

---

## 8. Security Considerations

### 1. End-to-End Encryption (E2EE)
*   server stores *Encrypted* blob.
*   Server *cannot* read messages.
*   Key Exchange: Signal Protocol (Double Ratchet Algorithm).
*   *Trade-off*: Search functionality must happen locally on device (or use Homomorphic Encryption - too slow).

---

## 9. Performance Considerations

*   **Heartbeats**: Keep WebSocket alive.
*   Ping/Pong every 30s.
*   If Pong checks fail -> Mark user Offline in Presence Service (Redis with TTL).

---

## 10. Real Production Lessons

### Discord
*   Started with MongoDB. Migrated to Cassandra. Then to **ScyllaDB**.
*   **Reason**: GC Pauses in Java (Cassandra) caused latency spikes. ScyllaDB (C++) solved it.

### Slack
*   Uses a "Channel" abstraction.
*   Heavy use of Edge Caching (Flannel) to reduce load on origins.

---

## 11. Interview Questions

### Basic
1.  HTTP Polling vs Long Polling vs WebSockets.
2.  How to know if a user is online?
3.  Database schema for 1-on-1 chat?
4.  What happens if user is offline?
5.  Difference between Sent, Delivered, Read.

### Intermediate
1.  Explain the flow of a Group Chat message.
2.  Why use NoSQL like Cassandra for Chat?
3.  How to handle "Typing..." indicators? (Ephemeral events, don't store in DB).
4.  How to sync messages across multiple devices? (Sequence IDs).
5.  WebSocket Load Balancing issues (Sticky Sessions).

### Advanced
1.  Design the "Last Seen" feature. (Update Redis on heartbeat. Read on chat open).
2.  How to implement End-to-End Encryption?
3.  Handle "Fan-out" for a celebrity livestream chat (1M viewers).
4.  Optimize media (Image/Video) upload flow. (Upload to S3, send URL in chat).
5.  Critique extracting the "Presence Service" vs keeping it coupled.

### Architect-Level
1.  "We have 1 Billion users. SQL Sharding vs Native NoSQL?" (Native NoSQL preferred for operational sanity).
2.  Architect a solution for "Message Search" in an E2EE system. (Client-side indexing using SQLite/Lucene).
3.  Evaluate the cost of maintaining 10M open TCP connections.

---

## 12. Scenario-Based System Design Problems

### 1. Design Telegram (Big Groups)
*   **Req**: Groups with 200k members.
*   **Arch**: Kafka topic per Group? No (Too many topics). Use Partitioned Topics.

### 2. Design Customer Support Chat
*   **Req**: Agent assignment. Session history.
*   **Arch**: Routing Engine (Assign to Agent). SQL DB (Context is valuable, scale is lower).

---

## 13. Summary & Architect Takeaways

1.  **State matters**: This is one of the few systems where you must manage Stateful connections.
2.  **Write Heavy**: The DB write load is enormous. Optimize for Write (LSM Trees).
3.  **Consistency**: Messages must appear in order. Use Monotonic Sequence IDs (Snowflake/KSUID).
