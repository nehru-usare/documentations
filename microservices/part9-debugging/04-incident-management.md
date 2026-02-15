# 04. Production Incident Management (Sev1/Sev2, Post-Mortems)

> **Part 9: Debugging & Troubleshooting**  
> **Difficulty:** ⭐⭐⭐ (Manager / Lead)  
> **Status:** Critical Process

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Don't panic when PagerDuty rings. |
| **Developer** | Write a Blameless Post-Mortem. |
| **Architect** | Define Severity Levels (SLAs) for the organization. |

---

## 1. Why This Topic Exists

### The Panic
The site is down. The CEO is screaming.
*   **Bad Response**: Developers start randomly restarting servers. No one knows what's happening.
*   **Good Response**: Open a Bridge (Zoom). Appoint an Incident Commander. Follow the Runbook.

---

## 3. Core Concepts (🟢 Beginner Level)

### Severity Levels
1.  **SEV-1 (Critical)**: Revenue loss. System totally down. Wake up everyone (24/7).
2.  **SEV-2 (High)**: Major feature broken (Checkout works, but Search is slow). Wake up team.
3.  **SEV-3 (Medium)**: Minor bug. Fix during business hours.

### The War Room Roles
1.  **Incident Commander (IC)**: The Boss. Makes decisions. "Restart the DB? Yes/No".
2.  **Scribe**: Writes down everything that happens. "10:05 - Alice restarted Server 1".
3.  **SME (Subject Matter Experts)**: The devs fixing the issue.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### The Blameless Post-Mortem
After the fire is out (24h later), write a document.
*   **Rule**: You cannot blame a person ("Bob deleted the DB").
*   **Why**: If you blame Bob, Bob will hide his mistakes next time.
*   **Real Cause**: "The tool allowed Bob to delete the DB without a confirmation prompt".

### 5 Whys (RCA)
1.  **Why** did the site crash? (DB overloaded).
2.  **Why** was DB overloaded? (Too many connections).
3.  **Why**? (Marketing sent a push notification to 1M users).
4.  **Why**? (No rate limiting on the API).
5.  **Why**? (We forgot to configure Nginx). -> **Root Cause**.

---

## 14. Summary & Architect Takeaways

*   **MTTR (Mean Time To Recovery)**: The most important metric. Failure is inevitable; recovery speed is what matters.
*   **Runbooks**: If you don't have a "How to restart" doc, you are not production ready.
