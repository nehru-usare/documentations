# 02. Strangler Fig Pattern (Legacy Migration)

> **Part 8: Real-world Architecture Patterns**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Architect)  
> **Status:** The only way to survive a Rewrite.

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand why "Rewrite from scratch" is a suicide mission. |
| **Developer** | Configure Nginx/Gateway to route `/new-api` to Microservice and `/old-api` to Monolith. |
| **Architect** | Plan a 2-year migration roadmap. |

---

## 1. Why This Topic Exists

### The Big Bang Failure
"The old code is messy. Let's start a new repo!"
*   **Result**: 
    1.  New System takes 2 years.
    2.  Old System keeps changing (Business needs features).
    3.  New System launches with missing features. **Explosion**.

### The Solution: Strangler Fig
Named after a tree that grows around a host tree.
1.  Put a Proxy in front of Monolith.
2.  Route 99% traffic to Monolith.
3.  Route 1% (`/users`) to New Microservice.
4.  Repeat until Monolith is gone.

---

## 2. Big Picture Architecture View

```mermaid
graph TD
    User -->|HTTPS| Proxy[API Gateway / Load Balancer]
    
    Proxy -->|/users| New[User Microservice]
    Proxy -->|/orders| New2[Order Microservice]
    
    Proxy -->|/* (Everything else)| Old[Legacy Monolith]
    
    New -.->|Read| OldDB[(Legacy DB)]
```

---

## 3. Core Concepts (🟢 Beginner Level)

### The Proxy (Interceptor)
You cannot strangle without a Gateway.
If Clients call the Monolith directly (`http://legacy-app.com`), you can't intercept them.
**Step 0**: Move all traffic to `http://api.company.com` (Gateway).

### Asset Capture
When you move "Orders" to a microservice, you also need the Data.
*   **Double Write**: Write to Old DB and New DB.
*   **CDC (Change Data Capture)**: Sync Old DB to New DB in background.

---

## 9. Architect-Level Best Practices

1.  **Stop Feeding the Beast**: New features MUST go into Microservices. Don't add code to the Monolith.
2.  **Vertical Slicing**: Don't extract "The Layer" (DAO). Extract "The Feature" (Checkout).
3.  **Kill the Zombies**: Once a feature is migrated, DELETE all code in the Monolith. If you leave it "just in case", people will use it.

---

## 14. Summary & Architect Takeaways

*   **Patience**: Strangling takes years. That's okay. The system delivers value every day.
*   **Fallback**: If the new Microservice bugs out, switch the Proxy route back to Monolith instantly. Low risk.
