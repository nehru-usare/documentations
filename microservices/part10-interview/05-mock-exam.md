# 05. Validating your Knowledge (Mock Exam)

> **Part 10: Interview & Scenario Kit**  
> **Difficulty:** ⭐⭐⭐  
> **Status:** Graduation

---

## 🎓 The Exam

### Section 1: Fundamentals (20 Points)

1.  **True or False**: Microservices share a single database to ensure consistency.
    *   [ ] True
    *   [ ] False (Correct)

2.  **Which theorem states you can't have Consistency, Availability, and Partition Tolerance simultaneously?**
    *   [ ] ACID
    *   [ ] BASE
    *   [ ] CAP (Correct)
    *   [ ] SOLID

### Section 2: Patterns (20 Points)

3.  **Which pattern prevents a slow microservice from crashing the caller?**
    *   [ ] Saga
    *   [ ] Circuit Breaker (Correct)
    *   [ ] CQRS
    *   [ ] Log Aggregation

4.  **In Saga pattern, what do you call the transaction that undoes a committed change?**
    *   [ ] Rollback
    *   [ ] Compensating Transaction (Correct)
    *   [ ] Delete
    *   [ ] Void

### Section 3: Architecture (20 Points)

5.  **You need to ensure an Order is processed exactly once, even if the server crashes. Which pattern helps?**
    *   [ ] Idempotency Key (Correct)
    *   [ ] Retry
    *   [ ] Load Balancing
    *   [ ] Caching

6.  **Which component is responsible for authenticating external traffic before it hits microservices?**
    *   [ ] Service Registry
    *   [ ] Config Server
    *   [ ] API Gateway (Correct)
    *   [ ] Distributed Trace

### Section 4: Scenario (40 Points)

**Scenario**: You are designing a Ticket Booking System (like Ticketmaster).
A high-demand concert goes on sale. 1 Million users try to buy 10,000 tickets at 10:00 AM.
**Question**: How do you prevent overselling?

*   **Bad Answer**: "Use `synchronized` in Java." (Fails on multiple servers).
*   **Okay Answer**: "Database Row Locking `SELECT FOR UPDATE`." (DB acts as bottleneck).
*   **Good Answer**: "Redis Atomic Decrement (`DECR`) or a Queue-based approach where only 10,000 requests are allowed into the 'Payment Area'."

---

## 🏆 Graduation

If you can answer 80% of these questions confidently:
**Congratulations! You are ready to lead a Microservices Architecture.**
