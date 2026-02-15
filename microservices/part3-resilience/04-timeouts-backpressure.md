# 04. Timeouts & Backpressure

> **Part 3: Resilience & Fault Tolerance**  
> **Difficulty:** ⭐⭐⭐⭐ (Architect)  
> **Status:** Production tuning

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Learn why "infinite" waits are bad. |
| **Developer** | Configure timeouts in Feign, RestTemplate, and JDBC. |
| **Architect** | Implement Backpressure strategies for high-load systems. |

---

## 1. Why This Topic Exists

### The Resource Exhaustion
Threads are expensive (1MB stack).
If 200 threads wait for a database that never answers, the server runs out of memory/threads.
**Timeouts release resources.**

### Backpressure
If `Consumer` reads 10 msg/sec but `Producer` sends 100 msg/sec, Consumer crashes (OOM).
Backpressure is the Consumer saying: "I can only handle 10 right now".

---

## 3. Core Concepts (🟢 Beginner Level)

### Connect Timeout
Time allowed to establish TCP handshake. (Should be fast, e.g., 2s).

### Read Timeout (Socket Timeout)
Time allowed waiting for data packets *after* connection. (Business logic time, e.g., 5s).

### Load Shedding
Intentionally dropping HTTP 503s to save the server when overload is detected.

---

## 5. Internal Mechanics (🔴 Architect Level)

### Reactive Backpressure (Project Reactor / RxJava)
*   **Pull Model**: Subscriber requests N items (`subscription.request(10)`).
*   **Strategies**:
    *   **Buffer**: Store excess in RAM (Risk: OOM).
    *   **Drop**: Discard latest actions.
    *   **Error**: Fail fast.

---

## 9. Architect-Level Best Practices

1.  **Defaults Suck**: Most JDBC drivers default to "No Timeout". Fix this globally.
2.  **Shorter is Better**: Fail fast. It's better to show an error in 1s than 60s.
3.  **Circuit Breaker vs Timeout**: Timeout handles *one* bad request. Circuit Breaker handles *many*. You need both.

---

## 14. Summary & Architect Takeaways

*   **Bounded Resources**: Never assume infinite memory or CPU.
*   **Flow Control**: If you can't process it, reject it. Don't queue it forever.
