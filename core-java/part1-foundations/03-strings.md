# Strings: Immutability, Interning, and Performance

> **Part 1: Language Foundations**  
> **Level:** Principal Engineer  
> **Status:** Review Ready

---

## 0. Learning Objectives

*   **Developer**: Why `String` is immutable.
*   **Senior**: The difference between `new String("a")` and `"a"`.
*   **Architect**: Tuning the String Deduplication features in G1GC.

---

## 1. Internal Representation

### Java 8 vs Java 9+ (Compact Strings)
*   **Java 8**: `char[]` (UTF-16). 2 bytes per character.
*   **Java 9+**: `byte[]` + `coder`.
    *   If content is Latin-1 (mostly English), use 1 byte per char.
    *   If content contains Asian/Emoji, use 2 bytes.
    *   **Impact**: Saves ~50% heap space for typical English apps.

---

## 2. Deep Concept: The String Pool

The JVM maintains a special memory area called the **String Constant Pool**.

### The Mechanism
1.  `String s1 = "Hello";` -> JVM checks Pool. If missing, create "Hello" and put in Pool. Return ref.
2.  `String s2 = "Hello";` -> JVM finds "Hello" in Pool. Returns SAME ref.
    *   `s1 == s2` is **true**.

### The Pitfall
*   `String s3 = new String("Hello");`
    *   Creates a NEW Object on Heap (not in Pool).
    *   It wraps the "Hello" from the Pool.
    *   `s1 == s3` is **false**.
    *   **Anti-Pattern**: Never use `new String("literal")`. It creates redundant objects.

### 2.1 Manual Interning
*   `s3.intern()`: Forces the String into the Pool. Returns the Pool reference.
*   **Use Case**: You read 1 million CSV rows with "USA" as country. Use `intern()` to store "USA" once, not 1 million times.

---

## 3. String Concatenation

### The `+` Operator
*   **Code**: `String s = "a" + "b" + "c";`
*   **Java 8**: Compiled to `StringBuilder`.
*   **Java 9+**: Compiled to `invokedynamic` (StringConcatFactory). Optimization is delayed to runtime.

### Loop Pitfall
```java
// BAD - O(N^2)
String s = "";
for (int i=0; i<1000; i++) {
    s += i; // Creates a NEW StringBuilder and NEW String in every iteration.
}

// GOOD - O(N)
StringBuilder sb = new StringBuilder();
for (int i=0; i<1000; i++) {
    sb.append(i);
}
```

---

## 4. Performance & Benchmarking

### String Deduplication (G1GC)
*   **Flag**: `-XX:+UseStringDeduplication` (Default on in some modern JDKs).
*   **Mechanism**: During GC, G1 checks if two different String objects point to the same `byte[]` content.
*   **Action**: It re-points them to share the same underlying `byte[]`.
*   **Benefit**: Reduces Heap usage without code changes.

---

## 5. Security Considerations

### Why is String Immutable?
1.  **Security**: Used for ClassLoading, Network connections, Database URLs. If it were mutable, an attacker could change `user` to `admin` *after* a security check passed.
2.  **Thread Safety**: Immutable objects are automatically thread-safe.
3.  **HashCode Caching**: String calculates `hashCode` once and caches it. Critical for HashMap keys.

### Sensitive Data
*   **Password**: Never store passwords in `String`.
*   **Reason**: You cannot clear a String from memory (have to wait for GC).
*   **Fix**: Use `char[]` or `byte[]`. You can overwrite the array with zeros (`0x00`) immediately after use.

---

## 6. Summary & Architect Takeaways

1.  **Compact Strings**: Upgrade to Java 11+ to get free memory reduction (byte[] vs char[]).
2.  **Immutability is Key**: It enables Pooling, Caching, and Security.
3.  **Interning**: Use `s.intern()` or `-XX:+UseStringDeduplication` for repetitive text data (JSON keys, XML tags).
4.  **Sensitive Data**: `char[]` > `String`.

---
*Next Chapter: Generics, Type Erasure, and the PECS Principle.*
