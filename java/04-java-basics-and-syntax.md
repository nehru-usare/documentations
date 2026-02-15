# 🧪 Java Basics & Syntax: The Deep Foundation

> **Document Level:** Architect (15+ years experience)  
> **Focus:** Primitive vs. Object performance, String Internals (Compact Strings), and Project Valhalla

---

## 🏗️ Layer 1: Data Types & The Memory Cost

Java is a hybrid language: **Primitives** for speed, **Objects** for flexibility.

### 1. Primitives vs. Wrappers (The Autoboxing Trap)
- **Primitive (`int`)**: Lives on the stack (usually). Fast, no header overhead.
- **Wrapper (`Integer`)**: Lives on the heap. 16-byte header + 4-byte data = 20 bytes (padded to 24).
- **Architect Impact**: A `List<Integer>` with 1 million items consumes ~24MB. An `int[]` with 1 million items consumes ~4MB. 
- **Performance**: Autoboxing in a loop (`sum += list.get(i)`) creates millions of temporary `Integer` objects, causing massive GC pressure.

### 2. Project Valhalla (Value Types)
The future of Java basics. Valhalla aims to bring "Codes like a class, works like an int."
- **Goal**: Allow data structures that behave like primitives (no header, allocated on stack/inline) but have methods and types like classes.

---

## 🛠️ Layer 2: String Internals - The Most Important Class

`java.lang.String` is unique in the JVM.

### 1. String Constant Pool
Strings are **Immutable**. When you create a literal `String s = "Hello"`, it goes into the **Constant Pool** in the Metaspace/Heap.
- **Interning**: `s.intern()` manually adds a string to the pool. Use this to save memory when you have millions of duplicate strings (e.g., Country names in a dataset).

### 2. Compact Strings (Java 9+)
Historically, Strings used `char[]` (2 bytes per character).
- **The Optimization**: Most strings are Latin-1 (1 byte). Java 9+ uses `byte[]` with an `encoding` flag. If the string contains only Latin-1, it takes half the memory.

### 3. String Concatenation (`+` vs `StringBuilder`)
- **Old Way**: The compiler used `StringBuilder`.
- **New Way (Java 9+)**: The compiler uses `StringConcatFactory` with the **`invokedynamic`** instruction. This allows the JVM to choose the most efficient concatenation strategy at runtime without recompiling your code.

---

## 🚀 Layer 3: The Evolution of Syntax

### 1. Local Variable Type Inference (`var`)
Introduced in Java 10. It is **not** dynamic typing (like JS). It is compile-time sugar.
- **Architect Rule**: Use `var` only when the type is obvious on the right side. Avoid it for method returns where the type is unknown without looking at the source.

### 2. Modern Switch Expressions
Java 14+ turned `switch` into an expression that can return a value. 
- **Advantage**: No "Fall-through" bugs, and the compiler enforces exhaustivity.

---

## 📜 Layer 4: Floating Point Precision & `BigDecimal`

Never use `double` or `float` for money.
- **The Why**: Binary floating-point can't represent `0.1` exactly. 
- **The Fix**: Use `BigDecimal`. 
- **Performance Note**: `BigDecimal` is an object and is immutable. Every operation creates a new instance. Use `long` (storing cents) if performance is ultra-critical.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: Why are Strings Immutable in Java?
**A**: 1. **Security**: Strings are used for File paths, URLs, and DB connections. If they were mutable, a hacker could change the path after a security check. 2. **Caching**: Allows the String Pool to work. 3. **Thread-Safety**: Immutable objects are natively thread-safe.

### Q: What is the difference between `==` and `.equals()`?
**A**: `==` checks for **Reference Equality** (do they point to the same memory address?). `.equals()` checks for **Content Equality** (do they have the same value?). For primitives, `==` is used for value comparison.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [03-setup-and-environment.md](./03-setup-and-environment.md) | Setup |
| ⏩ **Next** | [05-operators-and-control-flow.md](./05-operators-and-control-flow.md) | Operators |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026