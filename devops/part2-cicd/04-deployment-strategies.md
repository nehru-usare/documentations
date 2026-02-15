# Deployment Strategies

> **Part 2: CI/CD Pipelines**  
> **Difficulty:** ⭐⭐⭐ (Architect)  
> **Status:** Release the Kraken (Safely)

---

## 0. Learning Objectives
*   **Beginner**: Why "Maintenance Windows" are obsolete.
*   **Developer**: Using Feature Flags to separate Deploy from Release.
*   **Architect**: Designing Zero-Downtime deployment architectures (Blue/Green, Canary).

---

## 1. Context
Deployment is dangerous. It's when things break.
*   **Goal**: Minimize "Blast Radius" (Impact of failure).

---

## 2. Strategies (Overview)

| Strategy | Cost | Risk | Complexity |
| :--- | :--- | :--- | :--- |
| **Big Bang** | Low | **High** | Low |
| **Rolling** | Low | Medium | Low (K8s Default) |
| **Blue/Green**| **High** (Double Inf) | Low | Medium |
| **Canary** | Low | **Lowest** | **High** |

---

## 3. Architecture Breakdown (🟡 Developer Level)

### 1. Rolling Deployment (K8s Default)
*   Pods: `v1, v1, v1`
*   Start `v2`. Wait for Ready.
*   Kill `v1`.
*   Result: `v2, v1, v1` -> `v2, v2, v1` -> `v2, v2, v2`.
*   *Pros*: No extra cost.
*   *Cons*: Rollback is slow. API compatibility issues (v1 and v2 run simultaneously).

### 2. Blue/Green
*   **Blue**: Serving 100% Traffic (v1).
*   **Green**: Idle / Staging (v2).
*   **Deploy**: Deploy v2 to Green. Test Green.
*   **Switch**: Update Load Balancer to point to Green.
*   *Pros*: Instant Rollback (Switch LB back).
*   *Cons*: Double infrastructure cost.

### 3. Canary Release
*   Deploy v2 to a small subset of users (1%).
*   Monitor Metrics (Errors, Latency).
*   If Healthy -> Increase to 10% -> 50% -> 100%.
*   If Unhealthy -> Rollback.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Feature Toggles (Process)
*   **Deploy != Release**.
*   **Deploy**: Moving code to servers (Invisible to users).
*   **Release**: Enabling the feature flag (Visible to users).
*   *Mechanics*: Code contains `if (flags.isEnabled("new-ui"))`.
*   Allows "Dark Launching".

### 2. Database Migrations
*   **Challenge**: Code v2 needs DB Schema v2. But v1 is still running.
*   **Pattern: Expand-Contract**:
    1.  **Expand**: Add new column `phone_v2`. (v1 ignores it).
    2.  **Migrate**: Copy data `phone` -> `phone_v2`.
    3.  **Deploy Code**: v2 uses `phone_v2`.
    4.  **Contract**: Delete old `phone` column.

---

## 5. Scaling Considerations

### Session Stickiness
*   If utilizing Canary, ensure a user sticks to v1 or v2.
*   Use Consistent Hashing or Cookies.
*   Mixing v1/v2 UI assets causes broken pages.

---

## 6. Real Production Lessons

### Amazon
*   **Apollo Deployment Engine**:
    *   Deploys to 1 box in 1 AZ (Availability Zone).
    *   Then 1 AZ.
    *   Then 1 Region.
    *   Then Global.
    *   Stops automatically if metrics degrade.

---

## 7. Interview Questions

### Basic
1.  What is Zero Downtime deployment?
2.  Difference between Deploy and Release.
3.  Explain Rolling Update.

### Intermediate
1.  How does Blue/Green differ from Canary?
2.  What is a Feature Flag?
3.  How to handle DB migrations in Rolling Update? (Backward compatibility).

### Advanced
1.  Design a Canary pipeline using Istio/Service Mesh (Traffic splitting).
2.  Architect a "Dark Launch" strategy for a major UI overhaul.
3.  Handle "Schema Drift" in a Blue/Green SQL environment.

---

## 8. Summary & Architect Takeaways

1.  **Decouple**: Separate Deploy from Release (Feature Flags).
2.  **Safety**: Canary is the gold standard for large scale.
3.  **Cost**: Blue/Green is safe but expensive. Canary is complex but cheap.
