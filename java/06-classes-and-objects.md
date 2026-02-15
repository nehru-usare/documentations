# 🧱 Classes & Objects: The Memory & Runtime Internals

> **Document Level:** Architect (15+ years experience)  
> **Focus:** Object Header Layout, Compressed OOPS, Memory Padding, and JEP 447 Constructors

---

## 🏗️ Layer 1: The Object Layout (The 64-bit Reality)

When you call `new Object()`, what actually happens in RAM? 

### 1. The Object Header
In a standard 64-bit JVM, an object has a header consisting of:
-   **Mark Word (8 bytes)**: Stores the hashcode, GC age (4 bits), bias lock bit, and lock state.
-   **Klass Word (8 bytes)**: Stores the pointer to the class metadata in Metaspace.
    -   **Optimization**: **Compressed OOPs** (Ordinary Object Pointers) can reduce this to 4 bytes if the heap is < 32GB. 
-   **Array Length (4 bytes)**: Only present if the object is an array.

### 2. Memory Padding & Object Alignment
The JVM aligns objects to **8-byte boundaries**. 
- **The Padding**: If an object takes 21 bytes, the JVM will add 3 bytes of padding to make it 24 bytes.
- **Architect Impact**: Understanding padding is critical for high-performance caches. If two threads update fields on the same cache line (64 bytes), they cause **False Sharing**.
- **Fix**: Use `@Contended` to force fields onto separate cache lines.

---

## 🛠️ Layer 2: The Lifecycle - From Eden to Old

### 1. Fast-Path Allocation (TLAB)
The JVM first tries to allocate the object in the thread's **TLAB** (Thread Local Allocation Buffer). This is lock-free and extremely fast.

### 2. Initialization
- **Preparation**: The JVM zeroes out the memory (guaranteeing default values like `null` or `0`).
- **Initialization**: The `<init>` method (your constructor) is called.

### 3. Aging & Scavenging
Objects that survive a **Minor GC** in the Eden space are moved to Survivor Space (S0). Each survivor increment their **GC Age** in the Mark Word.
- **Promotion**: Once an object reaches the **Tenuring Threshold** (default 15), it is promoted to the Old Generation.

---

## 🚀 Layer 3: Modern Constructors (JEP 447)

### 1. Flexible Constructor Bodies (Java 22 Preview / 21+)
Historically, `super()` or `this()` had to be the **absolute first statement** in a constructor.
- **The Pain**: You couldn't validate arguments or perform complex logic *before* calling the parent constructor.
- **The Solution**: Java 22 allows statements *before* `this()`/`super()` as long as they don't access the instance being created.

---

## 📜 Layer 4: Static vs. Instance initialization

### 1. Class Initialization (`<clinit>`)
Triggered when the class is first used. It runs static blocks and initializes static fields.
- **Thread Safety**: The JVM handles the locking for `<clinit>`, making the "Initialization-on-demand holder" idiom the safest way to implement a Singleton.

### 2. Instance Initialization (`<init>`)
Runs before the constructor body but after the super-constructor. 
- **Architect Note**: If you initialize a field in the declaration line, it actually gets moved into the `<init>` method by the compiler.

---

## 🏁 Layer 5: Modern Entities - Records vs. POJOs

### 1. The Records Revolution
A `record` is a **Transparent Data Carrier**.
- **Internals**: The compiler generates a final class with private final fields, a public constructor, and auto-generated `equals`, `hashCode`, and `toString`.
- **Architect Rule**: Use records for any data that doesn't need to change (e.g., event payloads, database rows, API responses).

---

## 🧭 Interview Prep & Architect Scenarios

### Q: What is the maximum GC age an object can reach?
**A**: 15. This is because the "Age" is stored in 4 bits of the Mark Word (`2^4 = 16`, but the max value is 15).

### Q: Why is `Object.finalize()` deprecated?
**A**: It is unpredictable, slow, and can resurrect dead objects. It delays garbage collection because the object must be moved to a finalization queue instead of being cleaned immediately. Use `java.lang.ref.Cleaner` or "Try-with-resources" instead.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [05-operators-and-control-flow.md](./05-operators-and-control-flow.md) | Control Flow |
| ⏩ **Next** | [07-oop-concepts.md](./07-oop-concepts.md) | OOP Pillars |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026