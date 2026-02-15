# Iterator Pattern

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐ (Beginner)  
> **Status:** Built-in to Java

---

## 0. Learning Objectives

*   **Beginner**: Use `for (Item i : list)`.
*   **Developer**: Implement `Iterable<T>` for a Custom Data Structure.
*   **Architect**: Design lazy-loading streams.

---

## 1. Problem Statement

### The Traversal Problem
You have different collections: `ArrayList` (Index based), `LinkedList` (Node based), `HashSet` (Hash based), `Tree` (Graph based).
*   **Without Iterator**: You need different loops for each.
    *   `for (int i=0; i<arr.size(); i++)`
    *   `while (node != null) node = node.next`
*   **Solution**: Uniform interface `hasNext()` and `next()`.

---

## 2. Real-World Analogy

**TV Remote Channel Surfing**
*   **Next Channel**: You press "Next".
*   You don't care how the TV stores channels (Cable, Satellite, Antenna). You just go Next -> Next -> Next.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. Implementing Iterable
To make your custom class usable in `for-each` loops, implement `Iterable`.

```java
import java.util.Iterator;

public class MyCollection implements Iterable<String> {
    private String[] items = {"A", "B", "C"};

    @Override
    public Iterator<String> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<String> {
        private int index = 0;

        public boolean hasNext() { return index < items.length; }
        public String next() { return items[index++]; }
    }
}
```

### 2. Usage
```java
MyCollection col = new MyCollection();
for (String s : col) { // Works elegantly!
    System.out.println(s);
}
```

---

## 8. Advantages

1.  **Uniformity**: Client code works on List, Set, Tree exactly the same way.
2.  **Safety**: Iterators can implement "Fail-Fast" behavior (Throw exception if list is modified during iteration).

---

## 14. Interview Questions

### Basic
1.  **What is `ConcurrentModificationException`?** (Occurs if you modify a collection using `.remove()` while iterating over it, without using the Iterator's remove method).
2.  **Difference between `Iterator` and `ListIterator`?** (`ListIterator` can go backwards).

---

## 16. Summary & Architect Takeaways

*   **Streams**: Java 8 Streams are basically internal iterators on steroids.
