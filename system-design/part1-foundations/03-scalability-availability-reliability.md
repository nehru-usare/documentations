# Scalability, Availability, Reliability

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Holy Trinity of System Design

---

## 0. Learning Objectives
*   **Beginner**: Understand that "Scalable" does not mean "Reliable".
*   **Developer**: Learn the math behind "Three Nines" (99.9%) vs "Five Nines" (99.999%).
*   **Architect**: Design systems that are Reliable even when components are Unreliable (The Google Philosophy).

---

## 1. Problem Context
**Why does this exist?**
You can build a Scalable system (handles 1B users) that crashes every hour (Low Reliability).
You can build a Reliable system (never crashes) that can only handle 1 user (Low Scalability).
In Distributed Systems, these three concepts are entangled. You must balance them.

**Real-World Motivation**:
*   **AWS S3**: Promises 99.999999999% (11 nines) Durability but only 99.99% Availability.
*   **Twitter Fail Whale**: High Scalability (users grew fast) but Low Availability (site crashed often).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Scalability
*   **Definition**: The ability of a system to handle increased load by adding resources.
*   **Analogy**: A restaurant adding more tables to serve more customers.

### 2. Availability (Uptime)
*   **Definition**: The percentage of time the system is operational and accessible.
*   **Analogy**: The restaurant is "Open" 24/7.

### 3. Reliability (Correctness)
*   **Definition**: The probability that the system performs correctly without failure for a specific time.
*   **Analogy**: The restaurant doesn't poison the food.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The "Nines" of Availability

| Nines | Availability | Downtime / Year | Downtime / Day | Example |
| :--- | :--- | :--- | :--- | :--- |
| **99%** | 2 Nines | 3.65 days | 14.4 mins | MVP / Internal Tool |
| **99.9%** | 3 Nines | 8.76 hours | 1.44 mins | Standard Website |
| **99.99%** | 4 Nines | 52.6 mins | 8.64 secs | Enterprise SaaS |
| **99.999%** | 5 Nines | 5.26 mins | 0.86 secs | Telco / Banking |

**Note**: To get from 3 Nines to 4 Nines, cost usually increases **10x**.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Measuring Reliability
*   **MTBF (Mean Time Between Failures)**: Average time system works. (Target: High)
*   **MTTR (Mean Time To Recovery)**: Average time to fix a failure. (Target: Low)
*   **Availability Formula**:
    $$ Availability = \frac{MTBF}{MTBF + MTTR} $$
    *   *Architect Insight*: It is often easier to reduce MTTR (automate recovery) than to increase MTBF (make code perfect).

### 2. The Probability of Distributed Failure
If a server has 99.9% availability, and you need a chain of 3 servers (A -> B -> C) to handle a request:
*   Total Availability = $0.999 \times 0.999 \times 0.999 = 99.7\%$
*   **Insight**: Adding dependencies **reduces** availability.

---

## 5. Trade-Off Analysis

| Goal | Strategy | Trade-Off |
| :--- | :--- | :--- |
| **High Scalability** | Sharding/Partitioning. | Complex Joins. Possible Hotspots. |
| **High Availability** | Redundancy (Active-Active). | Data Consistency issues (Split Brain). 2x Cost. |
| **High Reliability** | Transactions (ACID), Checksums. | Lower Throughput. Higher Latency. |

---

## 6. Scaling Considerations

### Scalability Types
*   **Load Scalability**: Handling more RPS. (Solution: Stateless App Servers).
*   **Data Scalability**: Handling more TBs. (Solution: Sharding).
*   **Geographic Scalability**: Reducing latency for global users. (Solution: Edge/CDN).

---

## 7. Failure Scenarios & Recovery

### 1. The "Single Point of Failure" (SPOF)
*   **Scenario**: Your Load Balancer is redundant, Web Servers are redundant, but you have 1 Database Master.
*   **Event**: Master HDD fails.
*   **Impact**: System Unavailable.
*   **Mitigation**: **Automated Failover** (Orchestrator promotes Slave to Master).

### 2. Cascading Failure
*   **Scenario**: Service A slows down. Service B retries aggressively.
*   **Impact**: Service A is DDoSed by Service B. System collapses.
*   **Mitigation**: **Exponential Backoff** and **Circuit Breakers**.

---

## 8. Security Considerations

### Reliability vs Security
*   **DDoS Attack**: An attack on Availability.
*   **Ransomware**: An attack on Reliability (Data integrity).
*   **Defense**:
    *   **Rate Limiting**: Protect Availability.
    *   **Immutable Backups**: Protect Reliability.

---

## 9. Performance Considerations

*   **Reliability Overhead**: Writing data to disk (fsync) is reliable but slow. Writing to memory is fast but unreliable.
*   **Availability Overhead**: Replicating data to 3 nodes takes network bandwidth and latency.

---

## 10. Real Production Lessons

### The S3 Outage (2017)
*   **Event**: An engineer tried to remove a few servers. A typo removed **too many** servers.
*   **Impact**: Half the internet broke (Reliability failure due to Human Error).
*   **Lesson**: Build tools that prevent humans from doing catastrophic things (Rate limit operational commands).

---

## 11. Interview Questions

### Basic
1.  What is the difference between Availability and Reliability?
2.  What does "3 Nines" mean?
3.  Give an example of a Single Point of Failure.
4.  How does Redundancy improve Availability?
5.  What is Vertical Scaling?

### Intermediate
1.  Calculate the availability of two 99% components in Series vs Parallel.
2.  What is MTTR and why is it critical?
3.  How does Microservices architecture affect overall system availability?
4.  Explain "Graceful Degradation".
5.  How do you scale a Relational Database?

### Advanced
1.  Design a system with 99.999% availability. What are the cost implications?
2.  Explain the "Split Brain" problem in Active-Active clusters.
3.  How does Sharding improve Scalability but hurt Reliability?
4.  Derive Little's law in the context of Scalability.
5.  Critique the reliability of Eventual Consistency.

### Architect-Level
1.  "We want 100% reliability." How do you explain to a CEO that this is impossible/unprofitable?
2.  Design a control plane vs data plane separation to ensure availability during upgrades.
3.  Analyze the reliability of a Quorum-based write (N=3, W=2, R=2).
4.  How do you implement Disaster Recovery with RPO < 1 min across regions?
5.  Debug a scenario where high Scalability led to low Reliability (e.g. Gossip protocol storm).

---

## 12. Scenario-Based System Design Problems

### 1. Design a Stock Exchange
*   **Priority**: High Reliability (Zero data loss). High Availability.
*   **Trade-off**: Scalability (Can be lower, number of stocks is finite).

### 2. Design a Social Media Feed
*   **Priority**: High Scalability (Billions of posts). High Availability.
*   **Trade-off**: Reliability (It's okay if a post is lost or shown out of order).

---

## 13. Summary & Architect Takeaways

1.  **Failures are Normal**: Design systems that expect failure, not systems that try to avoid it.
2.  **Redundancy is Key**: 2 mediocre servers are more available than 1 super-server.
3.  **MTTR > MTBF**: You can't prevent failure (MTBF), but you can fix it fast (MTTR).
