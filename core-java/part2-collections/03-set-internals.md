# Set Internals: The Map Wrapper

> **Part 2: Collections Framework**  
> **Level:** Principal Engineer  
> **Status:** Review Ready

---

## 0. Learning Objectives

*   **Developer**: Why `HashSet` consumes exactly the same memory as `HashMap`.
*   **Senior**: TreeSet sorting logic.
*   **Architect**: Choosing `CopyOnWriteArraySet` for small, read-heavy lists.

---

## 1. Deep Technical Explanation: HashSet

**It is a HashMap.**
```java
public class HashSet<E> {
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object(); // Dummy value

    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
}
```
*   **Memory Impact**: You pay for the Key (Your Object) + Value (Dummy Object) + Node Overhead.
*   **Performance**: Same as HashMap (O(1)).

---

## 2. Deep Technical Explanation: TreeSet

**It is a TreeMap (Red-Black Tree).**
*   **Sorted**: Keeps elements in Natural Order (Comparable) or Custom Order (Comparator).
*   **Cost**: O(log N) for Add/Remove/Contains.
*   **Balancing**: Ensures tree height remains logarithmic.

---

## 3. Specialized Sets

### CopyOnWriteArraySet
*   **Baced by**: `CopyOnWriteArrayList`.
*   **Add**: Iterate entire array to check duplicate (O(N)). Then Copy array (O(N)). Total **O(N)**.
*   **Use Case**: Listeners / Observers. Small sets, rarely modified, frequently iterated.

---

## 4. Summary & Architect Takeaways

1.  **HashSet vs TreeSet**: O(1) vs O(log N). Use HashSet unless you need sorting.
2.  **Memory**: `HashSet` is heavy. For primitive sets, consider libraries like **Eclipse Collections** (`IntSet`).
3.  **LinkedHashSet**: Preserves insertion order. Slightly more memory (double linked list running through entries).

---
*Next Chapter: Stacks, Queues, and Deques.*
