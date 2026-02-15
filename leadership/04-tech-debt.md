# Managing Technical Debt

> **Part 1: Process**  
> **Target Audience:** Tech Leads  
> **Status:** Refactoring...

---

## 0. Core Concpet
*   **Technical Debt**: Taking a shortcut now (Speed) to pay interest later (Maintenance).
*   **It is NOT bad**: Just like financial debt, it's a tool. A startup with 0 debt usually fails (too slow).
*   **Unmanaged Debt is bad**: Bankruptcy.

---

## 1. The 4 Quadrants of Debt (Martin Fowler)
1.  **Reckless / Deliberate**: "We don't have time for tests". (Toxic).
2.  **Reckless / Inadvertent**: "What is Layering?". (Training issue).
3.  **Prudent / Deliberate**: "We must ship for Xmas. We will fix the UI later". (Valid).
4.  **Prudent / Inadvertent**: "Now that we finished, we realize we should have used Strategy Pattern". (Learning).

---

## 2. Strategies to Pay it Down

### 1. The Boy Scout Rule
*   "Leave the campground cleaner than you found it."
*   Touching `OrderService.java`? Rename that one bad variable. Extract that one long method.
*   **Impact**: Continuous, small improvements.

### 2. The 20% Rule
*   Google/Spotify model.
*   Dedicate 20% of Sprint Capacity to "Engineering Health" (Refactoring, Upgrades, Debt).
*   Product Owners should not dictate this. It's the cost of doing business.

### 3. The Tech Debt Radar
*   Maintain a list/backlog.
*   "Upgrade Log4j" (High Risk).
*   "Rename Utils" (Low Risk).

---

## 3. Migration Patterns (The Big Debt)

### The Strangler Fig Pattern
*   **Goal**: Rewrite Monolith -> Microservices.
*   **Wrong**: "The Big Bang Rewrite" (Rewrite everything in v2, launch in 2 years). **Fails 100%**.
*   **Right**: Strangler.
    1.  Put API Gateway in front of Monolith.
    2.  Write *New* features in New Service. Route Gateway to it.
    3.  Slowly migrate *Old* features one by one.
    4.  Monolith shrinks until it disappears.

---

## 4. Selling Refactoring to Business
*   Business doesn't care about "Clean Code".
*   They care about **Velocity** and **Stability**.
*   **Pitch**: "This refactoring will let us ship features 2x faster next month" or "This will reduce outages by 50%".

---

## 5. Summary
1.  **Visibilty**: Track debt in Jira.
2.  **Balance**: Don't aim for "Perfect Code". Aim for "Sustainable Code".
3.  **Tests**: You cannot refactor without Tests. (You are just changing bugs).
