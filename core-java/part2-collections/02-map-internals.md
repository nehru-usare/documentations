# Map Internals: HashMap, Treeification, and Concurrency

> **Part 2: Collections Framework**  
> **Level:** Principal Engineer  
> **Status:** Review Ready

---

## 0. Learning Objectives

*   **Developer**: Why `hashCode()` dictates performance.
*   **Senior**: The O(log N) guarnatee in Java 8 (Treeification).
*   **Architect**: Choosing between `HashMap`, `ConcurrentHashMap`, and `EnumMap`.

---

## 1. Deep Technical Explanation: HashMap

Structure: Array of Buckets. `Node<K,V>[] table`.
Default Size: 16.

### 1.1 The `put(K, V)` Algorithm
1.  **Hash**: Calculate `h = key.hashCode() ^ (h >>> 16)`. (XOR high bits to spread entropy).
2.  **Index**: `i = (n - 1) & h`. (Determine bucket index).
3.  **Collision**:
    *   **Empty Bucket**: New Node.
    *   **Collision**: Traverse the Linked List.
    *   **Duplicate**: If `Node.key.equals(key)`, replace Value.
    *   **Treeify**: If Chain length > 8, convert Linked List to **Red-Black Tree**.

### 1.2 Treeification (Java 8+)
*   **Problem**: In Java 7, a bad hash function created a long Linked List -> O(N) lookup. Denial of Service attack vector.
*   **Solution**: Java 8 converts long chains to Trees.
*   **Performance**: Worst case becomes **O(log N)**.

### 1.3 Resizing
*   **Load Factor**: 0.75 (Default).
*   **Threshold**: Capacity * Load Factor (16 * 0.75 = 12).
*   **Action**: When size > 12, double array size (16 -> 32).
*   **Rehash**: ALL entries must be recalculated and moved. Expensive!

---

## 2. Deep Technical Explanation: ConcurrentHashMap

Structure: Array of Buckets (Same as HashMap).
**Difference**: Thread Safety without global `synchronized`.

### 2.1 Java 7 (Segmented Locking)
*   Split Map into 16 Segments.
*   Lock only the segment you are writing to.

### 2.2 Java 8+ (CAS + Synchronized Bucket)
*   **No Segments**.
*   **Read**: `volatile` reads. Lock-Free.
*   **Write (Empty Bucket)**: `CAS` (Compare-And-Swap) to insert new Node. Lock-Free.
*   **Write (Collsion)**: `synchronized(NodeHead)`. Locks ONLY that single bucket chain.
*   **Result**: Extremely high concurrency.

---

## 3. Specialized Maps

### EnumMap
*   **Context**: Key is an Enum.
*   **Internals**: Uses a simple **Array** (`V[]`). The index is `enum.ordinal()`.
*   **Performance**: Faster than HashMap. No hashing. CPU Cache friendly.

### IdentityHashMap
*   **Context**: Use `==` instead of `equals()`.
*   **Internals**: Linear Probing (Open Addressing). No Linked Lists.

---

## 4. Production Debugging Guide

### The "Infinite Loop" (Java 7)
*   **Scenario**: Two threads resize a HashMap simultaneously.
*   **Result**: The Linked List becomes circular. CPU spikes to 100%.
*   **Fix**: Never use `HashMap` in multi-threaded code. Use `ConcurrentHashMap`.

### Memory Leak
*   **Scenario**: Using Objects as Keys but not overriding `equals/hashCode`.
*   **Result**: Duplicate entries. Map grows forever.

---

## 5. Summary & Architect Takeaways

1.  **Initial Capacity**: `new HashMap<>(expectedSize / 0.75F + 1.0F)`. Avoid resizing.
2.  **Immutability**: Keys must be immutable.
3.  **Concurrency**: `ConcurrentHashMap` > `Collections.synchronizedMap()`.
4.  **EnumMap**: Always use if keys are Enums.

---
*Next Chapter: Set Internals - The Map Wrapper.*
