# Java Security Evolution: Beyond the Security Manager

> **Part 6: Advanced I/O & Standards**  
> **Level:** Principal Engineer  
> **Status:** Secured

---

## 0. Learning Objectives

*   **Developer**: Why `System.setSecurityManager()` throws an Exception in Java 21.
*   **Senior**: Implementing strict encapsulation without SM.
*   **Architect**: Securing Java apps in the Container Era.

---

## 1. The Security Manager (1995 - 2021)

*   **Intent**: Allow running "Untrusted Code" (Applets) safely. Process Sandboxing.
*   **Mechanism**: AccessController, doPrivileged blocks. Check every file access.
*   **Reality**:
    *   Performance overhead.
    *   Extremely complex to configure correctly.
    *   Applets died.
*   **Status**:
    *   **Java 17**: Deprecated for Removal.
    *   **Java 21**: Disallowed by default.
    *   **Future**: Code removal.

---

## 2. The New Security Model

If we can't sandbox code *inside* the JVM, how do we secure it?

### 2.1 External Isolation (Containers)
*   **Strategy**: Don't trust the JVM to isolate code. Trust the **OS / Hypervisor**.
*   **Docker/Kubernetes**: Run untrusted code in a separate container with limited capabilities (`cap-drop ALL`) and resources (cgroups).

### 2.2 Strong Encapsulation (Modules)
*   **JPMS**: Prevents reflection into internal APIs. Reduces attack surface.

---

## 3. Cryptography Updates

### Modern Defaults
*   **TLS**: TLS 1.3 is default.
*   **Algorithms**: EdDSA (Ed25519) signatures. Chacha20-Poly1305 ciphers.
*   **Best Practice**: Don't hardcode "RSA". Use "KeyGenerator" and let the JDK pick the best provider.

---

## 4. Summary & Architect Takeaways

1.  **Delete `doPrivileged`**: It's dead code.
2.  **Containerize**: If you execute user scripts (e.g., Groovy/Nashorn), run them in an ephemeral Docker container, not in your main JVM.
3.  **Keep Updated**: Security patches in OpenJDK are critical. Automated builds should pick the latest patch version (e.g., 21.0.2).

---
*This creates the Core Java Engineering Handbook.*
