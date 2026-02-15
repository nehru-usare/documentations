# Overengineering

> **Part 7: Anti-Patterns**  
> **Severity:** 🟡 Medium  
> **Status:** Resume Driven Development

---

## 1. The Symptom
**Requirement**: "Create an endpoint that returns 'Hello World'."

**The Code**:
1.  `HelloWorldController`
2.  `HelloWorldService` interface + `HelloWorldServiceImpl`
3.  `HelloWorldRepository` interface + `HelloWorldRepositoryImpl`
4.  `MessageFactory` (Abstract Factory)
5.  `MessageBuilder`
6.  `HelloWorldAbstractBaseClass`
7.  `CleanArchitectureLayer`

**Result**: 15 files to return a String.

---

## 2. Why it happens
1.  **Speculative Generality**: "What if we need to switch the greeting to XML in the future?" (Spoiler: You won't).
2.  **Pattern Mania**: "I just learned the Visitor Pattern, I must use it here."
3.  **Boredom**: Writing simple code is boring. Creating a complex framework is fun.

---

## 3. The Core Principles

### YAGNI (You Ain't Gonna Need It)
Do not implement features/abstractions until you actually need them.
*   Don't build a Plugin System if you have 0 plugins.
*   Don't build a Generic Repository if you only have 1 entity.

### KISS (Keep It Simple, Stupid)
Simple code is easier to read, test, and debug.
Complex code hides bugs.

---

## 4. When is it NOT Overengineering?
If you are building a **Library** or **Framework** for others, you need abstraction.
But if you are building an **Application**, you usually don't.

---

## 5. Architect Takeaway
*   **Refactor Later**: Start simple. If complexity grows, **Result** (Refactor) to a Pattern. Don't start with the Pattern.
*   **The 3-Strike Rule**: Don't abstract code until you have copied it 3 times.
