# Proxy Pattern in Spring

> **Part 5: Patterns Inside Spring**  
> **Status:** The Magic Behind Annotations

---

## 1. What Uses Proxies?
Almost all "Magic" annotations in Spring work via the **Proxy Pattern**.
*   `@Transactional`
*   `@Cacheable`
*   `@Async`
*   `@Retryable`
*   `@PreAuthorize` (Security)

---

## 2. How it Works
When you annotate a bean:
```java
@Service
public class PaymentService {
    @Transactional
    public void pay() { ... }
}
```

Spring **does not** register `PaymentService` in the context.
It registers a **Proxy** (a dynamically generated subclass or wrapper).

```java
// Spring generates this pseudo-code at runtime
public class PaymentService$$EnhancerBySpringCGLIB extends PaymentService {
    
    private TransactionManager tm;
    private PaymentService target;

    public void pay() {
        try {
            tm.begin();      // 1. Aspect Logic
            target.pay();    // 2. Call Real Method
            tm.commit();     // 3. Aspect Logic
        } catch (Exception e) {
            tm.rollback();   // 4. Recovery Logic
        }
    }
}
```

---

## 3. JDK Dynamic Proxy vs CGLIB

### JDK Dynamic Proxy
*   **Requirement**: Target **MUST** implement an Interface.
*   **Mechanism**: Uses `java.lang.reflect.Proxy`. Creates a class implementing the interface.
*   **Pros**: Standard Java.
*   **Cons**: Can't proxy classes without interfaces.

### CGLIB (Code Generation Library)
*   **Requirement**: Target Class must **NOT** be `final`. Methods must **NOT** be `final`.
*   **Mechanism**: Generates a **Subclass** of the target.
*   **Pros**: Works on any non-final class.
*   **Modern Spring**: Boot 2.0+ uses CGLIB by default.

---

## 4. The Self-Invocation Pitfall
Because the logic lives in the **Proxy**, calling a method from **within** the same class bypasses the proxy.

```java
public void batchPay() {
    // THIS WILL FAIL TRANSACTION!
    // Because you are calling 'this.pay()', not 'proxy.pay()'
    for (Order o : orders) pay(o); 
}

@Transactional
public void pay(Order o) { ... }
```

**Solution**: Inject `PaymentService` into itself (Self-Injection) or move `pay()` to a different bean.
