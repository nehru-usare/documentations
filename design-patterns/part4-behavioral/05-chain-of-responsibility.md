# Chain of Responsibility

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐⭐ (Intermediate)  
> **Status:** Critical for Security

---

## 0. Learning Objectives

*   **Beginner**: Understand how "Middleware" works.
*   **Developer**: Implement a Validation Chain.
*   **Architect**: Design Security Filters (Auth -> Role -> Feature).

---

## 1. Problem Statement

### The "Massive If-Else" Problem
You have a request processing system.
```java
if (auth.failed()) return error;
else if (validation.failed()) return error;
else if (cache.miss()) loadFromDb();
else log();
```
*   **Issue**: Adding a new step (e.g., Rate Limiting) requires modifying the core logic. Tightly coupled.

### The Solution
Chain the objects: `RateLimitHandler` -> `AuthHandler` -> `ValidationHandler`.
Each handler decides: "Do I handle this? Or do I pass it to the next guy?"

---

## 2. Real-World Analogy

**Tech Support**
1.  **Level 1 (Chatbot)**: Can I solve "Reset Password"? Yes. Done.
2.  **Level 2 (Human)**: Can I solve "Server Crash"? No. Pass to L3.
3.  **Level 3 (Engineer)**: Solves it.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Abstract Handler
```java
public abstract class Handler {
    protected Handler next;

    public void setNext(Handler next) {
        this.next = next;
    }

    public void handleRequest(String request) {
        if (next != null) {
            next.handleRequest(request);
        }
    }
}
```

### 2. Concrete Handlers
```java
class AuthHandler extends Handler {
    @Override
    public void handleRequest(String request) {
        if (request.equals("ADMIN")) {
            System.out.println("Auth Passed.");
            super.handleRequest(request); // Pass to next
        } else {
            System.out.println("Auth Failed. Chain stops.");
        }
    }
}

class LogHandler extends Handler {
    @Override
    public void handleRequest(String request) {
        System.out.println("Logging: " + request);
        super.handleRequest(request);
    }
}
```

### 3. Client
```java
Handler h1 = new AuthHandler();
Handler h2 = new LogHandler();

h1.setNext(h2); // Build chain: Auth -> Log

h1.handleRequest("ADMIN"); // Passes Auth, then Logs.
h1.handleRequest("GUEST"); // Fails Auth. Log never called.
```

---

## 6. Spring Boot Implementation

The entire **Spring Security** framework is one giant Chain of Responsibility.
*   `SecurityFilterChain`: A list of Filters.
    *   `CorsFilter` -> `CsrfFilter` -> `UsernamePasswordAuthenticationFilter` -> `AuthorizationFilter`.
*   If ANY filter throws an Exception (401), the chain stops.

---

## 8. Advantages

1.  **Dynamic**: You can add/remove handlers at runtime (e.g., enable "Maintenance Mode" filter).
2.  **SRP**: Classes focus on ONE job (Auth, Logging, Compression).

---

## 9. Disadvantages

1.  **Uncertainty**: The request might fall off the end of the chain without being handled.
2.  **Debugging**: Stack traces become very deep and repetitive.

---

## 14. Interview Questions

### Basic
1.  **Example in Java API?** (`try-catch` blocks! The exception is passed up the chain of catch blocks. Also `ServletFilter`).
2.  **Does a handler always have to pass to the next?** (No. It can block/drop the request).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: ATM Dispenser ($100, $50, $20 notes).
    *   *Design*: $100Handler tries to dispense. Passes remainder to $50Handler.

2.  **Scenario**: Input Sanitization.
    *   *Design*: `TrimHandler` -> `XSSHandler` -> `SQLInjectionHandler`.

---

## 16. Summary & Architect Takeaways

*   **Middleware**: This is the pattern for cross-cutting concerns (Logging, Auth, Metrics) in Web Servers.
