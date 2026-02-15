# The Module System (JPMS): Strong Encapsulation

> **Part 5: Modern Java**  
> **Level:** Principal Engineer  
> **Status:** Exported

---

## 0. Learning Objectives

*   **Developer**: Why `module-info.java` exists.
*   **Senior**: Fixing "package is visible, but module is not" errors.
*   **Architect**: Reducing Docker image size with `jlink`.

---

## 1. The Problem: Classpath Hell

*   **Jar Hell**: Two versions of `commons-logging` on classpath. Who wins? Random.
*   **Internals Exposed**: You can use `sun.misc.Unsafe` because everything is public.
*   **Monolithic JDK**: You need the entire 200MB JRE to run "Hello World".

---

## 2. The Solution: Modules (Java 9)

**Definition**: A group of Packages.
**Descriptor**: `module-info.java` at root.

```java
module com.mycompany.app {
    requires java.sql;          // explicit dependency
    exports com.mycompany.api;  // public API
    // com.mycompany.internal is NOT exported. Hidden.
}
```

### 2.1 Validated Configuration
The JVM checks graph validity at **startup**.
*   Missing module? Error.
*   Cyclic dependency? Error.

---

## 3. JLink: Custom Runtimes

Since the JDK itself is modularized (`java.base`, `java.logging`...), we can pick only what we need.
*   **Command**: `jlink --add-modules java.base,java.sql --output my-jre`.
*   **Result**: A 30MB custom JRE containing only the bare minimum.
*   **Impact**: Tiny Docker images (Alpine + JLink = Microservices Heaven).

---

## 4. Summary & Architect Takeaways

1.  **Library Authors**: Must add `Automatic-Module-Name` to Manifest to start supporting modules.
2.  **App Developers**: Most apps run on Classpath (Backwards Compatible Mode). Migration is optional but recommended for strict encapsulation.
3.  **JLink**: The killer feature for Cloud Native Java.

---
*Next Chapter: Java 17-21 New Features.*
