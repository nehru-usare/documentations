# OOP Deep Dive: Design & Contracts

> **Part 1: Language Foundations**  
> **Level:** Principal Engineer  
> **Status:** Review Ready

---

## 0. Learning Objectives

*   **Developer**: Why "Composition over Inheritance" is not just a slogan.
*   **Senior**: Implementing `equals()` and `hashCode()` correctly to avoid `HashMap` bugs.
*   **Architect**: Designing APIs using Interfaces and Default Methods for backward compatibility.

---

## 1. Historical Context

*   **Java 1.0**: Inheritance was King. Deep hierarchies (`Component` -> `Container` -> `Panel` -> `Applet`).
*   **The Problem**: **Fragile Base Class**. Changing a superclass breaks subclasses invisibly.
*   **Java 8**: Introduction of **Default Methods** allowed Interfaces to evolve, reducing the need for Abstract Classes.

---

## 2. Specification-Level Behavior

From **JLS §8.1.5 (Inheritance)**:
*   A class can extend only one class but implement multiple interfaces.
*   **Method Overriding**: A subclass method overrides a superclass method if the signature matches.
*   **Liskov Substitution Principle (LSP)**: A subclass must be substitutable for its superclass without breaking the program.

---

## 3. Deep Concept: Composition over Inheritance

**Inheritance is White-Box Reuse**. You break encapsulation because the subclass depends on the implementation details of the superclass.
**Composition is Black-Box Reuse**. You depend only on the public interface.

### The "Instrumented HashSet" Bug
```java
// Broken Inheritance
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c); // BUG: HashSet.addAll() calls add() internally!
    }
}
```
**Result**: `addAll(List.of(1, 2, 3))` increments count by 6 (3 from `addAll`, 3 from internal `add`).
**Fix**: Use Composition. Hold a `private Set<E> set` inside.

---

## 4. Deep Technical Explanation: The Object Contract

### 4.1 `equals()` and `hashCode()`
**The Contract (JLS §3.1)**:
*   If `a.equals(b)`, then `a.hashCode() == b.hashCode()`.
*   If `hashCode` differs, they are NOT equal.
*   **Violation**: If you override `equals` but not `hashCode`, your object gets lost in a `HashMap` (It goes to the wrong bucket).

### 4.2 The `equals()` Recipe (Effective Java)
1.  Check `this == obj`. (Optimization).
2.  Check `obj instanceof ThisClass`. (Type check).
3.  Cast `obj` to `ThisClass`.
4.  Compare significant fields. (Primitives with `==`, Objects with `Objects.equals`, Arrays with `Arrays.equals`).

### 4.3 `clone()` is broken
*   **Never** use `Object.clone()`. It performs a shallow copy and bypasses constructors.
*   **Alternative**: Use a **Copy Constructor** or **Static Factory**.
    ```java
    public User(User other) {
        this.name = other.name;
        this.address = new Address(other.address); // Deep copy manually
    }
    ```

---

## 5. Modern OOP: Interface Evolution

### Default Methods
*   **Problem**: You want to add `stream()` to `Collection` interface.
*   **Impact**: Every class implementing `Collection` (ArrayList, HashSet, MyCustomList) breaks because they don't implement `stream()`.
*   **Solution**: `default Stream<E> stream() { ... }`.
*   **Architect Note**: Default methods are for **Interface Evolution**, not for Multiple Inheritance.

---

## 6. Performance & Benchmarking

### Virtual Calls vs Static Calls
*   **Virtual Call (`invokevirtual`)**: JVM must look up the VTable (Virtual Method Table) to find the correct overriding method. slightly slower.
*   **Static Call (`invokestatic`)**: Logic is bound at compile time. Fast.
*   **Final**: Declaring a method `final` *can* help the JIT inline it, removing the lookup cost.

---

## 7. Production Debugging Guide

### The "Disappearing Key"
*   **Scenario**: `map.put(key, val)`. Later `map.get(key)` returns `null`.
*   **Cause**: The Key is **Mutable**. You changed a field in the Key *after* putting it in the Map. Its `hashCode` changed. It is now in the wrong bucket.
*   **Fix**: Keys in Map MUST be Immutable.

---

## 8. Summary & Architect Takeaways

1.  **LSP is paramount**: If `Square extends Rectangle`, and you set `width=5`, `height` should not change. If it does, you violated LSP.
2.  **Composition > Inheritance**: Only use inheritance for "is-a" relationships where the superclass is designed for extension.
3.  **Immutable Keys**: Never use a mutable object as a Map Key.
4.  **Avoid `clone()`**: It is a legacy mistake. Use Copy Constructors.

---
*Next Chapter: Strings, Immutability, and the Intern Pool.*
