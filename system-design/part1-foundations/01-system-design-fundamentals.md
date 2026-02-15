# System Design Fundamentals

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Core Knowledge

---

## 0. Learning Objectives
*   **Beginner**: Understand the basic components of a web architecture (Client, Server, Database).
*   **Developer**: Learn how to transition from a single-server setup to a scalable, distributed architecture.
*   **Architect**: Master the trade-offs between different scaling strategies (Vertical vs Horizontal, SQL vs NoSQL, Consistency vs Availability).

---

## 1. Problem Context
**Why does this exist?**
When you build a "Hello World" app, it runs on your laptop. It handles 1 request per second.
When you build Facebook, it handles 10 million requests per second.
Your laptop catches fire.
**System Design** is the art and science of preventing your servers from catching fire while maintaining speed, reliability, and security.

**Real-World Motivation**:
*   Amazon loses $100M/hour if their system goes down.
*   Google Search must return results in <200ms or users leave.
*   Uber must match riders/drivers in real-time, globally.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Client-Server Model
*   **Client**: The browser or mobile app. Sends a request (`GET /home`).
*   **Server**: The computer listening for requests. Returns a response (`200 OK`).

### 2. Vertical Scaling (Scale Up)
*   **Definition**: Buying a bigger computer.
*   **Analogy**: You have a small car. You trade it for a Ferrari to go faster.

### 3. Horizontal Scaling (Scale Out)
*   **Definition**: Buying *more* computers.
*   **Analogy**: You have a small car. You buy 100 small cars to transport 100 people.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Evolution of Architecture

```mermaid
graph TD
    User[User] --> LB[Load Balancer]
    LB --> Web1[Web Server 1]
    LB --> Web2[Web Server 2]
    Web1 --> Cache[Redis Cache]
    Web2 --> Cache
    Web1 --> DB_Master[DB Master (Write)]
    Web2 --> DB_Master
    DB_Master --> DB_Slave[DB Slave (Read)]
```

1.  **Load Balancer**: Distributes incoming traffic across multiple servers to prevent overload.
2.  **Web Servers**: Stateless workers. You can add 1000 of them.
3.  **Cache**: Stores hot data in RAM for sub-millisecond access.
4.  **Database Replication**: Split usage into **Master** (Writes) and **Slaves** (Reads).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. State Management
*   **Stateless Architecture**: The server does NOT store user sessions (e.g., in memory). Every request contains all necessary info (JWT).
    *   *Mechanics*: Allows any request to be routed to *any* server. Essential for Horizontal Scaling.
*   **Stateful Architecture**: Sticky Sessions.
    *   *Failure Behavior*: If Server A dies, all users on Server A are logged out.

### 2. Database Bottlenecks
*   Vertical scaling has a hard limit (128 cores, 4TB RAM).
*   Horizontal scaling (Sharding) introduces network latency and consistency issues (Distributed Transactions).

### 3. Network Latency
*   **Light Speed**: Light travels 300km/ms.
*   **RTT (Round Trip Time)**: USA -> Europe -> USA is ~150ms.
*   *Implication*: A distributed transaction across continents is physically impossible to complete in <150ms.

---

## 5. Trade-Off Analysis

| Strategy | Pros | Cons | When to choose |
| :--- | :--- | :--- | :--- |
| **Vertical Scaling** | Simple. No code changes. | Expensive. Single Point of Failure (SPOF). Hard Limit. | Early stage startups. MVP. |
| **Horizontal Scaling** | Unlimited scale. High Availability. | Complex. Requires Load Balancer. Distributed Consistency issues. | Post-product-market fit. High traffic. |
| **SQL Database** | ACID Compliance. Strict structure. | Hard to scale horizontally. | Financial systems. Relational data. |
| **NoSQL Database** | Easy horizontal scaling. Flexible schema. | Eventual Consistency. Limited joins. | Social feeds. IoT logs. |

---

## 6. Scaling Considerations

### Bottlenecks
You optimize the Web Tier (add 100 servers).
Now the **Database** is the bottleneck.
You add Read Replicas.
Now the **Load Balancer** is the bottleneck.
You add DNS Load Balancing.
*System Design is the game of moving the bottleneck.*

