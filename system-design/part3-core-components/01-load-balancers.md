# Load Balancers (L4 vs L7)

> **Part 3: Core Components**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Traffic Police

---

## 0. Learning Objectives
*   **Beginner**: Understand why we need a Load Balancer (LB) to scale beyond 1 server.
*   **Developer**: Configure Nginx as a simple LB.
*   **Architect**: Choose between L4 (Performance) and L7 (Features) based on requirements.

---

## 1. Problem Context
**Why does this exist?**
You have 10 servers.
A user types `www.google.com`.
Which server answers?
*   You cannot give the user 10 IP addresses.
*   You need a single entry point that **distributes** the traffic.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Load Balancer (LB)
*   **Definition**: A device (hardware) or software that sits between the client and the server farm.
*   **Goal**: Maximize throughput, minimize response time, and avoid overloading any single server.

### 2. VIP (Virtual IP)
*   The single public IP address exposed by the LB. The backend servers have private IPs.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The OSI Model Context

1.  **L4 (Layer 4 - Transport)**: TCP/UDP.
    *   *Mechanism*: Looks at IP + Port only. Forwards packets.
    *   *Speed*: Ultra-fast.
    *   *Encryption*: Does NOT terminate SSL. (Pass-through).

2.  **L7 (Layer 7 - Application)**: HTTP/HTTPS.
    *   *Mechanism*: Looks at Header, Cookie, URL.
    *   *Speed*: Slower (Must decrypt, inspect, re-encrypt).
    *   *Encryption*: Terminates SSL.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Algorithms
*   **Round Robin**: Server 1 -> Server 2 -> Server 1. (Simple, assumes equal server power).
*   **Least Connections**: Send to server with fewest active TCP connections. (Good for long-lived sessions).
*   **Consistent Hashing**: `Hash(UserID) % N`. (Good for Caching compatibility).

### 2. Health Checks
*   **Passive**: Monitor actual traffic. If 500 errors spike, remove server.
*   **Active**: Ping `/health` endpoint every 5 seconds. If non-200, remove server.

---

## 5. Trade-Off Analysis

| Type | Pros | Cons | Use Case |
| :--- | :--- | :--- | :--- |
| **L4 LB** | Extreme Performance (Millions OPS). No Decryption cost. | No smart routing (Can't route `/api` vs `/static`). | DNS LB, TCP Relays. |
| **L7 LB** | Smart Routing. SSL Termination. WAF Integration. | CPU Intensive. Higher Latency. | Web Apps, Microservices Gates. |
| **Hardware LB** (F5) | Massive Throughput. | Expensive ($$$). | On-Premise Enterprise. |
| **Software LB** (Nginx/HAProxy) | Cheap. Flexible. | Tuned by CPU. | Cloud, K8s Ingress. |

---

## 6. Scaling Considerations

### The LB Bottleneck
*   The LB itself is a server. It can crash.
*   **Scaling the LB**:
    1.  **Active-Passive**: Two LBs. Use Heartbeat (Keepalived). If Primary dies, standby takes VIP.
    2.  **DNS Round Robin**: Map `www.google.com` to Multiple LB VIPs.

---

## 7. Failure Scenarios & Recovery

### 1. The "Black Hole" Server
*   **Scenario**: Server A hangs (TCP accepted, but no HTTP response).
*   **LB Logic**: "TCP Connected! Send more traffic."
*   **Result**: User requests time out.
*   **Fix**: **Deep Health Check** (Application Logic Check).

---

## 8. Security Considerations

### 1. SSL Offloading (Termination)
*   Decrypting HTTPS is CPU intensive.
*   Offloading this to L7 LB saves the backend web servers' CPU.
*   **Risk**: Traffic between LB and Web Server is unencrypted (HTTP). Secure the VPC!

### 2. DDoS Shield
*   LBs can drop SYN floods (L4 attack) before they hit your app.

---

## 9. Performance Considerations

*   **TCP Buffering**: L7 LB buffers the full request before sending to backend. This protects backend from slowloris attacks but adds latency.
*   **Keep-Alive**: LB maintains persistent connections to backend servers to avoid TCP handshake overhead.

---

## 10. Real Production Lessons

### The "Sticky Session" Trap
*   **Scenario**: App stores Session in Server RAM. L7 LB configured for Sticky Session (Cookie).
*   **Event**: Autoscaler removes Server A.
*   **Result**: All users on Server A are logged out.
*   **Fix**: Stateless App + Redis Session Store. Disable Stickiness.

---

## 11. Interview Questions

### Basic
1.  What is a Load Balancer?
2.  Name 3 LB algorithms.
3.  Difference between L4 and L7 balancing.
4.  What is a Health Check?
5.  What happens if the LB goes down?

### Intermediate
1.  Why is Round Robin bad for servers with different specs?
2.  Explain SSL Termination at the LB level.
3.  How does Consistent Hashing help with caching?
4.  What is "Connection Draining"?
5.  Can DNS act as a Load Balancer?

### Advanced
1.  Design a Load Balancer from scratch using a Hash Map.
2.  Analyze the latency impact of L7 inspection.
3.  How do you handle WebSocket connections in a Load Balancer? (Timeout issues).
4.  Explain "Direct Server Return" (DSR) to bypass LB for responses.
5.  Compare Nginx vs HAProxy vs Envoy.

### Architect-Level
1.  "We have 100M concurrent socket connections." Architect the LB layer. (L4/eBPF/XDP).
2.  Design a Global Server Load Balancing (GSLB) strategy using Anycast.
3.  Critique the use of sticky sessions in a Microservices architecture.

---

## 12. Scenario-Based System Design Problems

### 1. Design a Payment Gateway LB
*   **Require**: Zero dropped requests.
*   **Choice**: L4 LB (HAProxy) in Active-Passive mode. Least Connections algorithm.

### 2. Design a Multi-Tenant SaaS LB
*   **Require**: Routing `customer1.saas.com` to Cluster A.
*   **Choice**: L7 LB (Nginx). Inspect Host header.

---

## 13. Summary & Architect Takeaways

1.  **L4 for Speed, L7 for Intelligence**: Know the difference.
2.  **The LB is a SPOF**: Always pair them (Active-Passive).
3.  **Trust no Health Check**: A "200 OK" doesn't mean the DB is working. Test deep.
