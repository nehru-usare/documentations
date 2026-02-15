# Latency vs Throughput

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Performance Tuning Core

---

## 0. Learning Objectives
*   **Beginner**: Don't confuse "Fast" (Low Latency) with "Handling a lot" (High Throughput).
*   **Developer**: Memorize the "Latency Numbers Every Programmer Should Know".
*   **Architect**: Apply Little's Law to capacity planning.

---

## 1. Problem Context
**Why does this exist?**
"Make the system fast."
Does that mean:
A) Return the homepage in 10ms? (Latency)
B) Handle 1 million users at once? (Throughput)
Often, improving one hurts the other.
*   **Example**: To handle 1M users (Throughput), you queue requests. This increases wait time (Latency).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Latency (The Delay)
*   **Definition**: Time taken for a single request to complete.
*   **Unit**: Milliseconds (ms).
*   **Goal**: Minimize it.
*   **Analogy**: Driving a Ferrari (Fast individual speed).

### 2. Throughput (The Volume)
*   **Definition**: Number of requests processed per unit of time.
*   **Unit**: Requests Per Second (RPS) or Queries Per Second (QPS).
*   **Goal**: Maximize it.
*   **Analogy**: Driving a Bus (Slow individual speed, but moves 50 people at once).

### 3. Bandwidth
*   **Definition**: Maximum data transfer rate of the network.
*   **Unit**: Mbps / Gbps.
*   **Analogy**: The width of the highway. (Wider highway = More Throughput, but speed limit (Latency) is same).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Latency Numbers Everyone Should Know

| Action | Time (approx) |
| :--- | :--- |
| **L1 Cache Reference** | 0.5 ns |
| **L2 Cache Reference** | 7 ns |
| **Main Memory (RAM)** | 100 ns |
| **Read 1MB from Memory** | 250,000 ns (0.25 ms) |
| **Round Trip in Datacenter** | 500,000 ns (0.5 ms) |
| **Disk Seek** | 10,000,000 ns (10 ms) |
| **Read 1MB from Network** | 10,000,000 ns (10 ms) |
| **Read 1MB from Disk** | 20,000,000 ns (20 ms) |
| **Packet CA -> Netherlands** | 150,000,000 ns (150 ms) |

**Takeaway**:
*   Memory is 1000x faster than Disk.
*   Disk is 10x slower than Network (within DC).
*   Cross-region is the slowest reliability killer.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Little's Law
$$ L = \lambda \times W $$
*   $L$ = Average number of items in the system (Concurrency).
*   $\lambda$ = Average arrival rate (Throughput).
*   $W$ = Average wait time (Latency).

**Implication**: If your Latency ($W$) goes up (Database slows down), and Traffic ($\lambda$) stays same, Concurrency ($L$) MUST go up.
*   **Result**: Your server runs out of threads/RAM and crashes.
*   *Fix*: Improve Latency OR Limit Throughput (Backpressure).

### 2. The TCP Handshake
*   Every HTTP connection starts with a TCP 3-way handshake.
*   **Latency Impact**: 1 Round Trip Time (RTT) added before data is sent.
*   **TLS Impact**: Adds 2 more RTTs.
*   *Optimization*: Keep-Alive connections (Reuse TCP).

---

## 5. Trade-Off Analysis

### Optimizing for Latency vs Throughput

| Strategy | Effect on Latency | Effect on Throughput | Example |
| :--- | :--- | :--- | :--- |
| **Batching** | 🔴 Worsens (Wait for batch to fill) | 🟢 Improves (Less I/O overhead) | Kafka Producer |
| **Compression** | 🔴 Worsens (CPU time to compress) | 🟢 Improves (Less data on wire) | Gzip on Nginx |
| **Caching** | 🟢 Improves (RAM access) | 🟢 Improves (Offload DB) | Redis |
| **Parallelism** | 🟢 Improves | 🔴 May limit (Context switching) | Multi-threading |

---

## 6. Scaling Considerations

