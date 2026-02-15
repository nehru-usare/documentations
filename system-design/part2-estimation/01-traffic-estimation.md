# Traffic Estimation (QPS)

> **Part 2: Estimation**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Art of "Back of the Envelope" Math

---

## 0. Learning Objectives
*   **Beginner**: Convert "1 Million Users" into "Requests Per Second".
*   **Developer**: Calculate Read:Write ratios to select the right database.
*   **Architect**: Estimate Peak QPS to size Load Balancers and Auto-Scaling Groups.

---

## 1. Problem Context
**Why does this exist?**
In an interview (or real life), you get: "Design Twitter."
You CANNOT design anything until you know the **Volume**.
*   10 QPS? -> Single Server (SQLite).
*   100k QPS? -> 1000 Server Cluster (Cassandra + Redis).
Traffic Estimation is the compass that points you to the right architecture.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. DAU / MAU
*   **DAU**: Daily Active Users. (e.g., WhatsApp: 500M).
*   **MAU**: Monthly Active Users. (e.g., Facebook: 3B).
*   **Stickiness**: DAU / MAU. (High = Addictive app).

### 2. QPS / RPS
*   **QPS**: Queries Per Second (Database term).
*   **RPS**: Requests Per Second (Web Server term).
*   *Note*: 1 RPS (User click) might cause 10 QPS (DB calls).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Magic Formula

$$ \text{DAU} \times \text{Requests/User} = \text{Daily Requests} $$
$$ \frac{\text{Daily Requests}}{86,400 \text{ seconds}} = \text{Average QPS} $$

**Example**:
*   **DAU**: 1 Million.
*   **Requests/User**: 10 (View 10 pages/day).
*   **Daily Requests**: 10 Million.
*   **Seconds in Day**: $\approx 100,000$ (Rounding down from 86,400 for mental math).
*   **Average QPS**: $10,000,000 / 100,000 = 100 \text{ QPS}$.

---

## 4. Internal Mechanics (🔴 Architect Level)

### Peak Traffic Multiplier
Average QPS is useless for capacity planning. The internet sleeps at night and wakes up at 8 PM.
*   **Standard Multiplier**: $2\times$ Average.
*   **Flash Sale / SuperBowl**: $10\times$ to $100\times$ Average.
*   *Architect Rule*: Always provision for **Peak**, not Average.

### Read:Write Ratio
*   **Social Media**: 100:1 (Read Heavy).
    *   *Design*: Cache Aggressively. Read Replicas.
*   **IoT Sensor**: 1:100 (Write Heavy).
    *   *Design*: Metric Store (TimeScaleDB), Cassandra. No Caching.

---

## 5. Trade-Off Analysis

| Strategy | Accuracy | Speed | Use Case |
| :--- | :--- | :--- | :--- |
| **Back of Envelope** | Low ($\pm 50\%$) | Instant | Interviews, Initial Scoping. |
| **Log Analysis** | High | Slow | Capacity Planning for existing systems. |
| **Load Testing** | Medium | Slow | Preparing for Black Friday. |

---

## 6. Scaling Considerations

### The "Million User" Mental Model
*   **100 QPS**: 1 Laptop.
*   **1k QPS**: 1 Big Server + 1 DB.
*   **10k QPS**: Load Balancer + 5 Servers + DB Master/Slave.
*   **100k QPS**: Sharded DB + Distributed Cache + CDN.
*   **1M QPS**: Custom Infrastructure (Google/Meta level).

---

## 7. Failure Scenarios & Recovery

### 1. Underestimation
*   **Scenario**: You estimated 1k QPS. actual was 10k QPS.
*   **Event**: Load Balancer saturates. Connection pools exhausted.
*   **Recovery**: **Shed Load** (Return 503 to 90% of users) to save the system for the 10%.

---

## 8. Security Considerations

### 1. DDoS Impact on Estimation
*   Your estimate covers *Legitimate* traffic.
*   DDoS can add 1M QPS instantly.
*   **Design**: Ensure your Autoscaler has a **Max Limit** (Budget Cap) so DDoS doesn't bankrupt you.

---

## 9. Performance Considerations

*   **QPS vs Latency**: As QPS approaches system capacity, Latency increases exponentially (Hockey Stick curve).
*   **Safe Zone**: Run servers at 50-70% CPU usage. Never aim for 100%.

---

## 10. Real Production Lessons

### The Pokémon GO Launch
*   **Estimate**: 5x baseline.
*   **Reality**: 50x baseline.
*   **Outcome**: Global outage for days.
*   **Lesson**: Cloud elasticity is not instant. Quota limits exist.

---

## 11. Interview Questions

### Basic
1.  How many seconds are in a day? (86,400. Remember 10^5).
2.  What is QPS?
3.  If you have 10M DAU, calculate RPS.
4.  What is a typical Read:Write ratio for Twitter?
5.  Why do we estimate for Peak likely?

### Intermediate
1.  How do you estimate QPS for a new startup with 0 users?
2.  What is the QPS limit of a standard PostgreSQL instance? (~2k-5k writes/sec).
3.  How does QPS affect Connection Pooling?
4.  Why is "Average QPS" dangerous?
5.  How many servers do you need for 50k QPS? (Assume 1k QPS per tomcat).

### Advanced
1.  Estimate the QPS of Google Search.
2.  Develop a formula for QPS including API consumption (Mobile App + Web + API partners).
3.  How does WebSocket traffic differ from HTTP QPS calculations?
4.  Explain Little's Law relation to QPS.
5.  How do you capacity plan for a "World Cup Final" streaming event?

### Architect-Level
1.  Your CFO asks: "Why do we pay for 100 servers when average load uses 20?" Answer him.
2.  Derive the QPS for the "Like Service" on Instagram.
    *   (500M DAU * 20 likes/user) / 100k sec = 100k writes/sec.
3.  Design a Load Test to validate your QPS estimation.

---

## 12. Scenario-Based System Design Problems

### 1. Design a URL Shortener (TinyURL)
*   **Traffic**: 100M URLs generated per month.
*   **Write QPS**: $100M / (30 \times 24 \times 3600) \approx 40 \text{ writes/sec}$.
*   **Read QPS (100:1)**: $4,000 \text{ reads/sec}$.
*   **Conclusion**: Very low load. 1 Redis + 1 DB is enough.

### 2. Design a Chat App
*   **Traffic**: 10M DAU. 100 messages/day.
*   **QPS**: $(10M \times 100) / 100k = 10,000 \text{ Msg/sec}$.
*   **Peak**: $20,000 \text{ Msg/sec}$.
*   **Conclusion**: Needs Sharding or Cassandra.

---

## 13. Summary & Architect Takeaways

1.  **Memorize the numbers**: Day = 86,400s. Month = 2.5M sec.
2.  **Estimate High**: Hardware is cheap. Engineering time to fix an outage is expensive.
3.  **Validate**: Back of envelope is a guess. Load Test is proof.
