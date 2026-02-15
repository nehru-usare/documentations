# Strangler Pattern (Strangler Fig)

> **Part 6: Enterprise & Microservices Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Legacy Killer

---

## 1. Problem Statement

### The "Big Bang" Migration
You have a 10-year-old Monolith "God App".
Management wants Microservices.
**Bad Approach**: "Let's rewrite everything from scratch in 2 years."
*   **Result**: The project will fail. The business will change requirements in 2 years.

### The Solution: Strangler Fig
Plant a new vine (Microservice) around the host tree (Monolith).
Initially, the connection (Gateway) routes 99% of traffic to Monolith.
Slowly, you route more traffic to the Vine.
Eventually, the Host Tree dies (is decommissioned).

---

## 2. Implementation Strategy

### Step 1: Insert a Gateway (Facade)
Put an API Gateway (Nginx/Spring Cloud Gateway) in front of the Monolith.
*   `/*` -> Monolith.

### Step 2: Extract One Module
Rewrite "User Profile" as a Microservice.
Update Gateway:
*   `/users/**` -> New Microservice.
*   `/*` -> Monolith.

### Step 3: Glue Code
The Monolith might still need User Data.
*   **Option A**: Monolith calls New Microservice.
*   **Option B**: Double-Write (Write to both DBs) during transition.

---

## 3. Architect Takeaway
*   **Risk Mitigation**: You can rollback `/users` to Monolith in 1 second by changing the Gateway config.
*   **Continuous Value**: You deliver value *now*, not in 2 years.
*   **Don't Rewrite**: If a module works and never changes, **don't strangle it**. Let it live in the Monolith forever.
