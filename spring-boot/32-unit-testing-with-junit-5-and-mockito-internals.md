# Chapter 32: Unit Testing with JUnit 5 and Mockito Internals

## 0. Learning Objectives

- **🟢 Beginner**: Understand the difference between Unit Testing and Integration Testing, and how to write a simple test using `@Test`.
- **🟡 Professional**: Master the use of **Mockito** (`@Mock`, `@InjectMocks`, `verify`), AssertJ for readable assertions, and JUnit 5 lifecycle methods (`@BeforeEach`, `@AfterEach`).
- **🔴 Architect**: Deep dive into the `Mockito` proxy generation internals (ByteBuddy), understand the "Stubbing" vs. "Mocking" philosophy, and design test suites that are independent of the Spring Context for maximum execution speed.

---

## 1. Why This Topic Exists

### Real-World Business Problem
Software changes. Every time you fix a bug or add a feature, you risk breaking something else. You can't manually test 1,000 features every day. Automated tests are your **Safety Net**. They allow you to refactor code with confidence and ship software faster.

### Technical Limitations Solved
- **Isolation**: Unit tests allow you to test a single Java class in total isolation. You don't need a database, a network, or even the Spring Framework to run them. 
- **Feedback Loop**: A unit test takes milliseconds to run. If you break something, you know it in 1 second, not 10 minutes (at the end of a build).

---

## 2. Big Picture Architecture View

Unit testing happens **Outside** the Spring Context. Ideally, the Spring container should not even be started.

### Interaction with Other Modules
- **JUnit 5 (Jupiter)**: The core test runner.
- **Mockito**: The library that creates "Fakes" of your dependencies.
- **AssertJ**: The fluent API used to verify results.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Unit**: The smallest piece of testable code (Usually a single Method).
- **Mock**: A "Fake" object that mimics a real dependency.
- **Assertion**: A statement that verifies if the result is correct (e.g., "Result should be 10").

### Simple Explanation
Think of a **Car Factory**.
- **Integration Test**: Testing the whole car on a race track. (Slow, expensive).
- **Unit Test**: Testing a single **Light Bulb** in a separate laboratory. You don't need the whole car; you just need a battery (Mock) to see if the bulb shines. If the bulb works in the lab, you know it's good.

### Minimal Working Example
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock UserRepository repository; // The "Fake" DB
    @InjectMocks UserService service; // The thing we are testing

    @Test
    void shouldFindUser() {
        // 1. Arrange
        when(repository.findById(1L)).thenReturn(Optional.of(new User("John")));

        // 2. Act
        User result = service.getUser(1L);

        // 3. Assert
        assertThat(result.getName()).isEqualTo("John");
    }
}
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Mockito Mastery
- **`@Mock`**: Creates the fake dependency.
- **`@InjectMocks`**: Automatically puts the mocks into the constructor of your service.
- **`verify(mock, times(1)).method()`**: Ensures that a specific method was actually called (important for "void" methods).

### AssertJ for Readability
Don't use `assertEquals(a, b)`. It's hard to read.
Use AssertJ:
`assertThat(list).hasSize(3).contains("Apple").doesNotContain("Banana");`

### Exception Testing
JUnit 5 makes this easy:
`assertThrows(UserNotFoundException.class, () -> service.getUser(99L));`

---

## 5. Internal Mechanics (🔴 Advanced Level)

### How Mockito Works (ByteBuddy)
When you call `@Mock`, Mockito uses **ByteBuddy** to generate a new class at runtime. 
- **Subclassing**: It creates a subclass of your `UserRepository`.
- **Method Interception**: Every method in this subclass is "Intercepted." When you call `findByEmail()`, Mockito's internal logic looks at its "Stubbing Table" (the `when(...)` calls) to decide what to return.

### Stubbing vs. Mocking
- **Stub**: Providing a pre-defined answer to a call (State-based testing).
- **Mock**: Setting an expectation that a call will be made (Behavior-based testing).
**Architect Note**: Overusing `verify()` (Behavior testing) makes tests fragile. If you rename a private method or change internal logic, the test breaks even if the result is still correct. **Prefer Stubbing/Asserting state.**

---

## 6. Under the Hood

