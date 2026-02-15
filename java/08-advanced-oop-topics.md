# 🧠 Advanced OOP Topics: The Binary & Runtime Mechanics

> **Document Level:** Architect (15+ years experience)  
> **Focus:** Bridge Methods, The Diamond Problem, Inner Class Memory Leaks, and Dispatch Internals

---

## 🏗️ Layer 1: The Hidden Binary - Bridge Methods

When you combine Generics with Overriding, the compiler performs "Magic".

### 1. The Scenario
```java
public class Parent<T> {
    public void set(T value) { ... }
}
public class Child extends Parent<String> {
    @Override
    public void set(String value) { ... }
}
```

### 2. The Bridge
Because of **Type Erasure**, `Parent.set` takes an `Object`. But `Child.set` takes a `String`. These signatures don't match!
- **The Fix**: The compiler generates a **Bridge Method** in `Child.class`:
  ```bytecode
  public void set(Object v) { // Bridge
      set((String) v);        // Cast and call the actual method
  }
  ```
- **Architect Insight**: You can see these in stack traces or when using Reflection (`method.isBridge()`). They are critical for maintaining polymorphism in erased generic hierarchies.

---

## 🛠️ Layer 2: The Diamond Problem in Interfaces

Java 8 introduced `default` methods. What happens if you implement two interfaces with the same `default` method?

### 1. Conflict Resolution
```java
interface A { default void hello() { ... } }
interface B { default void hello() { ... } }
class C implements A, B {
    @Override
    public void hello() { 
        A.super.hello(); // Manual resolution required
    }
}
```
- **Architect Rule**: Classes always win over Interfaces. If a superclass has the method, the interface's `default` is ignored.

---

## 🚀 Layer 3: Inner Classes & Memory Leaks

A common source of `OutOfMemoryError` in UI and Android frameworks.

### 1. The Implicit Reference
Non-static inner classes hold a **hidden reference** to their parent instance (`Outer.this`).
- **The Leak**: If you pass an inner class instance (like a Listener) to a long-lived service, the **Outer class** instance can never be Garbage Collected, even if it's no longer needed.
- **The Fix**: Always use **`static` inner classes** unless you explicitly need to access the outer class's instance fields.

---

## 📜 Layer 4: Dispatch Mechanics - Static vs. Dynamic Binding

### 1. Static Binding (Compile-time)
Used for `private`, `static`, and `final` methods. The compiler knows exactly which code to run.
- **Performance**: Zero runtime overhead.

### 2. Dynamic Binding (Runtime)
Used for everything else (Virtual methods). 
- **Mechanism**: The JVM uses the `vtable` lookup at runtime based on the actual object type.

---

## 🚦 Layer 5: Design Pattern: The "Template" via Defaults

Instead of an `abstract` base class, use an interface with `default` methods to provide a template.
- **Architect Benefit**: This allows your classes to "Inherit" the template behavior while still being able to inherit from another class (Vertical vs Horizontal scaling).

---

## 🧭 Interview Prep & Architect Scenarios

### Q: Why can an Interface have `static` methods since Java 8?
**A**: To provide utility methods associated with the interface (like `Stream.of()` or `Comparator.comparing()`) without needing a separate "Helper" or "Utils" class.

### Q: Explain the difference between "Overriding" and "Hiding" (Shadowing).
**A**: **Overriding** is for instance methods (Dynamic binding). **Hiding** is for static methods and fields (Static binding). If you "hide" a static method in a child class, calling it via the parent reference will still execute the parent's version.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [07-oop-concepts.md](./07-oop-concepts.md) | OOP Pillars |
| ⏩ **Next** | [09-packages-and-access-modifiers.md](./09-packages-and-access-modifiers.md) | Packages |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026