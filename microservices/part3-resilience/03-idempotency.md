# 03. Idempotency & Failure Recovery

> **Part 3: Resilience & Fault Tolerance**  
> **Difficulty:** ⭐⭐⭐⭐ (Architect)  
> **Status:** Mandatory

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand what Idempotency means. |
| **Developer** | Implement Idempotent Consumers using Redis/DB. |
| **Architect** | Design reliable APIs that survive network retries. |

---

## 1. Why This Topic Exists

### The Retry Problem
Network calls fail. Clients retry.
*   **Request 1**: "Pay $100". (Server processes it, Response lost).
*   **Request 2 (Retry)**: "Pay $100". (Server processes it again).
*   **Result**: User charged $200.

### The Solution: Idempotency
An operation is idempotent if performing it multiple times yields the same result as performing it once.

---

## 2. Big Picture Architecture View

```mermaid
graph LR
    Client -->|1. POST /pay (Key: 123)| Gateway
    Gateway -->|2. Check Key 123| Redis[(Idempotency Store)]
    
    Redis -->|Found| Gateway
    Gateway -->|3. Return Cached Response| Client
    
    Redis -->|Not Found| Service
    Service -->|4. Process Payment| DB
    Service -->|5. Save Result (Key 123)| Redis
```

---

## 3. Core Concepts (🟢 Beginner Level)

### Safe Methods
`GET`, `HEAD`. Don't modify state. Inherently idempotent.

### Idempotent Methods
`PUT`, `DELETE`.
*   `DELETE /users/1`: If called twice, user is still deleted. (Result might be 200 then 404, but server state is same).

### Non-Idempotent Methods
`POST`. Usually creates a new resource. Needs **Idempotency Key**.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Implementing Idempotency Key
1.  **Client** generates UUID: `Idempotency-Key: a1b2-c3d4`.
2.  **Server** checks Database: `SELECT * FROM processed_requests WHERE key = 'a1b2-c3d4'`.
3.  **If exists**: Return saved 200 OK.
4.  **If new**: Process, save key + response, return 200 OK.

### Kafka Consumer Deduplication
Kafka guarantees "At Least Once". You WILL get duplicates.
*   **Pattern**: Use the Message ID as the Idempotency Key.
    ```sql
    INSERT INTO payments (id, amount) VALUES (?, !)
    ON CONFLICT (id) DO NOTHING;
    ```

---

## 9. Architect-Level Best Practices

1.  **Keys must expire**: Idempotency keys should live for ~24 hours. Don't fill DB forever.
2.  **Scope**: Keys should be scoped to the User.
3.  **Concurrency lock**: If two requests with same Key arrive at same millisecond, use `SELECT FOR UPDATE` or Redis Lock to ensure one waits.

---

## 12. Interview Questions

1.  Is `POST` idempotent? (No).
2.  How do you handle duplicate messages in Kafka?
3.  Designing an API for a Payment Gateway, how do you prevent double charging?

---

## 14. Summary & Architect Takeaways

*   **Assume Retries**: Your API clients *will* retry. Your Kafka consumers *will* see duplicates.
*   **Unique Keys**: The only way to deduplicate is if the sender identifies the distinct "intent" with a unique ID.
