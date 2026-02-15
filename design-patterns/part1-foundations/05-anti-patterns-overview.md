# Anti-Patterns Overview

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Senior)  
> **Status:** Warning Signs

---

## 0. Learning Objectives

*   **Beginner**: Recognize that "Bad Code" has names and classifications.
*   **Developer**: Stop creating "God Objects".
*   **Architect**: Identify "Lava Flow" (Dead code that nobody deletes).

---

## 1. Problem Statement

### The Road to Hell
Anti-Patterns often start as *good ideas*.
*   "I'll put all utility functions in one class to find them easily." -> **God Object**.
*   "I'll add a boolean flag to handle this edge case." -> **Spaghetti Code**.

### Definition
An **Anti-Pattern** is a common response to a recurring problem that is usually ineffective and risks being highly counterproductive.

---

## 2. Real-World Analogy

*   **The Blob (God Object)**: A Swiss Army Knife that is 5-feet long and weighs 100kg. It does everything, but you can't lift it.
*   **Spaghetti Code**: A tangled mess of cables behind your TV. If you pull one, the lamp falls off the table.

---

## 3. Core Concepts (🟢 Beginner Level)

### Common Anti-Patterns (Detailed in Part 7)

1.  **God Object (The Blob)**: One class does everything. 5000 lines. 50 methods.
2.  **Spaghetti Code**: Unstructured, difficult to follow logic. `GOTO` statements (or heavily nested `if-else`).
3.  **Lava Flow**: Old code that nobody understands, so nobody deletes it. It hardens like lava.
4.  **Copy-Paste Programming**: Duplicating code blocks instead of extracting a method.
5.  **Golden Hammer**: Applying the same pattern to every problem.

---

## 4. Software Entropy

### Broken Windows Theory
*   If a building has a broken window and it's not fixed, vandals will break more windows.
*   **Code**: If you leave a "Quick Hack" in the codebase, the next developer will add another hack on top of it.
*   **Solution**: Refactor immediately. (Boy Scout Rule).

---

## 7. Internal Mechanics (Architect Level 🔴)

### Technical Debt
*   **Strategic Debt**: "We need to launch for Black Friday. We will skip clean code and fix it In January." (Acceptable).
*   **Unintentional Debt**: "We don't know how to write clean code." (Dangerous).

### The Interest Rate
*   Anti-Patterns are **High Interest Debt**.
*   Every time you touch a God Object, you pay "Interest" (longer time to understand, bugs introduced).

---

## 14. Interview Questions

### Basic
1.  **What is an Anti-Pattern?** (A common bad solution).
2.  **Give an example of an Anti-Pattern.** (God Object, Magic Numbers).

### Intermediate
3.  **What is "Shotgun Surgery"?** (Changing one thing requires edits in many classes).
4.  **Why is "Copy-Paste" dangerous?** (If you find a bug in one copy, you must fix it in all copies. You will miss one).

### Advanced
5.  **Is Singleton an Anti-Pattern?** (Debatable. Yes, if it holds mutable global state. No, if it's a stateless service like in Spring).
6.  **How do you fix "Lava Flow"?** (Code coverage tools. Delete 1 method at a time. Canary deployment).

---

## 16. Summary & Architect Takeaways

*   **Awareness is the first step**. If you can name the demon, you can fight it.
*   **Don't blame the past**. The previous team probably had strict deadlines. Fix it forward.
