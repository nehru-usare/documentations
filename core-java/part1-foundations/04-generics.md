# Generics: Type Erasure and Invariance

> **Part 1: Language Foundations**  
> **Level:** Principal Engineer  
> **Status:** Review Ready

---

## 0. Learning Objectives

*   **Developer**: Why `List<String>` is not a subtype of `List<Object>`.
*   **Senior**: Using `? extends T` (PECS) for flexible APIs.
*   **Architect**: Understanding strictly why Arrays are covariant but Generics are invariant.

---

## 1. Historical Context

*   **Pre-Java 5**: `List list = new ArrayList(); list.add("hi"); Integer i = (Integer) list.get(0);` -> Runtime `ClassCastException`.
*   **Java 5**: Generics introduced to provide **Compile-Time Safety**.
*   **Constraint**: Backward Compatibility. Old JVMs had to run new Code.
*   **Decision**: **Type Erasure**.

---

## 2. Specification-Level Behavior: Type Erasure

Generics exist ONLY at Compile Time.
*   **Source**: `List<String> list = new ArrayList<>();`
*   **Bytecode**: `List list = new ArrayList();`
*   **Runtime**: The JVM knows nothing about `<String>`.

### Consequence 1: No Primitive Generics
*   You cannot do `List<int>`.
*   Why? Because `List` becomes `List<Object>` (erased). `int` is not an Object.

### Consequence 2: Runtime Type Checks
*   `if (list instanceof List<String>)` -> **Illegal**.
*   Runtime only knows it's a `List`. You must use `List<?>`.

---

## 3. Deep Concept: Invariance vs Covariance

### Arrays are Covariant
*   `Integer[]` is a subclass of `Number[]`.
*   `Number[] nums = new Integer[10];` // Legal
*   `nums[0] = 3.14;` // **Runtime ArrayStoreException**.
*   **Verdict**: Not Type Safe.

### Generics are Invariant
*   `List<Integer>` is **NOT** a subclass of `List<Number>`.
*   `List<Number> nums = new ArrayList<Integer>();` // **Compile Error**.
*   **Verdict**: Type Safe.

---

## 4. Deep Technical Explanation: Wildcards (PECS)

To allow flexibility, we use Wildcards.
**PECS**: Producer Extends, Consumer Super.

### 1. Producer Extends (`? extends T`)
*   You want to **read** `T`s from a collection.
    ```java
    public void printNumbers(List<? extends Number> list) {
        Number n = list.get(0); // Safe. We know it's at least a Number.
        // list.add(1); // ERROR! We don't know if it's List<Integer> or List<Double>.
    }
    ```

### 2. Consumer Super (`? super T`)
*   You want to **write** `T`s into a collection.
    ```java
    public void addIntegers(List<? super Integer> list) {
        list.add(10); // Safe. It's List<Integer>, List<Number>, or List<Object>.
    }
    ```

---

## 5. Internal Mechanics: Bridge Methods

How does Polymorphism work if Types are erased?

```java
class Node<T> {
    public void setData(T data) { ... }
}

class MyNode extends Node<Integer> {
    // This is effectively: setData(Integer data)
    @Override public void setData(Integer data) { ... }
}
```

**The Problem**: After erasure, `Node` has `setData(Object data)`. `MyNode` has `setData(Integer data)`. The signatures don't match! Override fails.
**The Fix**: Compiler generates a **Bridge Method** in `MyNode`.

```java
// Generated Synthetic Method
public void setData(Object data) {
    this.setData((Integer) data); // Delegates to the specific method.
}
```

---

## 6. Production Debugging: Heap Pollution

**Scenario**: Mixing Raw Types and Generics.
```java
List<String> strings = new ArrayList<>();
List raw = strings; // Warning
raw.add(10); // Compiles! (Erasure)
String s = strings.get(0); // Runtime ClassCastException (Integer cannot be cast to String).
```

*   **Debug**: If you get a `ClassCastException` at a line that looks safe (`get(0)`), suspect Heap Pollution elsewhere.
*   **Tool**: `Collections.checkedList(list, String.class)` wraps the list and performs runtime checks on `add()`, catching the bug at the source.

---

## 7. Summary & Architect Takeaways

1.  **Erasure is the Constraint**: It dictates why we can't have `new T()`, `new T[]`, or `instanceof T`.
2.  **Use PECS**: On Library APIs, always use wildcards. `void copy(List<? extends T> src, List<? super T> dest)`.
3.  **Bridge Methods**: Be aware them when doing Reflection. You will see "extra" methods.
4.  **No Raw Types**: Never use `List` without `<>`. It disables all type safety.

---
*Next Chapter: Exceptions, Stack Traces, and Robustness.*
