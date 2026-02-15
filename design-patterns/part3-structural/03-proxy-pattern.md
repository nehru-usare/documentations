# Proxy Pattern

> **Part 3: Structural Patterns**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Magic behind Spring

---

## 0. Learning Objectives

*   **Beginner**: Understand Lazy Loading (Hibernate).
*   **Developer**: Implement a JDK Dynamic Proxy.
*   **Architect**: Understand how Spring AOP works under the hood (CGLIB vs JDK).

---

## 1. Problem Statement

### The "Expensive Object" Problem
You have a `HighResolutionImage` (100MB).
You have a list of 1000 images.
If you load all of them into memory just to show a list of filenames, your app crashes.
*   **Solution**: Create a `ImageProxy`. It looks like an Image. It has `display()`. But it only loads the real image *when* `display()` is called.

### The "Security" Problem
You have a `BankService`.
You want to check if the user is Admin before calling `transfer()`.
*   **Solution**: Create a `SecurityProxy`. It checks credentials, then calls the real method.

---

## 2. Real-World Analogy

**The Credit Card**
*   **Real Object**: Cash in the Bank.
*   **Proxy**: The Plastic Card.
*   You use the card (Proxy) to pay. The card communicates with the Bank (Real Object) to move the money.

**The Assistant**
*   You want to meet the CEO (Real Object).
*   You talk to the Assistant (Proxy).
*   The Assistant filters spam, checks your calendar, and *then* lets you see the CEO.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Provide a surrogate or placeholder for another object to control access to it.

### Types of Proxies
1.  **Virtual Proxy**: Lazy loading (Hibernate).
2.  **Protection Proxy**: Access control (Spring Security).
3.  **Remote Proxy**: RMI / Feign Client (Local object represents a remote service).
4.  **Logging Proxy**: Log every method call.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. Static Proxy (Manual Wrapper)
```java
interface Image { void display(); }

class RealImage implements Image {
    public RealImage(String filename) { loadFromDisk(filename); } // Expensive!
    public void display() { System.out.println("Displaying..."); }
}

class ProxyImage implements Image {
    private RealImage realImage;
    private String filename;

    public ProxyImage(String filename) { this.filename = filename; }

    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename); // Lazy Load
        }
        realImage.display();
    }
}
```

### 2. JDK Dynamic Proxy (The Magic)
This creates a proxy at **runtime** for any interface.

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

interface Service { void serve(); }
class RealService implements Service { 
    public void serve() { System.out.println("Serving..."); } 
}

class LoggingHandler implements InvocationHandler {
    private final Object target; // The Real Object

    public LoggingHandler(Object target) { this.target = target; }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Start method: " + method.getName());
        Object result = method.invoke(target, args); // Call Real Object
        System.out.println("End method: " + method.getName());
        return result;
    }
}

// Client Usage
Service real = new RealService();
Service proxy = (Service) Proxy.newProxyInstance(
    RealService.class.getClassLoader(),
    new Class<?>[] { Service.class },
    new LoggingHandler(real)
);
proxy.serve(); // Prints: Start... Serving... End...
```

---

## 6. Spring Boot Implementation (Crucial!)

Spring uses Proxies for **Everything**.
*   `@Transactional`: Spring wraps your `@Service` bean in a Proxy.
    1.  Proxy starts Transaction.
    2.  Proxy calls your method.
    3.  If Exception -> Proxy Rollback.
    4.  If Success -> Proxy Commit.

### JDK Proxy vs CGLIB
*   **JDK Proxy**: Requires the target to implement an **Interface**.
*   **CGLIB (Code Generation Library)**: Can proxy **Classes** (by subclassing them and overriding methods).
*   *Modern Spring Boot*: Uses CGLIB by default so you don't *need* interfaces for every service, but interfaces are still best practice.

---

## 8. Advantages

1.  **Separation of Concerns**: Security/Logging/Transaction logic is separated from Business Logic (AOP).
2.  **Performance**: Lazy Loading saves huge amounts of memory.

---

## 9. Disadvantages

1.  **Complexity**: Debugging Proxies is hard. The stack trace is full of `$Proxy0.invoke`.
2.  **Self-Invocation Issue**:
    ```java
    @Transactional
    public void a() {
        b(); // Calls 'this.b()'. BYPASSES THE PROXY! Transaction on b() is ignored.
    }
    ```
    *   **Fix**: Inject `self` or use `AopContext.currentProxy()`.

---

## 14. Interview Questions

### Basic
1.  **What is Lazy Loading?** (Loading data only when requested. Proxy pattern).
2.  **Difference between Proxy and Decorator?** (Proxy controls access. Decorator adds features. Proxy can instantiate the object later. Decorator takes the object in constructor).

### Intermediate
3.  **Why does `@Transactional` not work if I call a private method?** (Proxies cannot override private methods. Only public methods are intercepted).
4.  **What is CGLIB?** (Bytecode manipulation library used to subclass classes for proxying).

### Advanced
5.  **Explain the "Self-Invocation" problem in Spring AOP.** (Calling a method within the same class bypasses the proxy wrapper).
6.  **How does Hibernate use Proxies?** (When you call `load(User.class, 1)`, it returns a Proxy with ID=1. It only hits the DB when you call `user.getName()`).

---

## 16. Summary & Architect Takeaways

*   **You use Proxies every day**: Spring, Hibernate, Mockito.
*   **Performance Warning**: Reflection (used in Dynamic Proxies) has a cost. Don't proxy methods called in a tight loop (millions of times/sec).
*   **Self-Invocation**: Be very aware that internal method calls bypass the proxy logic (Transactions/Security).
