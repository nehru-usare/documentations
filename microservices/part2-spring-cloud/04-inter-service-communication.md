# 04. Inter-Service Communication (Feign, WebClient)

> **Part 2: Spring Cloud Core**  
> **Difficulty:** ⭐⭐⭐ (Developer)  
> **Status:** Standard Practice

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Call another microservice without hardcoding the URL. |
| **Developer** | Use OpenFeign to create declarative REST clients. |
| **Architect** | Choose between Blocking (Feign) and Non-Blocking (WebClient). |

---

## 1. Why This Topic Exists

### The Problem
Using `RestTemplate` (Legacy) is verbose.
```java
restTemplate.getForObject("http://192.168.1.5:8080/users/" + id, User.class);
```
1.  IP is hardcoded (Bad).
2.  Boilerplate code (Bad).
3.  No automated Load Balancing.

### The Solution
**OpenFeign**: A declarative web service client. You write an Interface, Spring generates the implementation at runtime.

---

## 2. Big Picture Architecture View

```mermaid
graph LR
    OrderService -->|1. Call Interface| FeignProxy
    FeignProxy -->|2. Resolve 'user-service'| LoadBalancer
    LoadBalancer -->|3. Choose IP (Eureka)| TargetIP
    
    TargetIP -->|4. HTTP Request| UserService
```

---

## 3. Core Concepts (🟢 Beginner Level)

### Declarative Client
You define *what* call you want to make, not *how* to make it.
Similar to Spring Data JPA Repository interfaces.

### Client-Side Load Balancing
The client decides which instance to call (Round Robin), avoiding a central bottleneck.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### 1. Spring Cloud OpenFeign
Enable it: `@EnableFeignClients`.

**The Interface**:
```java
@FeignClient(name = "user-service", fallback = UserFallback.class)
public interface UserClient {
    
    @GetMapping("/users/{id}")
    UserDTO getUser(@PathVariable Long id);
}
```

**The Usage**:
```java
@Service
public class OrderService {
    @Autowired UserClient userClient;
    
    public void process() {
        UserDTO user = userClient.getUser(123L); // Logic abstracted
    }
}
```

### 2. Spring WebClient (The Future)
For Reactive stacks (WebFlux), Feign is blocking (mostly). Use WebClient.
```java
webClient.get()
    .uri("lb://user-service/users/{id}", id)
    .retrieve()
    .bodyToMono(UserDTO.class);
```

---

## 5. Internal Mechanics (🔴 Architect Level)

### Feign & Threads
Feign uses standard HTTP libraries (Apache HttpClient / OKHttp) under the hood. It is **Blocking**.
The thread calls `userClient.getUser()` and waits.
*   **Implication**: If `User Service` is slow, `Order Service` threads pile up.
*   **Fix**: Set read-timeouts and connection-timeouts strict.

### Error Decoding
When `User Service` returns `404`, Feign throws `FeignException`.
You can implement `ErrorDecoder` to translate this into a domain exception (`UserNotFoundException`).

---

## 6. Production & Failure Scenarios

### Scenario: The Domino Effect
*   **Event**: User Service takes 30s to respond.
*   **Impact**: Order Service threads block for 30s. Tomcat pool fills up. Order Service stops accepting reqs.
*   **Fix**: **Circuit Breaker** integration.
    *   `spring.cloud.openfeign.circuitbreaker.enabled=true`.
    *   If fallback triggers, return a "Guest User" or default value.

---

## 9. Architect-Level Best Practices

1.  **Shared DTO Library?**: **Controversial**.
    *   *Option A*: Share a `.jar` with `UserDTO`. (Coupling ↑, Safety ↑).
    *   *Option B*: Duplicate `UserDTO` in Order Service. (Coupling ↓, Maintenance ↑).
    *   *Recommendation*: Duplicate for Microservices. Share for Moduliths.
2.  **Timeouts**: Default Feign timeout is often too long (or short). Tune it per client.

---

## 12. Interview Questions

### Basic
1.  What is Feign?
2.  How does Feign know the IP address? (Combines with Eureka/LoadBalancer).

### Intermediate
1.  What is the fallback mechanism in Feign?
2.  Difference between `RestTemplate` and `WebClient`?

### Advanced
1.  How do you handle error propagation? (If Service B throws 400, should Service A throw 400 or 500?).
2.  Why is Client-Side load balancing better than Server-Side for internal traffic?

---

## 14. Summary & Architect Takeaways

*   **Abstraction**: Feign makes HTTP calls look like Java method calls. Dangerous if you forget it's a network call (Fallacy #1).
*   **Resilience**: Always assume the call will fail. Implement Fallbacks.
