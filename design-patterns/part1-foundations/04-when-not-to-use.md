# When NOT to Use Design Patterns

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐⭐ (Senior / Lead)  
> **Status:** The Voice of Reason

---

## 0. Learning Objectives

*   **Beginner**: Understand that simple code is better than "clever" code.
*   **Developer**: Stop creating Interfaces that only have one implementation.
*   **Architect**: Reject Pull Requests that add unnecessary complexity.

---

## 1. Problem Statement

### The "Golden Hammer" Syndrome
"I just learned the Visitor Pattern! Now I will use it to print a list of strings!"
*   **Result**: 10 classes where 1 loop was enough.
*   **Cost**: Maintenance nightmare. New developers can't understand the "Hello World" logic because it's buried in factories.

### Resume-Driven Development (RDD)
Developers implementing complex patterns (Event Sourcing, Microservices) just to put it on their CV, not because the business needs it.

---

## 2. Real-World Analogy

**The Crane vs The Hand**
*   **Problem**: Move a grocery bag from the car to the house.
*   **Simple Solution**: Pick it up with your hand.
*   **Pattern Solution**: Rent a construction crane. Build a scaffolding. Lift the bag.
    *   *Technically correct?* Yes.
    *   *Efficient?* No.

---

## 3. Core Concepts (🟢 Beginner Level)

### YAGNI (You Ain't Gonna Need It)
Don't implement a pattern for a "future requirement" that doesn't exist yet.
*   "We might switch databases next year!" -> (Spoiler: You probably won't).
*   *Action*: Just use the simplest `PostgresDriver` now. Refactor later **IF** you switch.

### KISS (Keep It Simple, Stupid)
A system is best kept simple rather than made complicated.
*   `if (x) else (y)` is better than `StrategyFactory.getStrategy(x).execute()`.

### The Rule of Three
1.  Write code once.
2.  Copy-paste it once (Total 2).
3.  Copy-paste it again (Total 3) -> **Refactor**.
    *   Don't abstract on the 2nd usage. You might be abstracting the wrong thing.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. Overengineering (Bad)

```java
// Abstract Factory for printing "Hello"
interface TextPrinterFactory { Printer getPrinter(); }
class ConsolePrinterFactory implements TextPrinterFactory { ... }
interface Printer { void print(String s); }
class ConsolePrinter implements Printer { ... }

public class Main {
    public static void main(String[] args) {
        Printer p = new ConsolePrinterFactory().getPrinter();
        p.print("Hello");
    }
}
```

### 1. Simple (Good)

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

---

## 7. Internal Mechanics (Architect Level 🔴)

### Complexity Cost
*   **Cognitive Load**: Reading 10 files takes more mental energy than reading 1.
*   **Debuggability**: Stepping through a Proxy/Decorator chain in a debugger is painful.
*   **Performance**: JIT compilers are good, but excessive indirection can prevent inlining.

---

## 14. Interview Questions

### Basic
1.  **What is YAGNI?** (Don't build features you don't need yet).
2.  **What is the downside of Design Patterns?** (Complexity, Learning Curve).

### Intermediate
3.  **When should you introduce a Factory Pattern?** (When object creation logic is complex or conditional. Not for `new String()`).
4.  **Is code duplication always bad?** (No. "Duplication is far cheaper than the wrong abstraction." - Sandi Metz).

### Advanced
5.  **Critique this code: `interface IString { String get(); }`**. (Useless abstraction. Primitives don't need interfaces).
6.  **How do you handle a junior dev who over-engineers?** (Code Review. Ask "What problem does this solve right now?").

---

## 16. Summary & Architect Takeaways

*   **Premature Optimization is the root of all evil**.
*   **Premature Abstraction is the root of all complexity**.
*   **Best Pattern**: No Pattern (if the code is simple).
