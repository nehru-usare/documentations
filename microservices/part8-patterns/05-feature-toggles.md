# 05. Feature Toggles & Canary Releases

> **Part 8: Real-world Architecture Patterns**  
> **Difficulty:** ⭐⭐⭐⭐ (DevOps)  
> **Status:** Safety Net

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand that code can be in Prod but turned off. |
| **Developer** | Implement a Feature Flag in Java (`if (flag.isEnabled)`). |
| **Architect** | Plan a Canary Release strategy for critical updates. |

---

## 1. Why This Topic Exists

### The Fear of Friday Deployments
"If I deploy this at 5 PM on Friday, I might break production."
*   **Result**: Fear-driven development.
*   **Fix**: Deploy the code, but keep the feature **OFF**. Turn it on Monday morning.

### Deploy != Release
*   **Deploy**: Moving binary to server. (Technical).
*   **Release**: Making feature visible to users. (Business).

---

## 3. Core Concepts (🟢 Beginner Level)

### Feature Flag (Toggle)
A simple boolean.
`boolean showNewUI = unleash.isEnabled("new-ui");`

### Types of Toggles
1.  **Release Toggle**: Short-lived (days). Used for Trunk-Based Development.
2.  **Ops Toggle**: Circuit breaker override. Long-lived.
3.  **Permission Toggle**: "Beta Users".

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Java Implementation (Unleash / LaunchDarkly)
Don't use Database for flags (Performance hit). Use a specific Flag engine pushed to SDK.

```java
@RestController
public class CheckoutController {
    
    @GetMapping("/checkout")
    public String checkout() {
        if (featureManager.isActive("new-checkout-flow")) {
            return newCheckoutService.process();
        } else {
            return legacyCheckoutService.process();
        }
    }
}
```

---

## 5. Internal Mechanics (🔴 Architect Level)

### Canary Release
Traffic-based rollout.
1.  Deploy V2.
2.  Route 1% of users to V2 (Istio/Nginx).
3.  Monitor Error Rate.
4.  If Error Rate < 1%, increase to 10%, then 50%, then 100%.

---

## 14. Summary & Architect Takeaways

*   **Dark Launching**: Facebook launches features to employees 1 month before the public.
*   **Kill Switch**: If a feature has a bug, turn off the flag. Instant rollback without re-deploying.
