# 🧬 Generics & Enums: The Type Safety Masterclass

> **Document Level:** Architect (15+ years experience)  
> **Focus:** Type Erasure, Binary Compatibility, PECS Rule, and Enum Strategy Patterns

---

## 🏗️ Layer 1: Generics & The Erasure Trade-off

Why does Java use **Type Erasure** instead of **Reification** (like C# or C++)?

### 1. The History: Java 1.4 vs. 5.0
In 2004, Java 5 introduced Generics. To ensure that 10 years of existing code (using raw types) could run on a new JVM, the designers chose **Type Erasure**.
- **Erasure**: At compile time, `<T>` is replaced by its bound (usually `Object`). 
- **Bridge Methods**: The compiler generates hidden "bridge" methods to ensure polymorphism works correctly with erased types.

### 2. Erasure vs. Reification
- **Java (Erasure)**: `List<String>` and `List<Integer>` are the SAME class at runtime. 
- **C# (Reification)**: They are DIFFERENT classes. 
- **Architect Impact**: Java's approach allows for "Generic Libraries" that are backward compatible, but it prevents features like `if (obj instanceof T)` or `new T()`.

---

## 🛠️ Layer 2: Master Class - Wildcards & PECS

The most confusing part of Generics is **Variance**.

### 1. The Problem: Covariance
Is `List<String>` a subtype of `List<Object>`? **No.** If it were, you could add an `Integer` to a `List<Object>` reference that points to a `String` list, causing a crash.

### 2. The Solution: PECS Rule
**Producer Extends, Consumer Super.**
- **Producer (Extends)**: `List<? extends Shape>` - You can **READ** from it (it produces Shapes), but you cannot write to it.
- **Consumer (Super)**: `List<? super Circle>` - You can **WRITE** to it (it consumes Circles), but you cannot read from it as a specific type safely.

> [!TIP]
> **Architect Insight**: Use PECS to make your APIs flexible. A method like `copy(List<? extends T> src, List<? super T> dest)` is the gold standard for generic manipulation.

---

## 🚀 Layer 3: Advanced Enums (More than just Constants)

In Java, an `Enum` is a **Full Class** that inherits from `java.lang.Enum`.

### 1. Enums as Singletons
The most robust way to implement a **Singleton** in Java is via Enum. It provides native serialization and thread-safety by the JVM.

### 2. The Strategy Pattern
Instead of a giant `switch(type)` inside a method, define an abstract method in the Enum and let each constant implement it.
```java
public enum Operation {
    PLUS { public int apply(int a, int b) { return a + b; } },
    MINUS { public int apply(int a, int b) { return a - b; } };
    public abstract int apply(int a, int b);
}
```

### 3. EnumSet & EnumMap
These are highly optimized bit-vector collections.
- **Architect Note**: If you have a collection of Enum keys, an `EnumMap` is significantly faster than a `HashMap` because it uses a simple array under the hood.

---

## 📜 Layer 4: Modern Evolution (Records & Type Inference)

### 1. Records with Generics
Java 17+ Records work perfectly with Generics.
```java
public record Result<T>(T data, String message) {}
```

### 2. The `var` Keyword
Type inference (`var`) works by looking at the initializer. 
- **Caution**: `var list = new ArrayList<>();` results in an `ArrayList<Object>`. Always specify the type on the right if using `var`.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: Why can't we create an array of a generic type (`new T[10]`)?
**A**: Because of **Array Covariance** vs. **Generic Invariance**. Arrays in Java know their type at runtime. Generics don't (erasure). If the JVM allowed `new T[]`, it wouldn't know which type to verify against at runtime, potentially leading to an `ArrayStoreException` later.

### Q: Explain "Type Token" or "Super Type Tokens".
**A**: Since `List<String>.class` doesn't exist, we use "Type Tokens" (like `Class<T>`) or the "Neal Gafter" Super Type Token pattern (used in Guice/Jackson) to "capture" generic type information at runtime using subclassing.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [10-exception-handling.md](./10-exception-handling.md) | Exceptions |
| ⏩ **Next** | [12-streams-and-functional-programming.md](./12-streams-and-functional-programming.md) | Streams & FP |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026