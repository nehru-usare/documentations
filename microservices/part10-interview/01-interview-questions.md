# 01. 100+ Microservices Interview Questions

> **Part 10: Interview & Scenario Kit**  
> **Difficulty:** ⭐ to ⭐⭐⭐⭐⭐  
> **Status:** The Gauntlet

---

## 🟢 Level 1: Beginner / Junior Developer

1.  **What is a Microservice?**
    *   *Answer*: Small, independent service that does one thing well. Loosely coupled, independently deployable.
2.  **Monolith vs Microservices: Name 3 differences.**
    *   *Answer*: Single codebase vs Multiple repos. Shared DB vs Database per Service. Function calls vs Network calls.
3.  **What is the CAP Theorem?**
    *   *Answer*: You can only pick 2: Consistency, Availability, Partition Tolerance. In distributed systems, P is mandatory, so choice is CP or AP.
4.  **What is a REST API?**
    *   *Answer*: Representational State Transfer. Uses HTTP verbs (GET, POST, PUT, DELETE) and resources.
5.  **Why use Docker?**
    *   *Answer*: "It works on my machine" -> "It works everywhere". Containerization.

---

## 🟡 Level 2: Senior Developer / Lead

21. **Explain the Saga Pattern.**
    *   *Answer*: A sequence of local transactions. If one fails, execute compensating transactions to undo previous steps. Used because 2PC is too slow.
22. **What is the difference between Orchestration and Choreography?**
    *   *Answer*: Orchestration = Central Conductor (God Service). Choreography = Event-driven (Dancers knowing their steps).
23. **How do you handle Distributed Transactions?**
    *   *Answer*: Sagas, Two-Phase Commit (Avoid if possible), or Eventual Consistency.
24. **What is a Circuit Breaker?**
    *   *Answer*: A proxy that fails fast if the downstream service is slow/down, preventing cascading failures.
25. **How do you prevent Idempotency issues in Kafka?**
    *   *Answer*: Consumer stores `MessageID` in DB. If `MessageID` exists, ignore the message.

---

## 🔴 Level 3: Architect / Principal

51. **Design a Rate Limiter for 100M req/sec.**
    *   *Answer*: Token Bucket Algorithm. Use Redis (with Lua scripts) or a Sidecar (Envoy).
52. **How to migrate a Legacy Monolith to Microservices without downtime?**
    *   *Answer*: Strangler Fig Pattern.
53. **Database per Service: How to do Joins?**
    *   *Answer*: Application-Side Joins (Fetch ID from A, then fetch details from B) or Data Replication (CQRS).
54. **Explain the difference between Blue/Green and Canary Deployment.**
    *   *Answer*: Blue/Green = Switch 100% traffic at once. Canary = Gradual rollout (1% -> 10% -> 100%).
55. **How to secure Microservices?**
    *   *Answer*: mTLS for internal communication. OAuth2/OIDC for user auth. Network Policies.

---

## 🔥 Level 4: The "Impossible" Scenarios

91. **"The system is slow." How do you debug it?**
    *   *Answer*: Check Metrics (CPU/RAM). Check Tracing (Where is time spent?). Check DB Locks. Check Network Latency.
92. **Redis is down. What happens to the system?**
    *   *Answer*: Design for failure. Fallback to DB (slower) or return degraded response (empty list). System should NOT crash.