### JUnit 5 Extension Model
The `@ExtendWith(MockitoExtension.class)` is the "Hook" that allows Mockito to initialize `@Mock` before your tests run. It's much cleaner than the old `MockitoAnnotations.initMocks(this)` call from JUnit 4.

---

## 7. Real-World Use Cases

- **TDD (Test-Driven Development)**: Writing the test for a new "Discount Calculation" logic before writing the service itself.
- **Bug Regression**: When a bug is found, first write a unit test that "Reproduces" the bug (fails), then fix the code until the test passes.

---

## 8. Production & Performance Considerations

- **Execution Speed**: A professional project should have 5,000+ unit tests. They MUST run in under 2 minutes. 
  - **Best Practice**: Avoid `@SpringBootTest` in unit tests. It takes 5-10 seconds to start Spring, which is far too slow for a unit test.
- **Parallel Execution**: JUnit 5 supports running tests in parallel. Configure `junit.jupiter.execution.parallel.enabled=true` in your `junit-platform.properties` to use all your CPU cores.

---

## 9. Architect-Level Best Practices

- **The Testing Pyramid**: Your project should have 80% Unit Tests, 15% Integration Tests, and 5% E2E tests.
- **Clean Tests**: Treat your test code with the same respect as your production code. A messy test suite is a maintenance nightmare that people will eventually stop running.
- **Don't Mock what you don't own**: Only mock your own services and repositories. Don't mock third-party libraries (like `ArrayList` or `RestTemplate`). Use the real object if it's simple or use a "Fake Server" for integration tests.

---

## 10. Common Mistakes & Anti-Patterns

- **Mocking the Class Under Test**: Never mock the service you are actually testing (`@Spy` can be used, but sparingly).
- **Tests with Logic**: Putting `if` statements or `for` loops in tests. If your test has logic, who tests the test? Keep tests linear: **Arrange, Act, Assert.**

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`UnfinishedStubbingException`**: You called `when(repo.find())` but forgot to call `.thenReturn(...)`.
- **`NullPointerException`**: You forgot `@ExtendWith(MockitoExtension.class)`, so your `@Mock` objects were never initialized and are all `null`.

---

## 12. Comparisons

### Unit Testing vs. Integration Testing
| Feature | Unit Testing | Integration Testing |
| :--- | :--- | :--- |
| **Spring Context** | No | Yes (`@SpringBootTest`) |
| **Dependencies** | Mocked | Real/Testcontainers |
| **Speed** | Instant | Slow |
| **Goal** | Logic correctness | Wiring & Database |
| **Recommendation** | **Check 1 class** | **Check 1 flow** |

---

## 13. Interview Questions

### 🟢 Basic
1. What is the difference between `@Mock` and `@InjectMocks`?
2. What is the "AAA" pattern (Arrange, Act, Assert)?

### 🟡 Intermediate
1. How do you verify that a method was NEVER called in Mockito?
2. What is AssertJ and why is it preferred over JUnit assertions?

### 🔴 Advanced
1. Explain how Mockito creates a mock object at runtime using bytecode manipulation.
2. How do you test a private method? (Don't—test the public method that calls it).

### 🔥 Tricky
1. Can you mock a `static` method? (Yes, using `mockStatic`, but it's usually a sign of bad architecture).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You have a service that calls 5 different APIs. Your unit test is taking 30 seconds to run. What is likely wrong? (You are likely using `@SpringBootTest` or `@MockBean` which starts the whole Spring context. Switch to `@ExtendWith(MockitoExtension.class)` for a pure unit test).
2. **Performance**: Your project has 10,000 tests and the CI/CD build takes 1 hour. How do you optimize it? (1. Identify slow `@SpringBootTest` and convert to unit tests. 2. Enable JUnit 5 Parallel Execution. 3. Use a "Test Cache" if possible).

---

## 15. Summary & Key Takeaways

- **Core Insight**: A test is a **Living Specification**.
- **Architect Mindset**: High test coverage isn't enough; the tests must be fast and reliable. A "Flaky" test (passing 50% of the time) is worse than no test at all.
- **Production Reminder**: Tests are your insurance policy. You pay for them with development time, but they pay you back during every production deployment.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 7: Chapter 32**
