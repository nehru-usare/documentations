# Class Loading: Delegation and Isolation

> **Part 4: JVM Internals**  
> **Level:** Principal Engineer  
> **Status:** Loaded

---

## 0. Learning Objectives

*   **Developer**: Resolving `ClassNotFoundException` vs `NoClassDefFoundError`.
*   **Senior**: How Tomcat hosts multiple apps with different library versions.
*   **Architect**: Designing plugin systems with Custom ClassLoaders.

---

## 1. The Delegation Model

1.  **Bootstrap ClassLoader**: Loads JDK core (`rt.jar`, `java.lang.*`). Native code.
2.  **Platform ClassLoader** (Ext): Loads JDK extensions.
3.  **App ClassLoader**: Loads `-classpath`.

**Logic**: "Ask Parent First". If Parent can't find it, try self.
**Why**: Security. You can't write your own `java.lang.String` and trick the App ClassLoader into loading it. Bootstrap will always load the real one first.

---

## 2. Custom ClassLoaders (Isolation)

Used by Containers (Tomcat, OSGi, Spring Boot).
*   **Tomcat**:
    *   Has `CommonLoader` (Shared libs).
    *   Has `WebappLoader` (Per WAR file).
    *   **Breaks Delegation**: WebappLoader looks locally *first* (Child First) to allow WARs to override shared libs.

---

## 3. Production Debugging Guide

### NoClassDefFoundError
*   **Meaning**: The class was present at **Compile Time**, but missing at **Runtime**.
*   **Cause**: Maven dependency mismatch (Scope `provided` but not in container).
*   **Static Initializer Fail**: If `static { ... }` throws an exception, the class fails to load. Subsequent attempts throw `NoClassDefFoundError`.

### ClassLoader Leaks
*   **Scenario**: You redeploy a WAR. Old ClassLoader stays in memory. Metaspace fills up. `OutOfMemoryError: Metaspace`.
*   **Cause**: A ThreadLocal or JDBC Driver holds a reference to the Old ClassLoader.

---

## 4. Summary & Architect Takeaways

1.  **Isolation**: Use separate ClassLoaders to run conflicting versions of a library (Shadowing).
2.  **Resources**: `getResourceAsStream()` usually delegates to the ClassLoader of the caller.
3.  **Flat Classpath**: The "JAR Hell". The Module System (Java 9) attempts to fix this.

---
*End of Part 4. Next: Part 5 - Modern Java.*
