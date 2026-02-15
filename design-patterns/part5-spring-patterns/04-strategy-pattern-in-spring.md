# Strategy Pattern in Spring

> **Part 5: Patterns Inside Spring**  
> **Status:** The Foundation of DI

---

## 1. Dependency Injection = Strategy
Dependency Injection (DI) is essentially a framework-managed **Strategy Pattern**.

*   **Strategy**: Interface (`StorageService`).
*   **ConcreteStrategies**: `S3StorageService`, `LocalStorageService`.
*   **Context**: `FileController`.

In standard Java, you use `controller.setStrategy(new S3...)`.
In Spring, you use `@Autowired`.

---

## 2. Configuration through Profiles
Spring Profiles allow you to swap Strategies at boot time.

```java
public interface EmailSender { void send(String msg); }

@Profile("dev")
@Service
public class MockEmailSender implements EmailSender {
    public void send(String msg) { System.out.println("LOG: " + msg); }
}

@Profile("prod")
@Service
public class SmtpEmailSender implements EmailSender {
    public void send(String msg) { smtp.send(msg); }
}
```

*   **Dev Environment**: Spring injects `MockEmailSender`.
*   **Prod Environment**: Spring injects `SmtpEmailSender`.
*   The `Client` code **never changes**.

---

## 3. `ResourceLoader` Strategy
Spring's `ResourceLoader` uses Strategy to load files.
*   `ctx.getResource("classpath:config.xml")` -> `ClassPathResource`
*   `ctx.getResource("file:/etc/config.xml")` -> `FileSystemResource`
*   `ctx.getResource("http://api.com/config.xml")` -> `UrlResource`

The `Resource` interface abstracts away the underlying storage implementation.

---

## 4. Architect Takeaway
*   **Embrace Interfaces**: Always inject Interfaces, not Concrete Classes. This enables the Strategy pattern (and Proxies).
*   **Map Injection**: You can inject ALL strategies into a Map.
    ```java
    @Autowired
    private Map<String, PaymentGateway> gateways;
    
    public void pay(String type) {
        gateways.get(type).process();
    }
    ```
    This turns the Bean ID into a Key for runtime selection.
