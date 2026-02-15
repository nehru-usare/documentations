# Factory Pattern in Spring

> **Part 5: Patterns Inside Spring**  
> **Status:** The Core of IOC

---

## 1. The Ultimate Factory: `ApplicationContext`
Spring's IoC Container (`ApplicationContext`) is essentially a giant **Factory**.
*   **Traditional Factory**: `MyService s = ServiceFactory.create("prod");`
*   **Spring Factory**: `MyService s = context.getBean(MyService.class);`

It handles:
1.  **Instantiation** (`new MyService()`).
2.  **Configuration** (Setting properties).
3.  **Assembly** (Injecting dependencies).
4.  **Lifecycle** (`@PostConstruct`).

---

## 2. The `FactoryBean` Interface
Sometimes, creating an object is too complex for a standard XML/Annotation definition.
*   **Scenario**: Creating a Hibernate `SessionFactory`. You typically need to configure DataSource, Dialect, Scanning Paths, etc.
*   **Solution**: Implement `FactoryBean<T>`.

```java
import org.springframework.beans.factory.FactoryBean;

public class MyComplexServiceFactory implements FactoryBean<ComplexService> {
    
    @Override
    public ComplexService getObject() {
        ComplexService s = new ComplexService();
        s.setConfiguration(loadHeavyConfig()); // Complex logic here
        s.init();
        return s;
    }

    @Override
    public Class<?> getObjectType() {
        return ComplexService.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

### Usage
```xml
<bean id="myService" class="com.example.MyComplexServiceFactory"/>
```
*   When you ask for `myService`, Spring sees it implements `FactoryBean`.
*   It calls `getObject()` and gives you the **Result**, not the Factory itself.
*   If you want the factory, use `&myService`.

---

## 3. Scope as Factory Logic
The Factory pattern in Spring is smart enough to handle Scopes.
*   **Singleton**: The Factory checks a cache. If exists, return it. Else create, cache, return.
*   **Prototype**: The Factory executes `new` every time.

---

## 4. Architect Takeaway
*   **Don't write manual Factories**: In Spring, let the Container be the Factory.
*   **Use `@Configuration`**: This is the modern Factory Method pattern.
    ```java
    @Configuration
    class AppConfig {
        @Bean // <--- This is a Factory Method!
        public MyService myService() {
            return new MyService(dependency());
        }
    }
    ```
