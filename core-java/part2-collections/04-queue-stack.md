# Queue & Stack: ArrayDeque vs PriorityQueue

> **Part 2: Collections Framework**  
> **Level:** Principal Engineer  
> **Status:** Review Ready

---

## 0. Learning Objectives

*   **Developer**: Why you should never use `java.util.Stack`.
*   **Senior**: How `ArrayDeque` implements a circular buffer.
*   **Architect**: Using `PriorityQueue` for task scheduling.

---

## 1. The Legacy: Stack Class

*   **Status**: Deprecated (conceptually).
*   **Why**: Extends `Vector`. Synchronized. Slow.
*   **Replacement**: `Deque<Integer> stack = new ArrayDeque<>();`

---

## 2. Deep Technical Explanation: ArrayDeque

**Double Ended Queue**.
*   **Implementation**: Circular Array.
*   **Pointers**: `head` and `tail`.
*   **Capacity**: Must be power of 2.
*   **Resizing**: Doubles when full. Uses `System.arraycopy`.
*   **Performance**: Faster than `LinkedList` (Cache Locality). Used as Stack or Queue.

---

## 3. Deep Technical Explanation: PriorityQueue

**Binary Min-Heap**.
*   **Structure**: Array `Object[] queue`.
*   **Logic**:
    *   `queue[0]` is the smallest element.
    *   Children of `queue[k]` are at `2*k+1` and `2*k+2`.
*   **Operations**:
    *   `peek()`: O(1).
    *   `add()`: O(log N) (Sift Up).
    *   `poll()`: O(log N) (Sift Down).
*   **Use Case**: Scheduling tasks, Dijkstra's algorithm.

---

## 4. BlockingQueues (Concurrency Preview)

For multi-threaded Producer-Consumer:
1.  **ArrayBlockingQueue**: Fixed size array. One ReentrantLock.
2.  **LinkedBlockingQueue**: Linked Nodes. Two Locks (Head, Tail). Higher concurrency.

---

## 5. Summary & Architect Takeaways

1.  **Circular Buffers**: `ArrayDeque` is the Swiss Army Knife of stacks/queues.
2.  **Heaps**: `PriorityQueue` does not guarantee iteration order. Only `poll()` order.
3.  **Nulls**: Queues generally do not allow nulls (as `poll()` returning null indicates empty).

---
*End of Part 2. Next: Part 3 - Concurrency & The Java Memory Model.*
