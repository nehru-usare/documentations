# DDoS Mitigation

> **Part 8: Security**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** The Flood

---

## 0. Learning Objectives
*   **Beginner**: Why sites go down when hackers attack.
*   **Developer**: Implementing simple IP Rate Limiting.
*   **Architect**: Designing layers of defense (CDN -> WAF -> Origin) to absorb 1 Tbps attacks.

---

## 1. Problem Context
**Why does this exist?**
Server capacity: 1000 req/sec.
Attacker (Botnet): 1 Million req/sec.
**Result**: Legitimate users are denied service.
**DDoS**: Distributed Denial of Service (Many attackers, one victim).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Volumetric Attacks (L3/L4)
*   **Goal**: Saturate Bandwidth.
*   **Method**: UDP Floods, NTP Amplification.
*   **Traffic**: "Junk" packets.

### 2. Application Attacks (L7)
*   **Goal**: Exhaust CPU/RAM.
*   **Method**: HTTP GET `/search?q=very_expensive_query`.
*   **Traffic**: Looks like real user traffic.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Defense Layers

1.  **ISP / Cloud Edge (Scrubbing)**:
    *   Absorbs massive Volumetric attacks. Drops malformed packets.
2.  **CDN / Load Balancer**:
    *   Absorbs HTTP floods. Caches content.
3.  **WAF (Web Application Firewall)**:
    *   Inspects HTTP Usage. "Is this User-Agent suspicious?" "Is this SQL Injection?"
4.  **Application**:
    *   Rate Limiting. Circuit Breakers.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. SYN Flood (L4)
*   **Attack**: Bot sends `SYN` (Start Connection). Server replies `SYN-ACK` and *Allocates Memory*. Bot never sends `ACK`. Memory fills up.
*   **Defense**: **SYN Cookies**. Server doesn't allocate memory until the final ACK is received. It encodes state in the sequence number.

### 2. Amplification (L3)
*   **Attack**: Bot spoofs Victim IP. Sends 1 byte query to Open DNS Resolver. DNS sends 500 bytes response to Victim. 500x amplification.
*   **Defense**: Drop UDP traffic at firewall if strictly Web. Rate limit UDP.

---

## 5. Trade-Off Analysis

| Strategy | Cost | Protection | False Positives |
| :--- | :--- | :--- | :--- |
| **Over-provisioning** | High | Low | None |
| **Rate Limiting** | Low | Medium (L7) | Medium (NAT users) |
| **Cloud Scrubbing** | High | High (L3/L4) | Low |

---

## 6. Scaling Considerations

### Anycast Routing
*   Spreads attack traffic across global PoPs (Points of Presence).
*   Instead of 1 datacenter taking 1 Tbps, 100 datacenters take 10 Gbps each.
*   Managedable.

---

## 7. Failure Scenarios & Recovery

### 1. The "Smart" Attack
*   Attacker mimics exact browser behavior (headless Chrome).
*   Bypasses simple WAF rules.
*   **Fix**: JavaScript challenges (CAPTCHA). Behavior analysis (Mouse movements).

---

## 8. Security Considerations

### 1. Pay-per-Request Exploits
*   If you use Serverless, DDoS = Financial Ruin (Denial of Wallet).
*   **Fix**: Billing alarms and hard limits.

---

## 9. Performance Considerations

*   **WAF Latency**: Inspecting every packet takes CPU.
*   *Optimization*: Use hardware/FPGA WAFs or Edge WAFs (Cloudflare) to keep latency off your origin.

---

## 10. Real Production Lessons

### GitHub (1.3 Tbps Attack)
*   **Attack**: Memcached Amplification.
*   **Relief**: Akamai Prolexic routed traffic to scrubbing centers.
*   **Recovery**: 10 minutes.

---

## 11. Interview Questions

### Basic
1.  What does DDoS stand for?
2.  Difference between DoS and DDoS.
3.  What is a Botnet?
4.  What is a WAF?
5.  Why is UDP easier to spoof than TCP? (No handshake).

### Intermediate
1.  Explain SYN Flood.
2.  What is Amplification attack?
3.  How does Anycast help DDoS?
4.  L4 vs L7 attack difference.
5.  What is Blackholing? (Dropping all traffic to target IP - Last resort).

### Advanced
1.  Design a system to detect "Slowloris" attack (Opening connections and sending headers 1 byte/sec).
2.  Analyze the economics of DDoS (Cost to attack vs Cost to defend).
3.  How to distinguish Flash Crowd (Real users) from DDoS?
4.  Critique Blocking IPs vs CAPTCHAs.
5.  Explain "Cookie Poisoning" or "Replay Attacks" in context of DDoS.

### Architect-Level
1.  "Defense in Depth strategy for a FinTech app." (ISP -> CDN -> WAF -> App).
2.  Design a Rate Limiting algorithm specific for "Expensive Search Queries".
3.  Evaluate the risks of "BGP Hijacking" during an attack.

---

## 12. Scenario-Based System Design Problems

### 1. Design Protection for API
*   **Attacker**: Scraper hitting 1000 req/sec.
*   **Def**: Token Bucket Rate Limit per API Key. Ban IP if errors > 50%.

### 2. Design Gaming Server Protection
*   **Protocol**: UDP. Real-time.
*   **Def**: Cannot use TCP-based WAF. Need proprietary L4 filter allowing only signed game packets.

---

## 13. Summary & Architect Takeaways

1.  **You cannot win alone**: You need a Vendor (Cloudflare/AWS Shield). You don't have enough bandwidth.
2.  **Hide Origin**: Never expose Origin IP. Only allow traffic from CDN IPs.
3.  **Fail Gracefully**: If search is under attack, disable search but keep the homepage alive.
