# Count-Min Sketch

> **Part 6: Patterns**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** The Approximate Counter

---

## 0. Learning Objectives
*   **Beginner**: counting "Pokemon" mentions in a Twitter stream without storing every tweet.
*   **Developer**: Implementing the Sketch data structure (2D Array).
*   **Architect**: Detecting DDoS attacks in real-time at the Edge using constant memory.

---

## 1. Problem Context
**Why does this exist?**
Stream: 1 Million IPs per second.
Question: "Which IP appeared most frequently?" (Top-K).
*   **HashMap**: `Map<IP, Long>`. Memory explodes.
*   **Count-Min Sketch**: Uses fixed memory (e.g., 2KB) to count frequency with small error.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. The Matrix
*   A 2D Array of counters. `width (w)` x `depth (d)`.
*   All initialized to 0.

### 2. The Logic
*   **Add(x)**:
    *   Hash1(x) -> col 3. `Increment Matrix[1][3]`.
    *   Hash2(x) -> col 8. `Increment Matrix[2][8]`.
*   **Query(x)**:
    *   Get values at `[1][3]` and `[2][8]`.
    *   Return the **Minimum** of these values.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Why Minimum?
*   Counters are shared (Collisions).
*   `Matrix[1][3]` might count "X" AND "Y". So the value is `Count(X) + Count(Y)`.
*   The value is always **Overestimated**. Never Underestimated.
*   By taking the *Minimum* of $k$ hash functions, we reduce the noise from collisions.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Error Bounds
*   **$\epsilon$ (Epsilon)**: Acceptable error factor (e.g., count is off by 1% of total stream). Width $w = 2/\epsilon$.
*   **$\delta$ (Delta)**: Probability that estimate is bad. Depth $d = \ln(1/\delta)$.
*   *Power*: You can count trillions of events with a few MBs of RAM.

### 2. Heavy Hitters
*   CM Sketch is excellent for finding "Heavy Hitters" (Items appearing > 0.1% of time).
*   Noise (Rare items) is drowned out. Heavy items dominate their buckets.

---

## 5. Trade-Off Analysis

| Strategy | Space | Precision | Order Preserved |
| :--- | :--- | :--- | :--- |
| **Counters (Map)** | O(Unique Items) | Exact | Yes |
| **Sampling** | Low | Low (Misses rare items) | No |
| **Count-Min Sketch** | O(1) [Fixed] | Approximate (Over-counts) | No |

---

## 6. Scaling Considerations

### Mergeability
*   CM Sketches can be merged.
*   Sketch(Total) = Sketch(Server A) + Sketch(Server B).
*   Element-wise addition of the matrices.
*   *Benefit*: Ideal for MapReduce / Distributed Aggregation.

---

## 7. Failure Scenarios & Recovery

### 1. Counter Overflow
*   If counting 4 Billion events, a 32-bit Integer rolls over.
*   *Fix*: Use 64-bit Longs or "Morris Counter" (Probabilistic counter).

---

## 8. Security Considerations

### 1. Hash DoS
*   If attacker knows hash functions, they can create collisions to artificially inflate the count of an IP (Framing an innocent user).

---

## 9. Performance Considerations

*   **CPU Friendly**: Array access is fast.
*   **Cache Friendly**: Small sketches fit in L1/L2 Cache.

---

## 10. Real Production Lessons

### Apple's Privacy
*   **Use Case**: Collecting typing usage data from millions of iPhones.
*   **CM Sketch**: Used to aggregate data without tracking individual user sessions. (Differential Privacy).

---

## 11. Interview Questions

### Basic
1.  What is the Count-Min Sketch used for? (Frequency Estimation).
2.  Does it underestimate or overestimate? (Overestimate).
3.  Can you iterate over keys in a CM Sketch? (No, it's a summary).
4.  Difference between Bloom Filter and CM Sketch. (Set Membership vs Frequency).
5.  What is "Top-K"?

### Intermediate
1.  Why take the Minimum and not the Average?
2.  How to handle "Reset" or "Time Decay"? (Halving counters periodically).
3.  Estimate memory for 99% accuracy.
4.  Collision handling strategy.
5.  How to find Top-K if you can't iterate keys? (Maintain a separate Min-Heap of "Suspected Heavy Hitters" during ingestion).

### Advanced
1.  Design a distributed Top-K system for Twitter Trending Topics.
2.  Compare CM Sketch vs Count-Mean-Min Sketch.
3.  How to use CM Sketch for Range Query estimation? (Dyadic intervals).
4.  Analyze the impact of skew (Zipfian distribution) on CM Sketch accuracy.
5.  Critique "Lossy Counting" algorithm vs CM Sketch.

### Architect-Level
1.  "We need to detect 'Hot Keys' in our Sharded DB in real-time." Architect a Sidecar proxy using CM Sketch.
2.  Design a billing system for a CDN based on bandwidth usage using sketches (Is approx billing legal? Probably not. Use for alerts only).
3.  Evaluate HyperLogLog (Cardinality) vs Count-Min Sketch (Frequency).

---

## 12. Scenario-Based System Design Problems

### 1. Design Youtube View Counter (Real-time)
*   **Req**: Show "1M views" quickly.
*   **Core**: CM Sketch for real-time display.
*   **Truth**: Batch logs (MapReduce) for accurate billing/ads later. The Lambda Architecture.

### 2. Design DDoS Detector
*   **Req**: Alert if IP > 1000 req/sec.
*   **Core**: CM Sketch.
*   **Logic**: On Request, `Add(IP)`. If `Estimate(IP) > 1000`, block.

---

## 13. Summary & Architect Takeaways

1.  **Accept Error**: In Big Data, exactness is too expensive. 99.9% is mostly fine.
2.  **Streaming**: Algorithms that work in one pass are invaluable.
3.  **Additive**: The ability to sum sketches from different servers simplifies distributed monitoring.
