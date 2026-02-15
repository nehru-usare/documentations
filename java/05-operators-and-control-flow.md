# ⚙️ Operators & Control Flow: The Logic of Efficiency

> **Document Level:** Architect (15+ years experience)  
> **Focus:** Bitwise Mastery, Hardware Branch Prediction, and Pattern Matching Flow

---

## 🏗️ Layer 1: The Binary Level - Bitwise Operators

High-frequency trading (HFT) and game engines use bitwise logic for extreme performance and memory density.

### 1. The Bitwise Toolkit
-   **`&` (AND)**: Masking specific bits.
-   **`|` (OR)**: Setting specific bits (Flags).
-   **`^` (XOR)**: Toggling bits. Fast encryption/checksums.
-   **`~` (NOT)**: Flipping all bits.
-   **`<<` / `>>`**: Shifting bits (Multiply/Divide by power of 2).
-   **`>>>` (Unsigned Right Shift)**: Fills the high bits with `0`. Crucial for handling signed vs. unsigned byte data in network protocols.

### 2. Architect Scenario: The "BitSet"
Instead of a `List<Boolean>` (which uses massive memory), use a `long` or a `BitSet`. A single `long` can track 64 independent status flags with O(1) read/write time and zero GC overhead.

---

## 🛠️ Layer 2: Control Flow & The CPU

How you write your `if` statements affects how the CPU executes your code.

### 1. Branch Prediction
Modern CPUs try to guess which way an `if` statement will go.
- **The Cost**: If the CPU guesses wrong (**Branch Misprediction**), it must flush the instruction pipeline, costing 15-20 cycles.
- **Architect Rule**: Keep hot-path logic predictable. Sort your data if you are checking conditions in a loop (e.g., `if (val > threshold)` is faster on sorted data due to predictable branches).

### 2. Loop Unrolling
The JIT compiler often "unrolls" your loops (e.g., executing 4 iterations in one block) to reduce the overhead of the loop counter and the "Jump" instruction.

---

## 🚀 Layer 3: Modern Control Flow - Java 21 Pattern Matching

Java has moved from "Imperative branching" to "Declarative selection".

### 1. Pattern Matching for `switch`
```java
return switch (obj) {
    case Integer i -> i * i;
    case String s && s.length() > 5 -> s.toUpperCase(); // Guard clause
    case null -> 0;
    default -> -1;
};
```
- **Architect Impact**: Replaces messy `instanceof` + casting blocks with clean, compiler-verified logic.
- **Exhaustivity**: When using `sealed` types, the compiler ensures every possible subclass is handled, removing the need for a "default" catch-all.

---

## 📜 Layer 4: The "Tail Call" Debate

### 1. Why Java lacks Tail Call Optimization (TCO)
Functional languages (like Scala/Haskell) optimize recursion into loops. Java doesn't.
- **The Why**: Java prioritizes **Stack Trace Integrity**. TCO would erase the stack frames, making debugging nearly impossible for the JVM.
- **Architect Fix**: Always convert deep recursion into an iterative loop or use a `Deque` (Trampolining) to avoid `StackOverflowError`.

---

## 🏁 Layer 5: Short-Circuiting & Boolean Logic

### 1. `&&` vs `&` (and `||` vs `|`)
- **`&&` (Short-Circuit)**: If the first part is false, the second is **never** executed.
- **`&` (Logical)**: Both sides are always executed.
- **Architect Warning**: Never use `&` for boolean checks if the second part depends on the first (e.g., `if (obj != null & obj.isValid())` will crash).

---

## 🧭 Interview Prep & Architect Scenarios

### Q: How does the JVM optimize a `switch` on a String?
**A**: At compile time, the JVM calculates the `hashCode()` of the strings and uses a `lookupswitch` or `tableswitch` (O(1) or O(log N) jump table). It then performs one final `.equals()` check to handle hash collisions.

### Q: Why is `i = i + 1` different from `i += 1` for a `short`?
**A**: `i = i + 1` requires an explicit cast (`short = (short)(i + 1)`) because the `+` operator promotes the result to an `int`. `i += 1` has an implicit cast built-in.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [04-java-basics-and-syntax.md](./04-java-basics-and-syntax.md) | Basics |
| ⏩ **Next** | [06-classes-and-objects.md](./06-classes-and-objects.md) | Classes |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026