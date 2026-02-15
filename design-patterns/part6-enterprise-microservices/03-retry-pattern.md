# Retry Pattern

> **Part 6: Enterprise & Microservices Patterns**  
> **Difficulty:** ⭐ (Beginner)  
> **Status:** The Persistence Mechanism

---

## 1. Problem Statement

### The "Network Blip" Problem
Networks are unreliable.
Sometimes a DB connection drops for 10ms.
Sometimes a packet is lost.
If you fail immediately, you annoy the user.
**Solution**: Try again.

---

## 2. Core Concept

### 1. Transient vs Permanent Errors
*   **Transient (Retry)**: Network Timeout, 503 Service Unavailable, 429 Too Many Requests (maybe).
*   **Permanent (Don't Retry)**: 400 Bad Request, 401 Unauthorized, 404 Not Found.

### 2. Backoff Strategies
*   **Fixed**: Wait 1s, 1s, 1s.
*   **Exponential**: Wait 1s, 2s, 4s, 8s. (Prevents hammering a struggling service).
*   **Jitter**: Wait 1s + random(100ms). (Prevents **Thundering Herd** problem where all clients retry at exact synchronization).

---

## 3. Spring Boot Implementation (Resilience4j)

```java
@Service
public class PaymentService {

    @Retry(name = "payment", fallbackMethod = "fallback")
    public boolean processPayment(String id) {
        // If this throws Exception, it auto-retries
        return remoteBank.charge(id);
    }

    public boolean fallback(String id, Throwable t) {
        // After 3 failed attempts
        return false; // Transaction failed
    }
}
```

### Configuration
```yaml
resilience4j.retry:
  instances:
    payment:
      maxAttempts: 3
      waitDuration: 500ms
      enableExponentialBackoff: true
      exponentialBackoffMultiplier: 2
```

---

## 4. The Critical Rule: Idempotency
**NEVER** retry a non-idempotent operation.
*   **Scenario**:
    1.  Client sends "Pay $100".
    2.  Server charges $100.
    3.  Ack is lost on network.
    4.  Client Retries "Pay $100".
    5.  Server charges $100 **AGAIN**.
*   **Fix**: Server tracks `TransactionID`. If seen before, return "Success" without charging again.

---

## 5. Architect Takeaway
*   **Timeouts > Retries**: Always set a global timeout. Retrying 10 times with 10s wait = 100s latency.
*   **Don't Retry Post-Methods**: Unless you are 1000% sure the receiver is Idempotent.
