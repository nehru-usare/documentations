# Distributed Monolith

> **Part 7: Anti-Patterns**  
> **Severity:** 🔴 Critical  
> **Status:** The Worst of Both Worlds

---

## 1. The Symptom
You have 10 Microservices.
*   **Requirement**: "Add a 'Middle Name' field to the User."
*   **Action**: You have to deploy `UserService`, `OrderService`, `ProfileService`, and `ReportService` **at the same time**.
*   **Result**: If you deploy one without the others, the system breaks.

---

## 2. Signs you have a Distributed Monolith
1.  **Shared Database**: 10 services connect to the same `USERS` table.
2.  **Lock-Step Deployments**: "We deploy everything on Tuesday night."
3.  **Chatty Interfaces**: Service A calls Service B 50 times to render one page.
4.  **Shared Libraries**: A `common-lib.jar` that contains Domain Objects. If you change a User object in the lib, you must rebuild all 10 services.

---

## 3. Why is this Bad?
You have expected the **Agility** of Microservices.
You got the **Complexity** of Distributed Systems (Latency, Network Failures).
But you depend on each other like a **Monolith**.
**Result**: High Latency + Low Agility.

---

## 4. The Fix

### 1. Database per Service
Each service MUST own its data. If `OrderService` needs User data, it should store a local copy (Cache/Replica) or ask `UserService` via API (not SQL).

### 2. Interface Stability
Use **Consumer Driven Contracts** (CDC).
Ensure that Service A can be upgraded to v2.0 without Service B breaking.

### 3. Kill Shared Libraries
Do not share **Domain Logic** in libraries. Share **Utils** (String manipulation, Date formatting) only.
It is better to duplicate the `User` class in 5 services than to couple them all to a single Jar.

---

## 5. Architect Takeaway
*   **Loose Coupling** is the #1 goal of Microservices. If you are tightly coupled, you assume all the risk with no reward.
*   **Rule of Thumb**: If two services must be deployed together, they should probably be merged into one.
