# 05. Chaos Engineering & Testing

> **Part 3: Resilience & Fault Tolerance**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (SRE)  
> **Status:** Advanced

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand that "Hope is not a strategy". |
| **Developer** | Use `Toxiproxy` to simulate broken networks locally. |
| **Architect** | Plan and execute "Game Days" to verify system recovery. |

---

## 1. Why This Topic Exists

### The Hypothesis
"I configured the Circuit Breaker. It *should* work."
**Chaos Engineering** challenges this. We purposefully break the system to **prove** it works.
> "Chaos Engineering is the discipline of experimenting on a system in order to build confidence in the system’s capability to withstand turbulent conditions in production."

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Simulating Failures with Toxiproxy
Before Prod, test locally.
`Toxiproxy` is a TCP proxy that lets you inject faults via API.
*   **Latency**: Add 5000ms delay to MySQL connection.
*   **Bandwidth**: Limit to 1KB/s.
*   **Reset**: Cut TCP (RST packet).

---

## 5. Internal Mechanics (🔴 Architect Level)

### Blast Radius
Start small.
1.  **Local**: Dev laptop.
2.  **Staging**: QA environment.
3.  **Canary**: 1% of Prod users.
4.  **Prod**: The entire region (Netflix Chaos Kong).

---

## 14. Summary & Architect Takeaways

*   **Don't panic**: Chaos Engineering is managed, monitored, and stoppable.
*   **Verify Resilience**: A fallback meant for safety is useless if it has a bug. Test it.
