# 🚀 Performance & Optimization Patterns: Mechanical Sympathy in Java

> **Document Level:** Architect (15+ years experience)  
> **Focus:** Cache Locality (L1/L2), Memory Barriers, Inlining, and False Sharing

---

## 🏗️ Layer 1: Mechanical Sympathy - Aligning with the Hardware

The fastest code is the code that respects the CPU and Memory architecture.

### 1. The CPU Cache Line (64 Bytes)
The CPU doesn't read 1 byte from RAM; it reads a "Cache Line" (usually 64 bytes).
- **The Optimization**: If you store data in a flat `byte[]` or `long[]`, when the CPU reads element `0`, it also pulls elements `1...7` into the L1 cache for free.
- **Architect Rule**: **Avoid `LinkedList`**. Every node in a `LinkedList` is a separate object pointer, likely causing a **Cache Miss** during traversal. Use `ArrayList` for 99% of high-performance scenarios.

### 2. False Sharing & `@Contended`
If two threads update two different `long` variables that happen to live on the same 64-byte cache line, the CPU will "thrash" the cache line across cores, killing performance.
- **The Fix**: Use the `@jdk.internal.vm.annotation.Contended` annotation or manual padding to ensure "hot" variables are isolated on their own cache lines.

---

## 🛠️ Layer 2: JIT Optimization Mastery

The C2 compiler is your best friend—but only if you write "JIT-friendly" code.

### 1. The Inlining Budget
Inlining (replacing a method call with the body) is the #1 optimization. However, the JIT has a "Budget":
-   **FreqInlineSize**: 325 bytes (by default). If a method is frequently called but larger than this, it won't be inlined.
-   **MaxInlineSize**: 35 bytes. For rarely called methods.
- **Architect Rule**: Keep methods small and focused. "Clean Code" isn't just about readability; it makes your app faster by allowing the JIT to inline more aggressively.

### 2. Monomorphic vs. Megamorphic Calls
- **Monomorphic**: A method call that always goes to the same implementation. The JIT can inline it perfectly.
- **Megamorphic**: A method call that goes to 3+ different implementations (e.g., an interface with many subclasses). The JIT must use a virtual method table lookup every time.
- **Optimization**: Favor direct types over interfaces in performance-critical inner loops to stay monomorphic.

---

## 🚀 Layer 3: Data Structures for High-Scale

### 1. The Cost of Hashing
`HashMap` is fast (O(1)), but it involves calculating a Hash, checking for equality, and potential link-list/tree traversal on collisions.
- **Optimization**: For very large sets of numeric data, use **Primitive Collections** (like those in High-Scale-Lib or Agrona) to avoid boxing and memory overhead.

### 2. Off-Heap Memory (`DirectBuffer`)
Storing massive amounts of data in the JVM Heap causes long GC pauses.
- **Pattern**: Allocate memory outside the heap using `ByteBuffer.allocateDirect()`. 
- **Architect Note**: Off-heap memory is NOT managed by the GC. You must handle deallocation manually (or via `Cleaner`), but it allows you to handle TBs of data with zero GC impact.

---

## 📜 Layer 4: String Optimization Patterns

### 1. `String.intern()` vs. Custom Caching
While `intern()` saves memory, it uses a global, native JVM table that can become a bottleneck under high contention.
- **Pattern**: Use a `ConcurrentHashMap` with a limited size to cache frequently used strings (like category names) to avoid duplicate objects.

### 2. `StringConcatFactory`
Modern Java (9+) uses `invokedynamic` for string concatenation. It is more efficient than `StringBuilder` for simple cases because the JVM can optimize the concatenation strategy at runtime based on the strings' types and lengths.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: What is "Escape Analysis"?
**A**: It is the JIT's ability to see if an object created inside a method ever "leaves" that method. If it doesn't, the JIT can perform **Scalar Replacement** (placing fields in registers or the stack) and eliminate the heap allocation entirely.

### Q: How does a `volatile` variable affect performance?
**A**: A `volatile` write causes a **Store Memory Barrier**, and a read causes a **Load Memory Barrier**. This prevents the CPU from using its local store-buffer, forcing it to talk to the cache/RAM. While essential for visibility, over-using `volatile` in a hot loop can slow down execution by 10x.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [14-memory-management-and-jvm-tuning.md](./14-memory-management-and-jvm-tuning.md) | Memory & GC |
| ⏩ **Next** | [16-jvm-monitoring-and-production-tuning.md](./16-jvm-monitoring-and-production-tuning.md) | Monitoring |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026