# Java 21 Deep Dive: The LTS Features

> **Part 5: Modern Java**  
> **Level:** Principal Engineer  
> **Status:** Sequenced

---

## 0. Learning Objectives

*   **Developer**: Using `list.getFirst()` and `list.reversed()`.
*   **Senior**: Writing multi-line interpolated strings with String Templates.
*   **Architect**: Understanding the full scope of the Java 21 LTS release.

---

## 1. Sequenced Collections (JEP 431)

**The Problem**:
*   Get first element of List? `list.get(0)`.
*   Get first element of Deque? `deque.getFirst()`.
*   Get first element of SortedSet? `set.first()`.
*   **Inconsistent**.

**The Solution**:
New interfaces `SequencedCollection`, `SequencedSet`, `SequencedMap`.
*   **Methods**:
    *   `getFirst()`, `getLast()`
    *   `addFirst()`, `addLast()`
    *   `removeFirst()`, `removeLast()`
    *   `reversed()` (Returns a view, not a copy!)

```java
List<String> list = new ArrayList<>();
list.addLast("Side");
list.addFirst("Server");
System.out.println(list.getFirst()); // "Server"
```

---

## 2. String Templates (JEP 430 - Preview)

**The Problem**: String concatenation is ugly. `String.format` is verbose. Text Blocks (`"""`) are static.

**The Solution**: String Interpolation.
```java
String name = "Java";
String info = STR."Hello \{name}";
```
*   **STR**: A template processor.
*   **Safety**: It simulates a method call, preventing injection attacks (if using a SQL processor).

---

## 3. Unnamed Patterns & Variables (JEP 443 - Preview)

**The Problem**: Unused variables clutter code.
```java
try { ... } catch (Exception e) { ... } // 'e' is unused
```

**The Solution**: The Underscore `_`.
```java
try { ... } catch (Exception _) { ... }
```
```java
if (obj instanceof Point(int x, int _)) { ... } // Don't care about y
```

---

## 4. Virtual Threads (JEP 444)

(Covered in Part 3).
**Reminder**: This is the flagship feature of Java 21.
*   **Throughput**: 1M+ threads per JVM.
*   **Blocking**: Code that blocks on IO unmounts from the OS thread.

---

## 5. Summary & Architect Takeaways

1.  **Sequenced Collections**: Refactor your utility classes. Replace `list.get(list.size()-1)` with `list.getLast()`.
2.  **Previews**: Enable `--enable-preview` to use Templates and `_`.
3.  **LTS Status**: Java 21 is the target platform for the next 3-5 years. Upgrade now.

---
*End of Part 5.*