### The "Tail Latency" Problem (p99)
*   User A: 10ms.
*   User B: 10ms.
*   User C (p99): 5000ms (Hit a GC pause or cold disk).
*   **Average Latency**: 1676ms. (Misleading).
*   **Architect Rule**: Always optimize for **p95** or **p99**, never Average. Average hides the bodies.

---

## 7. Failure Scenarios & Recovery

### 1. Throughput Saturation
*   **Scenario**: 1Gbps network card is handling 1Gbps.
*   **Event**: Packet Loss increases dramatically.
*   **Result**: Retries occur. Throughput effectively **drops** because bandwidth is wasted on retries.
*   **Mitigation**: Rate Limiting. Shed load before saturation.

---

## 8. Security Considerations

### 1. Encryption Costs
*   AES-256 is fast (Hardware support).
*   RSA (Asymmetric) is slow.
*   **Impact**: TLS Handshake (RSA) hurts Latency. Data transfer (AES) affects Throughput minimally.

---

## 9. Performance Considerations

### Tuning Linux for Throughput
*   Increase File Descriptors (`ulimit -n`).
*   Increase TCP Buffer Limits (`net.ipv4.tcp_rmem`).

### Tuning for Latency
*   Disable Nagle's Algorithm (`TCP_NODELAY`).
*   Use Kernel Bypass networking (DPDK) for extreme cases (HFT).

---

## 10. Real Production Lessons

### The "N+1" Query Disaster
*   **Scenario**: Fetch User + Fetch 10 Posts (10 queries).
*   **Throughput**: DB hit with 11 queries per request.
*   **Latency**: 11 sequential Network RTTs.
*   **Fix**: `JOIN` or `WHERE IN (...)`. 1 Query.
    *   Latency: 1 RTT.
    *   Throughput: DB CPU usage drops 10x.

---

## 11. Interview Questions

### Basic
1.  Define Latency and Throughput.
2.  Which is faster: Reading from Disk or Network?
3.  What is p99 latency?
4.  How does Compression affect Latency?
5.  What is Bandwidth?

### Intermediate
1.  Explain Little's Law in simple terms.
2.  Why do we monitor p99 instead of Average?
3.  How does a Load Balancer improve Throughput?
4.  Does adding threads always improve Throughput? (Context switching).
5.  What is the latency cost of a TLS handshake?

### Advanced
1.  How do you optimize a system for low latency (e.g., Gaming)?
2.  How do you optimize a system for high throughput (e.g., File Upload)?
3.  Compare UDP vs TCP regarding latency.
4.  Explain "Head-of-Line Blocking" in HTTP/1.1 vs HTTP/2.
5.  Calculate the theoretical max throughput of a 1Gbps link with 1KB packets.

### Architect-Level
1.  Design a metrics ingestion service targeting 1M EPS (Events Per Second). Optimize for Throughput.
2.  Your p99 latency is spiking every 60 seconds. Diagnose. (Hint: Java GC, Cron jobs).
3.  Architect a global video conferencing tool. (UDP, Edge locations, Latency focus).
4.  Evaluate the trade-off of using a Service Mesh (Sidecar) on latency.
5.  Explain Coordinated Omission in load testing.

---

## 12. Scenario-Based System Design Problems

### 1. Design a Log Ingestion System
*   **Goal**: High Throughput.
*   **Latency**: Can be high (logs can appear 5 mins later).
*   **Choice**: Batch logs on client. Compress. Send to Kafka.

### 2. Design a Real-Time Bidding (AdTech)
*   **Goal**: Ultra-Low Latency (<100ms hard limit).
*   **Throughput**: Extremely High.
*   **Choice**: In-memory everything. No Disk I/O outcome path. Geographically distributed servers.

---

## 13. Summary & Architect Takeaways

1.  **Context is King**: "Is it fast?" is a bad question. "Is it Low Latency?" or "Is it High Throughput?" are good questions.
2.  **Buffers**: The universal tool to trade Latency for Throughput.
3.  **Distance Kills**: You cannot beat the speed of light. Move data closer to the user (CDN).
