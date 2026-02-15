# Applied Design Patterns: Singleton, Factory, and Builder

> **Part 1: Language Foundations**  
> **Level:** Principal Engineer  
> **Status:** Constructed

---

## 0. Learning Objectives

*   **Developer**: Why `new Singleton()` is a compile error.
*   **Senior**: Implementing a Thread-Safe Singleton without locks (Enum).
*   **Architect**: Using the Builder Pattern for Immutable DTOs.

---

## 1. The Singleton Pattern

**Intent**: Ensure a class has only one instance and provide a global access point.

### 1.1 The "Enum" Singleton (Best Practice)
Effective Java recommends this. It handles Serialization and Thread-Safety automatically.
```java
public enum Database {
    INSTANCE;
    
    public void connect() { ... }
}
// Usage: Database.INSTANCE.connect();
```

### 1.2 The "Double-Checked Locking" (Lazy Loading)
Use this ONLY if you need lazy initialization and cannot use Enum.
```java
public class LazySingleton {
    private static volatile LazySingleton instance; // Must be volatile!

    private LazySingleton() {} // Private Constructor

    public static LazySingleton getInstance() {
        if (instance == null) {
            synchronized (LazySingleton.class) {
                if (instance == null) {
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}
```

---

## 2. The Factory Pattern

**Intent**: Encapsulate object creation logic.

### 2.1 Static Factory Methods
Replace constructors with static methods with names.
*   **Constructor**: `new User(true, false)` -> Unclear.
*   **Factory**: `User.createAdmin()` -> Clear.

```java
public class User {
    private User(String type) { ... }
    
    public static User createAdmin() { return new User("Admin"); }
    public static User createGuest() { return new User("Guest"); }
}
```

---

## 3. The Builder Pattern

**Intent**: Construct complex objects step-by-step.
**Problem**: "Telescoping Constructors" (`new User("name", null, null, 18, ...)`).

### 3.1 Implementation
```java
public class ServerConfig {
    private final String host;
    private final int port;

    private ServerConfig(Builder b) { this.host = b.host; this.port = b.port; }

    public static class Builder {
        private String host;
        private int port = 80; // Default

        public Builder host(String h) { this.host = h; return this; }
        public Builder port(int p) { this.port = p; return this; }
        
        public ServerConfig build() { return new ServerConfig(this); }
    }
}
// Usage: new ServerConfig.Builder().host("localhost").port(8080).build();
```

---

## 4. The Strategy Pattern

**Intent**: Define a family of algorithms and make them interchangeable.
**Modern Java**: Use **Lambdas** or **Enums** instead of heavy classes.

```java
// Logic
public void process(List<Integer> numbers, final Predicate<Integer> selector) {
    for (int n : numbers) {
        if (selector.test(n)) print(n);
    }
}

// Usage changes behavior cleanly
process(nums, n -> n % 2 == 0); // Strategy: Evens
process(nums, n -> n > 100);    // Strategy: Big numbers
```

---

## 5. Summary & Architect Takeaways

1.  **Singleton**: Default to `Enum`. It's un-hackable (Reflection-proof).
2.  **Builder**: Mandatory for DTOs with > 4 fields. Keep the class Immutable.
3.  **Factory**: Use `List.of()` (Factory) instead of `new ArrayList() {{ add() }}`.

---
*End of Part 1.*
