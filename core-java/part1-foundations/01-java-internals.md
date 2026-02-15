# Java Internals: Primitives, Objects, and Memory Layout

> **Part 1: Language Foundations**  
> **Level:** Principal Engineer  
> **Status:** Review Ready

---

## 0. Learning Objectives

*   **Developer**: Understand why `int` is faster than `Integer`.
*   **Senior**: Visualize the memory layout of an Object on the Heap.
*   **Architect**: Calculate the exact memory footprint of a Class to optimize infrastructure costs.

---

## 1. Historical Context

Java (1995) was designed as a "Blue collar" language. It hid memory management (pointers) to make development safe.
*   **The Trade-off**: Pointers still exist; they are just hidden as "References".
*   **The Problem**: To support Objects (Heap) and high performance (Stack), Java created a dual-type system: **Primitives** and **Objects**.
*   **Modern Context**: Project Valhalla (Future Java) aims to bridge this gap with Value Types, but for now (Java 21), understanding the cost of Objects is critical.

---

## 2. Specification-Level Behavior

From **JVM Specification §2.4**:
*   **Primitives**: The JVM supports `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`. Their values are integral parts of the frame (Stack) or field (Heap).
*   **Reference Types**: A reference is a pointer to an Object. The Spec does not mandate *how* references are implemented, but in HotSpot, they are direct pointers (or compressed pointers).

---

## 3. Deep Concept: The Two Worlds of Java Memory

### 3.1 The Stack (Primitives & References)
*   Thread-Local.
*   Fastest access (L1 Cache friendly).
*   Stores:
    *   Primitives (`int x = 5`).
    *   Object **References** (`User u = ...`). *Note: The Object itself is NOT here.*

### 3.2 The Heap (Ref Objects & Arrays)
*   Shared across threads.
*   Managed by Garbage Collector.
*   Stores:
    *   The actual `User` object.
    *   Wrapper types (`Integer`, `Long`).

### 3.3 The Bridge: Autoboxing
*   **Code**: `Integer x = 5;`
*   **Bytecode**: `invokestatic Integer.valueOf(5)`.
*   **Implication**: Pure overhead. You moved a 4-byte value from the Stack to a 16-byte Object on the Heap, possibly triggering GC.

---

## 4. Deep Technical Explanation: Object Memory Layout

How much memory does `new Object()` take?
Answer: **16 Bytes** (in a standard 64-bit JVM).

### The Formula
`Object Size = Header + Fields + Padding`

### 4.1 The Object Header (12 Bytes)
Every object in Java has a header containing two machine words:

1.  **Mark Word (8 Bytes)**:
    *   Stores runtime data.
    *   **HashCode**: 31 bits.
    *   **GC Age**: 4 bits (Max 15. That's why Tenuring Threshold < 16).
    *   **Lock State**: Are we locked? Biased? (2 bits).
    *   **Thread ID**: If biased lock.

2.  **Klass Pointer (4 Bytes)**:
    *   Points to the `Class` metadata in Metaspace.
    *   *Wait, isn't a 64-bit pointer 8 bytes?* Yes, but... **Compressed Oops**.

### 4.2 Compressed Oops (`-XX:+UseCompressedOops`)
*   **Problem**: 64-bit pointers waste memory compared to 32-bit.
*   **Solution**: Since Objects are 8-byte aligned, the last 3 bits are always 0. The JVM shifts the address right by 3 bits.
*   **Result**: We can address 32GB of Heap with 32-bit (4-byte) pointers.
*   **Architect Note**: If you set Heap > 32GB, pointers double in size (4B -> 8B). You might effectively *lose* memory.

### 4.3 Padding (Alignment)
*   HotSpot enforces 8-byte alignment.
*   If Header (12B) + Fields (1B) = 13 Bytes, JVM adds **3 Bytes** of "Padding" to reach 16.

---

## 5. Pass-by-Value: The Great Misconception

**Rule**: Java is **strict Pass-by-Value**.
*   **Primitive**: Copies the bits (The value).
*   **Object**: Copies the bits of the **Reference** (The address).

### The "Proof"
```java
public void swap(Point a, Point b) {
    Point temp = a;
    a = b;
    b = temp;
}
// Caller:
Point p1 = new Point(0,0);
Point p2 = new Point(1,1);
swap(p1, p2); 
// p1 is STILL (0,0). The LOCAL references 'a' and 'b' swapped. 
// The caller's references 'p1' and 'p2' were not touched.
```

---

## 6. Performance & Benchmarking

### The Cost of Wrappers (`ArrayList<Integer>`)
*   `int[]`: Contiguous memory. CPU Prefetcher loves it.
*   `ArrayList<Integer>`: Array of References.
    *   Jump 1: Array -> Reference.
    *   Jump 2: Reference -> Heap Object (Random location).
    *   **Cache Miss**: High probability.
*   **Benchmark**: Summing 1M ints vs 1M Integers can be **5x-10x slower**.

---

## 7. Production Debugging Guide

### OutOfMemoryError
*   **Scenario**: Heap usage is high.
*   **Check**: Are you using `Integer` where `int` suffices?
*   **Tool**: `jhsdb jmap --heap --pid <pid>`.
*   **Heap Dump**: Open in Eclipse MAT. Look for "Retained Size".

### "False Sharing" (Memory)
*   If two volatile Objects sit on the same Cache Line (64 bytes), updating thread A invalidates cache for Thread B.
*   **Fix**: `@Contended` (Java 8+) adds padding to isolate the object in memory.

---

## 8. Summary & Architect Takeaways

1.  **Primitives are king**: Always prefer `long` over `Long` for local data.
2.  **Wrappers are pointers**: An `Integer` is not a number; it's a pointer to a header wrapping a number. The overhead is ~400% (4 bytes vs 16 bytes).
3.  **Know the 32GB Cliff**: Increasing Heap from 31GB to 33GB often **reduces** available memory due to Compressed Oops disabling.
4.  **Pass-by-Value**: Never rely on a method swapping reference arguments. It won't work.

---
*Next Chapter: Deep Dive into OOP (Composition, Inheritance, and the Liskov Substitution Principle).*
