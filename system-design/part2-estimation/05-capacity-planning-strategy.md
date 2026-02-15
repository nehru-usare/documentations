# Capacity Planning Strategy

> **Part 2: Estimation**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The CTO's Handbook

---

## 0. Learning Objectives
*   **Beginner**: Understand why we test systems before launch.
*   **Developer**: Learn to write Load Tests (Gatling/JMeter).
*   **Architect**: Master the strategy of "Plan, Test, Monitor, Scale".

---

## 1. Problem Context
**Why does this exist?**
Estimation is Theory.
Capacity Planning is Practice.
You estimated 100 servers.
*   Did you test it?
*   Do you have the budget?
*   How do you configure the triggers to add Server #101?

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Load Testing
*   Simulating users to break the system in a controlled environment.
*   **Stress Test**: Find the breaking point.
*   **Soak Test**: Run for 24 hours to find memory leaks.

### 2. Auto-Scaling
*   Automatically adding/removing servers based on CPU/RAM/Request metrics.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The AKF Scale Cube

1.  **X-Axis (Horizontal)**: Cloning. Run multiple copies of the app. (Load Balancing).
2.  **Y-Axis (Functional)**: Split by Service. (Microservices).
3.  **Z-Axis (Data)**: Split by Data Partition. (Sharding).

**Strategy**:
*   Start with X.
*   Move to Z (Sharding) if Data is big.
*   Move to Y (Microservices) if Team/Code is complex.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. The Auto-Scaling Lag
*   **Event**: CPU hits 80%.
*   **Action**: Scaler orders new VM.
*   **Boot Time**: 3 minutes.
*   **Result**: During these 3 minutes, the existing servers might crash (hit 100%).
*   **Fix**: Predictive Scaling (Scale up at 7:55 PM for 8:00 PM traffic) or Over-provisioning.

### 2. Bottleneck Shift
*   You fix CPU by adding servers.
*   Now the DB connection pool is exhausted.
*   You fix Pool by adding Read Replicas.
*   Now the Network NAT Gateway limits are hit.
*   *Capacity Planning is a game of Whack-a-Mole.*

---

## 5. Trade-Off Analysis

| Strategy | Speed to Scale | Cost | Complexity |
| :--- | :--- | :--- | :--- |
| **VMs (EC2)** | Slow (Mins) | Medium | Medium |
| **Containers (K8s)** | Fast (Secs) | Medium | High |
| **Serverless (Lambda)** | Instant (Ms) | High (at scale) | Low (Ops) |

---

## 6. Scaling Considerations

### CapEx vs OpEx
*   **CapEx (Capital Expense)**: Buying hardware upfront. (On-Prem). Cheaper in long run for stable load.
*   **OpEx (Operational Expense)**: Renting Cloud. (AWS). More flexibility.

---

## 7. Failure Scenarios & Recovery

### 1. The "Scale Down" Death
*   **Scenario**: Traffic drops. Auto-scaler kills 50% of servers.
*   **Event**: Servers were processing async jobs locally.
*   **Result**: Jobs lost.
*   **Fix**: Graceful Shutdown hooks. Drain connections before kill.

---

## 8. Security Considerations

### 1. Economic Denial of Sustainability (EDoS)
*   Attacker triggers your Auto-Scaling.
*   You scale to 1000 servers.
*   Attacker stops.
*   You get a bill for $50,000.
*   **Defense**: Budget Ceilings. Anomaly Detection.

---

## 9. Performance Considerations

*   **Warm-up**: Java/JVM needs time to JIT compile. New capacity performs poorly for first few minutes.
*   **Mitigation**: Probes (Liveness/Readiness). Don't send traffic until "Warm".

---

## 10. Real Production Lessons

### Analyzing the "Black Friday" Prep
*   **Action**: Retailers "Freeze" code updates in November.
*   **Action**: They run synthetic load tests at 200% expected volume.
*   **Result**: Boring, uneventful Black Fridays. (Success).

---

## 11. Interview Questions

### Basic
1.  What is Load Testing?
2.  What is Stress Testing?
3.  Difference between Vertical and Horizontal Scaling options.
4.  Why do we need a "Staging" environment?
5.  What is Auto-Scaling?

### Intermediate
1.  How do you determine the "Breaking Point" of a system?
2.  Explain the AKF Scale Cube.
3.  What metrics should trigger an Auto-Scale event? (CPU? Latency? Queue Depth?).
4.  How do you handle "State" during scaling?
5.  Pros/Cons of Serverless for capacity planning.

### Advanced
1.  Design a Load Test for a WebSocket Chat app.
2.  How do you capacity plan for a legacy Monolith that cannot be horizontally scaled?
3.  Analyze the cost of "Over-provisioning" vs "Lost Revenue due to Downtime".
4.  Explain "Coordinated Omission" in Load Testing tools.
5.  Develop a Predictive Scaling algorithm based on historical data.

### Architect-Level
1.  Create a "Capacity Model" spreadsheet for a CFO.
2.  Design a hybrid-cloud strategy for "Bursting" capacity.
3.  Post-Mortem: "We auto-scaled, but the site still went down." (Database saturation).

---

## 12. Scenario-Based System Design Problems

### 1. Capacity Plan for "Ticketmaster" (Taylor Swift Concert)
*   **Traffic**: 0 to 10 Million in 1 second.
*   **Strategy**: Auto-scaling is too slow.
*   **Solution**: **Pre-provisioning**. Boot 10,000 servers 1 hour before. Implement a Queue (Waiting Room).

---

## 13. Summary & Architect Takeaways

1.  **Hope is not a Strategy**: "I think it will hold" = It will fail.
2.  **Test at Scale**: Unit tests don't find concurrency bugs. Load tests do.
3.  **Headroom**: Always keep 30-50% buffer.
