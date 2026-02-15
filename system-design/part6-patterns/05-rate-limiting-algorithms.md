# Rate Limiting Algorithms

> **Part 6: Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** None Shall Pass (Quickly)

---

## 0. Learning Objectives
*   **Beginner**: Why APIs return `429 Too Many Requests`.
*   **Developer**: Implement Token Bucket in code.
*   **Architect**: Design a distributed rate limiter using Redis and Lua scripts.

---

## 1. Problem Context
**Why does this exist?**
Protect resources from abuse (DDoS, Scrapers, Buggy Loops).
**Fairness**: Prevent one user from hogging all threads.
**Revenue**: API Quotas (Bronze = 100 req/min, Gold = 1000 req/min).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. The 429 Code
*   Standard HTTP Status for "Slow Down".
*   Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`.

### 2. Throttling vs Rate Limiting
*   **Rate Limiting**: Hard Cap ("Reject everything > 100").
*   **Throttling**: Slow down ("Queue requests > 100, process later").

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Algorithms

1.  **Token Bucket** (Standard):
    *   Bucket holds `N` tokens. Refills at `R` tokens/sec.
    *   Request costs 1 token. If greedy, block.
    *   *Features*: Allows **Bursts** (up to bucket size).

2.  **Leaky Bucket**:
    *   Queue with constant output rate.
    *   *Features*: **Smoothes** traffic. No bursts. Good for write-heavy DBs.

3.  **Fixed Window Counter**:
    *   "100 reqs in 12:00-12:01".
    *   *Flaw*: **Edge Case**. If user sends 100 at 12:00:59 and 100 at 12:01:01, they sent 200 in 2 seconds. Violates rate.

4.  **Sliding Window Log**:
    *   Store timestamp of every request. Count logs in last minute.
    *   *Pros*: Precise.
    *   *Cons*: High Memory.

5.  **Sliding Window Counter**:
    *   Weighted average of Previous Window + Current Window.
    *   *Pros*: Low memory + Good approximation.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Distributed Rate Limiting
*   Local Memory Limiter works for 1 server.
*   For 10 servers, you need a central store (**Redis**).
*   **Race Conditions**:
    *   Get `count`. `count++`. Set `count`. (Concurrency fail).
    *   *Fix*: **Lua Script** in Redis (Atomic execution).

### 2. Synchronization Cost
*   Calling Redis for *every* request adds latency (2ms).
*   *Optimization*: **Batching** (Sync local counters to Redis every 5 seconds). Trade-off: Precision.

---

## 5. Trade-Off Analysis

| Algo | Burst Support | Memory | Precision | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Token Bucket** | Yes | Low | Medium | Amazon API |
| **Leaky Bucket** | No | Low | Medium | Nginx |
| **Sliding Log** | Yes | High | High | Strict Quotas |
| **Sliding Window** | Yes | Low | Medium-High | General Purpose |

---

## 6. Scaling Considerations

### The "Global" Limiter
*   Limiting across Datacenters is hard.
*   Async sync using Gossip or CRDTs is preferred over Strong Consistency.

---

## 7. Failure Scenarios & Recovery

### 1. Redis Down
*   If Rate Limiter Down -> **Fail Open** (Allow traffic) vs **Fail Closed** (Reject traffic).
*   *Default*: Fail Open. Don't block users because your limiter broke.

---

## 8. Security Considerations

### 1. ID based vs IP based
*   **IP Based**: Easy to bypass (Proxy/VPN). Blocks NAT users (Coffee shop).
*   **User ID / API Key**: Best.
*   *Strategy*: IP Limit (DDoS) + User Limit (Quota).

---

## 9. Performance Considerations

*   **Look-Aside**: Don't put Limiter on the critical path?
*   Actually, Limiter MUST be on critical path.
*   Make it fast (In-memory, Sidecar).

---

## 10. Real Production Lessons

### Stripe's Rate Limiter
*   Uses **Token Bucket**.
*   Does "Cost-based" limiting. (Read = 1 token. Complex Search = 5 tokens).
*   Returns extensive headers for DX.

---

## 11. Interview Questions

### Basic
1.  What is HTTP 429?
2.  Token Bucket vs Leaky Bucket.
3.  Why use Redis for Rate Limiting?
4.  Client side throttling (Exponential Backoff).
5.  What is a Burst?

### Intermediate
1.  Explain Sliding Window Counter algorithm.
2.  How to handle Race Conditions in distributed counting? (Lua / Locks).
3.  Fail Open vs Fail Closed trade-offs.
4.  Rate Limiting vs Circuit Breaker.
5.  How do you limit "Expensive" requests differently?

### Advanced
1.  Design a Rate Limiter that scales to 1M req/sec. (Local batching).
2.  Analyze the storage cost of Sliding Window Log for 1M users.
3.  Critique the "Fixed Window" boundary problem.
4.  How to implement Rate Limiting in a Service Mesh (Istio)?
5.  Derive the formula for Sliding Window weighted count.

### Architect-Level
1.  "We have a multi-tenant system. Tenant A is spamming. Isolate them." (Bulkhead + Rate Limit).
2.  Design a Hierarchical Rate Limiter (Global -> Region -> Service -> User).
3.  Evaluate using Header-based limits vs Body-based limits (Cost of parsing).

---

## 12. Scenario-Based System Design Problems

### 1. Design Configurable API Gateway
*   **Req**: Plan A: 10/s. Plan B: 100/s.
*   **Impl**: Store Limits in Cache. Check API Key. Lua Script to increment and check.

### 2. Design Login Protection
*   **Req**: Block IP after 5 failed logins.
*   **Impl**: Fixed Window (5 per min). Specific key `login_fail:{ip}`.

---

## 13. Summary & Architect Takeaways

1.  **Protect the Core**: Limiting is defense.
2.  **Token Bucket is King**: Flexible, simple, good enough for 99% cases.
3.  **Client Responsibility**: Clients must handle 429s gracefully (Retry-After).