### Cost Implications
*   **Vertical**: Exponential cost curve (64GB RAM is 4x the price of 32GB).
*   **Horizontal**: Linear cost curve (2 servers cost exactly 2x of 1 server).

---

## 7. Failure Scenarios & Recovery

### 1. Region Outage
*   **Scenario**: AWS `us-east-1` goes down entirely (Database, Servers, everything).
*   **Mitigation**: **Multi-Region Active-Passive**.
    *   Have a standby database in `us-west-2`.
    *   On failure, update DNS to point to West.
    *   *Trade-off*: High cost (paying for idle servers) + Data replication lag.

### 2. Thundering Herd
*   **Scenario**: Cache clears. 10,000 requests hit the DB simultaneously. DB crashes.
*   **Mitigation**:
    *   **Circuit Breaker**: Fail fast before DB dies.
    *   **Request Coalescing**: Combine identical requests into one.

---

## 8. Security Considerations

### 1. Attack Surface
*   More servers = More targets.
*   Internal communication (Service A -> Service B) implies a trusted network. If an attacker breaches the perimeter, they can move laterally.

### 2. Secure Architecture
*   **Zero Trust**: Assume the internal network is hostile. Use mTLS between services.
*   **Rate Limiting**: Protect APIs from DDoS and brute force.

---

## 9. Performance Considerations

*   **Throughput**: Requests per second (RPS). Optimized by Batching and Async processing.
*   **Latency**: Time to first byte. Optimized by Caching and Edge Computing (CDN).
*   *Trade-off*: Batching improves Throughput but worsens Latency.

---

## 10. Real Production Lessons

### The "Unlimited" Query Mistake
*   **Mistake**: `SELECT * FROM Orders WHERE userId = ?` (No limit, No pagination).
*   **Outcome**: It works for 1 year. One day, a VIP user with 50,000 orders logs in. The query tries to load 50k rows into RAM. OutOfMemoryError. Application crashes.
*   **Lesson**: Always use `LIMIT` and Pagination.

---

## 11. Interview Questions

### Basic
1.  What is the difference between Vertical and Horizontal scaling?
2.  What is a Load Balancer?
3.  Why do we need a Cache?
4.  What is Latency vs Throughput?
5.  What is ACID?

### Intermediate
1.  How do you handle session state in a horizontal scaling setup?
2.  What is the difference between SQL and NoSQL scaling?
3.  Explain master-slave replication.
4.  What is a Reverse Proxy?
5.  How do you prevent a Single Point of Failure (SPOF)?

### Advanced
1.  How would you design a system to handle 1 Million writes per second?
2.  Explain the Thundering Herd problem and how to fix it.
3.  What are the trade-offs of using Microservices vs Monolith for scaling?
4.  How does Sharding affect ACId transactions?
5.  How do you handle "Hot Partitions" in a sharded database?

### Architect-Level
1.  Design a multi-region Active-Active architecture with strict consistency requirements. (Hint: It's nearly impossible due to CAP).
2.  Evaluate the cost-benefit analysis of moving from On-Premises to Cloud for a stable workload.
3.  How would you architect a legacy migration for a banking core without downtime?
4.  Explain the impact of False Sharing in CPU caches on High-Frequency Trading systems.
5.  Design a disaster recovery plan with RPO=0 and RTO<5mins.

---

## 12. Scenario-Based System Design Problems

### 1. Scale for Black Friday
*   **Scenario**: Your e-commerce site expects 50x traffic in 2 days.
*   **Plan**:
    *   Pre-warm Load Balancers.
    *   Increase Auto-Scaling Group max limits.
    *   Switch to aggressively caching static content (CDN).
    *   Degrade non-essential features (Recommendations, Review generation).

### 2. Recover from Data Center Fire
*   **Scenario**: Primary DC is gone.
*   **Plan**:
    *   DNS Failover to Secondary DC.
    *   Promote Read Replica to Master.
    *   Restore missing data from async replication logs (Potential data loss if async).

---

## 13. Summary & Architect Takeaways

1.  **It Depends**: The answer to every system design question. The skill is explaining *what* it depends on.
2.  **No Silver Bullet**: Every decision is a trade-off. Consistency costs Latency. Availability costs Consistency.
3.  **Keep It Simple**: Complex distributed systems fail in complex ways. A Monolith on a mainframe is sometimes the right answer.
