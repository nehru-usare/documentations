# Circuit Breaker Pattern

> **Part 6: Enterprise & Microservices Patterns**  
> **Difficulty:** ⭐⭐ (Intermediate)  
> **Status:** The Safety Fuse

---

## 1. Problem Statement

### The "Cascading Failure" Problem
Service A calls Service B.
Service B is slow/down.
Service A's threads get stuck waiting for Service B.
Service A runs out of threads.
Service A dies.
Everything calling Service A dies.
**Result**: Total System Collapse.

### The Solution
Install a **Circuit Breaker**.
If Service B fails 50% of the time, **Open** the circuit.
Future calls to Service B fail *immediately* (Fast Fail) without waiting.
After a timeout, try again (**Half-Open**).

---

## 2. Core Concept

### States
1.  **CLOSED**: Normal operation. Requests go through.
2.  **OPEN**: Failure threshold reached. Request fails immediately.
3.  **HALF-OPEN**: Timeout passed. Allow 1 test request.
    *   Success? -> Go to CLOSED.
    *   Failure? -> Go to OPEN.

---

## 3. Spring Boot Implementation (Resilience4j)

```java
@Service
public class ProductService {

    @CircuitBreaker(name = "inventory", fallbackMethod = "fallbackInventory")
    public String getInventory(String id) {
        // Call remote service
        return restTemplate.getForObject("http://inventory-service/" + id, String.class);
    }

    // Fallback when Open or Exception
    public String fallbackInventory(String id, Throwable t) {
        return "Default Inventory: Available";
    }
}
```

### Configuration (`application.yml`)
```yaml
resilience4j.circuitbreaker:
  instances:
    inventory:
      failureRateThreshold: 50
      waitDurationInOpenState: 10000ms
```

---

## 4. Architect Takeaway
*   **Fail Fast**: It is better to return an error in 10ms than to timeout in 30s.
*   **Fallback**: Always provide a fallback (Default value, Cached value).
*   **Monitoring**: You MUST monitor the state of your breakers. (Micrometer/Grafana).
