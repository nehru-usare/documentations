# 03. API Security (Rate Limiting, Throttling, Input Validation)

> **Part 6: Security**  
> **Difficulty:** ⭐⭐⭐⭐ (Security Engineer)  
> **Status:** Critical

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand OWASP Top 10. |
| **Developer** | Use `@Valid` and Sanitization libraries. |
| **Architect** | Implement Global Rate Limiting at Network Edge. |

---

## 1. Why This Topic Exists

### The Open Door
APIs are public. Robots scan them 24/7.
If you don't validate input, you get **SQL Injection**.
If you don't limit rates, you get **DDoS**.

---

## 3. Core Concepts (🟢 Beginner Level)

### Rate Limiting vs Throttling
*   **Rate Limiting**: "You can make 100 requests per minute." (Rejects 101st).
*   **Throttling**: "You can make 100 req/min." (Queues the 101st to process later).

### CORS (No, it's not a bug)
It's a feature. Browsers block `domain-a.com` from calling `domain-b.com` unless `domain-b` explicitly allows it.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Input Validation
Never trust the client.
*   **Java Bean Validation**:
    ```java
    data class UserDto(
        @field:Email val email: String,
        @field:Size(min=8) val password: String
    )
    ```
*   **Sanitization**: Use libraries like **JSoup** to strip `<script>` tags from user input preventing XSS.

---

## 14. Summary & Architect Takeaways

*   **Defense in Depth**: Validate at Gateway, Validate at Controller, Validate at Database.
*   **Fail Safe**: Default response should be "403 Forbidden".
