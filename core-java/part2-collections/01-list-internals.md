# List Internals: ArrayList vs LinkedList & Hardware Sympathy

> **Part 2: Collections Framework**  
> **Level:** Principal Engineer  
> **Status:** Review Ready

---

## 0. Learning Objectives

*   **Developer**: Determining when (never) to use LinkedList.
*   **Senior**: Understanding `modCount` and Fail-Fast iterators.
*   **Architect**: CPU Cache Locality effects on Collection choice.

---

## 1. Deep Technical Explanation: ArrayList

Backed by `Object[] elementData`.

### 1.1 Resizing Strategy
*   **Initial Capacity**: 10 (Default).
*   **Growth**: 50% (`newCapacity = oldCapacity + (oldCapacity >> 1)`).
*   **Copy**: `System.arraycopy` (Native, fast) moves elements to new array.

### 1.2 Performance
*   **Random Access**: O(1).
*   **Insert at End**: Amortized O(1).
*   **Insert in Middle**: O(N) (Shift elements).

---

## 2. Deep Technical Explanation: LinkedList

Doubly Linked List (`Node<E>`).

### 2.1 Memory Overhead
*   **ArrayList**: 1 Reference (4-8 bytes) per element.
*   **LinkedList**: Node object (Header 12B + Prev 4B + Next 4B + Item 4B) = **24-32 Bytes** per element.
*   **Overhead**: ~4x more memory than ArrayList.

---

## 3. Hardware Sympathy (CPU Cache)

This is why **LinkedList is almost always dead**.

### The L1/L2 Cache
*   CPU fetches memory in **Cache Lines** (64 bytes).
*   **ArrayList**: Elements are contiguous. Fetching `list[0]` likely loads `list[1]`, `list[2]` into L1 Cache. Traversing is instant.
*   **LinkedList**: Nodes are scattered across Heap.
    *   Fetch Node A.
    *   Fetch Next Pointer -> Cache Miss (Wait 100 cycles).
    *   Fetch Node B.
    *   **Result**: LinkedList iteration is significantly slower than ArrayList on modern hardware.

---

## 4. Internal Mechanics: Fail-Fast

### The `modCount` variable
*   Every `add()`, `remove()` increments `modCount`.
*   **Iterator**: Captures `expectedModCount = modCount` at creation.
*   **Next()**: Checks `if (modCount != expectedModCount) throw ConcurrentModificationException`.
*   **Purpose**: Detect bugs immediately rather than returning undefined data.

---

## 5. Summary & Architect Takeaways

1.  **Default to ArrayList**: Usage of LinkedList is rare (only for heavy head/tail shuffling).
2.  **Size Matters**: If you know the size, `new ArrayList<>(1000)`. Avoid resizing overhead.
3.  **Vector is Dead**: It is synchronized (slow). Use `ArrayList` or `CopyOnWriteArrayList`.

---
*Next Chapter: The most important collection: HashMap.*
