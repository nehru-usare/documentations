# Bandwidth Estimation

> **Part 2: Estimation**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Monthly Bill Killer

---

## 0. Learning Objectives
*   **Beginner**: Distinguish between Mbps (Speed) and MBps (Storage transfer).
*   **Developer**: Calculate egress costs for AWS/Cloud.
*   **Architect**: Design network topology based on bandwidth constraints (Backbone vs Public Internet).

---

## 1. Problem Context
**Why does this exist?**
"Design Netflix."
*   User watches a 4GB movie.
*   10 Million users watch movies tonight.
*   Total Data: 40,000 TB in few hours.
*   Can the internet cables handle this? (Spoiler: No, unless you use Edge Caching).
Bandwidth bottlenecks are harder to fix than CPU bottlenecks because you can't just "download more RAM" for the fiber optic cables under the ocean.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Bits vs Bytes
*   **Storage** is measured in **Bytes** (B).
*   **Network** is measured in **Bits** (b).
*   **8 bits = 1 Byte**.
*   **1 Gbps** (Gigabit per sec) = **125 MB/s** (Megabytes per sec).
*   *Interview Trap*: Don't forget to divide by 8!

### 2. Ingress vs Egress
*   **Ingress**: Data coming IN (Uploads). Usually Free on Cloud.
*   **Egress**: Data going OUT (Downloads). Expensive ($0.09/GB on AWS).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Formula

$$ \text{Bandwidth} = \text{QPS} \times \text{Avg Request Size} $$

**Example: Twitter Feed**
*   **QPS**: 100,000 Reads/sec.
*   **Size**: 10KB per JSON feed.
*   **Throughput**: $100,000 \times 10 \text{ KB} = 1,000,000 \text{ KB/s} = 1 \text{ GB/s}$.
*   **Network required**: $1 \text{ GB/s} \times 8 = 8 \text{ Gbps}$.
    *   *Result*: A standard 10Gbps Network Card can handle this (barely).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Packet Overhead
Sending 1 Byte of data requires ~64 Bytes of Headers (Ethernet + IP + TCP).
*   **Small Packets** (Chat apps): Very inefficient. 90% bandwidth is headers.
*   **Jumbo Frames**: Enable MTU 9000 to improve efficiency for internal massive transfers.

### 2. Uplink Saturation
*   If you saturate the Uplink (100% usage), Latency spikes from 10ms to 2000ms (Bufferbloat).
*   *Rule*: Plan for Max 70% utilization.

---

## 5. Trade-Off Analysis

| Strategy | Cost | Complexity | Use Case |
| :--- | :--- | :--- | :--- |
| **Public Internet** | High (Egress) | Low | Standard APIs. |
| **Cloud Backbone (VPC)** | Low | Medium | Service-to-Service comms. |
| **Direct Connect** | Fixed (High setup) | High | On-Prem to Cloud Hybrid. |
| **CDN (Edge)** | Medium | Low | Serving Static Assets (Images/Video). |

---

## 6. Scaling Considerations

### Horizontal Scaling for Network
*   One server has a 10Gbps card.
*   You need 100Gbps.
*   **Solution**: 10 Servers + Load Balancer.
*   **Warning**: The Load Balancer itself needs a 100Gbps card (Hardware LB or DNS Balancing).

---

## 7. Failure Scenarios & Recovery

### 1. "Slashed Cable"
*   **Scenario**: Construction worker cuts a fiber cable.
*   **Event**: Bandwidth drops by 50% instantly. Packets drop.
*   **Recovery**: BGP Rerouting (Automatic, takes minutes). Performance degrades.

---

## 8. Security Considerations

### 1. DDoS Volumetric Attack
*   **Attack**: 500 Gbps of junk UDP traffic.
*   **Impact**: Your 10Gbps pipe is clogged. Legitimate users are blocked.
*   **Defense**: Cloudflare / AWS Shield. They define "Bandwidth" as 50 Tbps (Global Anycast). They absorb the ocean; you drink from a straw.

---

## 9. Performance Considerations

### Data Locality
*   Moving 1TB inside same Availability Zone: Free & Fast (100Gbps).
*   Moving 1TB across Regions (US -> EU): Expensive & Slow.
*   *Design*: Keep Compute close to Data.

---

## 10. Real Production Lessons

### The "Cost" of 4K Video
*   **Scenario**: Startup decides to support 4K video uploads.
*   **Math**: 1hr 4K = 20GB.
*   **Bandwidth**: User streams it. Egress Cost = $0.09 * 20 = $1.80 per view.
*   **Business**: Ad revenue per view = $0.01.
*   **Outcome**: Bankruptcy.
*   **Lesson**: Bandwidth costs must align with business models.

---

## 11. Interview Questions

### Basic
1.  Convert 1Gbps to MB/s.
2.  What is Egress vs Ingress?
3.  Why is video streaming a bandwidth challenge?
4.  Why do we compress data (GZIP)?
5.  What is CDN?

### Intermediate
1.  Estimate bandwidth for a YouTube Clone.
2.  How does Protocol Overhead affect bandwidth? (WebSocket vs HTTP).
3.  Calculcate the Egress cost for 1PB data transfer on AWS.
4.  What is "Store and Forward" vs "Streaming"?
5.  Explain "Bitrate" in video.

### Advanced
1.  Design a topology for a high-frequency trading platform (Kernel Bypass, Solarflare).
2.  How does "Anycast" IP help with bandwidth distribution?
3.  Analyze the bandwidth savings of Peer-to-Peer (BitTorrent) architecture for updates.
4.  Explain TCP incast in a data center.
5.  Optimize the "Chat Protocol" to minimize bandwidth (Protobuf vs JSON).

### Architect-Level
1.  Negotiate a peering agreement with an ISP to reduce bandwidth costs.
2.  Design a multi-CDN strategy to handle terrestrial fiber cuts.
3.  Evaluate the ROI of running your own fiber dark/lit vs using Cloud.

---

## 12. Scenario-Based System Design Problems

### 1. Design Zoom
*   **Traffic**: Video + Audio. Real-time.
*   **Bandwidth**: 2 Mbps per user.
*   **100 Users**: 200 Mbps.
*   **Server**: Sending 200 Mbps to EACH user? 20 Gbps?
*   **Optimization**: SFU (Selective Forwarding Unit). Determine active speaker, prioritize their stream, downgrade others.

### 2. Design Netflix CDN
*   **Goal**: Reduce ISP Backbone usage.
*   **Solution**: **Open Connect**. Put hard drives physically inside the ISP's data center.
*   **Bandwidth**: User -> ISP Local Rack (Free).

---

## 13. Summary & Architect Takeaways

1.  **Divide by 8**: Never confuse Bits and Bytes.
2.  **CDN is Mandatory**: For any public-facing media app, CDN is not optional.
3.  **Cost Awareness**: Bandwidth is the most variable bill in the cloud. Watch it like a hawk.
