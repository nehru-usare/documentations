# Functional Programming: Streams and Lambdas

> **Part 5: Modern Java**  
> **Level:** Principal Engineer  
> **Status:** Reduced

---

## 0. Learning Objectives

*   **Developer**: Using `map`, `filter`, `reduce`.
*   **Senior**: Why `Stream` reuse throws Exception.
*   **Architect**: Parallel Streams and ForkJoinPool saturation.

---

## 1. Deep Concept: Laziness

*   **Code**: `list.stream().filter(x -> x > 5).limit(2)`.
*   **Execution**:
    *   `filter` is an **Intermediate Operation**. It does nothing immediately.
    *   `limit` is a **Short-Circuiting State**.
    *   **Terminal Operation** (`collect`, `forEach`) triggers the pipeline.
*   **Optimization**: It pulls elements one by one. If 2 elements satisfy filter, it stops. It touches minimal data.

---

## 2. Internal Mechanics: Lambdas

*   **Pre-Java 8**: Anonymous Inner Classes (`new Runnable() { ... }`). Created a `.class` file on disk. High overhead.
*   **Java 8**: `invokedynamic`.
    *   The Lambda body becomes a `private static` method in your class.
    *   At runtime, `LambdaMetafactory` spins a lightweight class to call it.
    *   **Cost**: Almost zero allocation. High performance.

---

## 3. Parallel Streams

`list.parallelStream()`.
*   **Mechanism**: Splits data (Spliterator) and submits tasks to `ForkJoinPool.commonPool()`.
*   **Danger**: The Common Pool is shared by **entire JVM**.
*   **Risk**: If you do Blocking I/O in a Parallel Stream, you freeze the entire application's parallel capabilities.
*   **Rule**: Use Parallel Stream ONLY for pure CPU tasks (matrix multiplication).

---

## 4. Summary & Architect Takeaways

1.  **Side Effects**: Keep Streams pure. Don't modify external state inside `map()`.
2.  **Debug**: `stream.peek(x -> log(x))` allows inspection without terminating.
3.  **Primitive Streams**: Use `IntStream`, `LongStream` to avoid boxing overhead.

---
*Next Chapter: Optional and Null Safety.*
