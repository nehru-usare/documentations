# SRE vs DevOps

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Concept)  
> **Status:** Class SRE implements Interface DevOps

---

## 0. Learning Objectives
*   **Beginner**: Difference between SRE and DevOps.
*   **Developer**: What is an Error Budget?
*   **Architect**: Defining SLOs for your services.

---

## 1. The Definition
> "SRE is what happens when you ask a software engineer to design an operations team." — *Ben Treynor Sloss, Google VP*

*   **DevOps**: The cultural philosophy (Abstract).
*   **SRE**: A prescriptive implementation of DevOps (Concrete).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. The Acronyms (SLI, SLO, SLA)
*   **SLI (Indicator)**: The measure. "Latency is 200ms".
*   **SLO (Objective)**: The goal. "Latency should be < 300ms 99% of time".
*   **SLA (Agreement)**: The contract. "If Latency is > 300ms, we pay you money".
*   **Rule**: SLA < SLO < SLI (Safety margin).

### 2. The Error Budget
*   **Assume** 100% uptime is impossible and expensive.
*   **Goal**: 99.9% uptime.
*   **Budget**: 0.1% downtime is *allowed* (43 mins / month).
*   **Usage**: You can use this budget to release risky features.
*   **If Budget Empty**: Freeze features. Focus on stability.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Automation
*   SREs hate manual work.
*   **Rule**: 50% Ops work (Tickets), 50% Coding time (Automation).
*   If Ops work > 50%, the excess tickets act as "Overflow" back to Devs.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Choosing Good SLIs
*   **VALET**:
    *   **V**olume (Traffic).
    *   **A**vailability (Error Rate).
    *   **L**atency (Response Time).
    *   **E**rrors.
    *   **T**ickets.
*   Don't measure CPU usage (User doesn't care). Measure Latency (User cares).

### 2. Burn Rate Alerting
*   Don't alert on "Error rate > 1%".
*   Alert on "Error Budget Burning too fast".
*   *Complex math*: "Alert if we burn 2% of our 30-day budget in 1 hour".

---

## 5. Trade-Off Analysis

| Role | Focus | Toolset |
| :--- | :--- | :--- |
| **SysAdmin** | Running Server | Bash, SSH |
| **DevOps** | Pipelines & Culture | Jenkins, Docker |
| **SRE** | Reliability & Code | Go/Python, Prometheus, Terraform |

---

## 6. Scaling Considerations

### Toil Limits
*   SRE teams cap operational load.
*   If a service is too unstable (Burn rate too high), SREs "Hand back the pager" to the Dev team.
*   This incentivizes Devs to fix the code.

---

## 7. Failure Scenarios & Recovery

### 1. Blameless Culture (Again)
*   SRE relies heavily on Post-Mortems (Incident Reports).
*   Must be thorough and systematic.

---

## 8. Security Considerations

### 1. Availability vs Integrity
*   SRE prioritizes Availability.
*   Security prioritizes Confidentiality/Integrity.
*   Conflict: "Shut down the service to stop the breach" (Security decision) vs "Keep it running" (SRE decision).

---

## 9. Performance Considerations

*   **Latency SLOs**:
    *   P99 Latency (The slowest 1% of requests) is what SREs optimize.
    *   Average Latency is useless (hides outliers).

---

## 10. Real Production Lessons

### Google
*   Invented SRE.
*   Has strict error budget policies. If you blow the budget, no launches for you.

---

## 11. Interview Questions

### Basic
1.  Difference between SLI and SLO.
2.  What is an Error Budget?
3.  Why 100% reliability is bad.

### Intermediate
1.  Calculate the Error Budget for 99.95% Availability.
2.  How to choose a "Golden Signal"?
3.  Explain "Handing back the pager".

### Advanced
1.  Design an Alerting strategy based on Burn Rate.
2.  How to implement SRE in a small startup? (Don't hire SREs yet, Devs do SRE).
3.  Critique "Five Nines" (99.999%). (Cost vs Value).

---

## 12. Summary & Architect Takeaways

1.  **Hope is not a strategy**: SRE brings engineering discipline to operations.
2.  **Budget your risk**: Use Error Budgets to make data-driven decisions on when to deploy.
3.  **Code the Infrastructure**: SREs are software engineers.
