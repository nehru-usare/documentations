# Chapter 10: Exception Handling and Error Strategies

## 0. Learning Objectives

- **🟢 Beginner**: Understand the default Spring Boot error page and basic use of `@ResponseStatus`.
- **🟡 Professional**: Master `@ControllerAdvice`, `@ExceptionHandler`, and the use of custom error DTOs.
- **🔴 Architect**: Deep dive into the `HandlerExceptionResolver` chain, implement **RFC 7807 (Problem Details)**, and design a centralized, multi-tier error handling architecture for microservices.

---

## 1. Why This Topic Exists

### Real-World Business Problem
When something goes wrong in your code (Database is down, User is missing, API timeout), you shouldn't return a 500-line Java stack trace to the user. It's illegible for humans and a security risk for the enterprise.

### Technical Limitations Solved
- **Failure Isolation**: Proper exception handling ensures that a single failed component doesn't crash the entire request or cascade into other services.
- **Client Predictability**: It allows API clients (Mobile/UI) to receive structured, machine-readable error messages so they can show the correct "Try Again" or "Login" UI.

---

## 2. Big Picture Architecture View

Exception handling in Spring MVC is an **Aspect-Oriented** concern. It intercepts the normal request flow when an error is thrown.

### Interaction with Other Modules
- **Web Layer**: It uses `HandlerExceptionResolver` to map Java exceptions to HTTP status codes.
- **Observability**: Error handlers are the perfect place to log **Correlation IDs** and metrics for SRE monitoring.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Exception**: An event that disrupts the normal flow of the program.
- **Whitelabel Error Page**: The default gray-and-white page Spring Boot shows when something breaks.

### Simple Explanation
Think of a **Hospital Reception**.
- If you arrive with a **Broken Arm** (Bad Request), the receptionist gives you a form to fill (Error 400).
- If the hospital is **Full** (Service Unavailable), they tell you to go elsewhere (Error 503).
- Without a Receptionist (Exception Handler), the hospital would just be a building with people screaming in the hallways (Unstructured stack traces).

### Minimal Working Example
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### @ControllerAdvice: The Global Interceptor
Don't put try-catch blocks in your controllers. Use a global handler.
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("USER_NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```

### HandlerExceptionResolver Chain
Spring iterates through these until it finds one that can handle the exception:
1. `ExceptionHandlerExceptionResolver` (Handles `@ExceptionHandler`).
2. `ResponseStatusExceptionResolver` (Handles `@ResponseStatus`).
3. `DefaultHandlerExceptionResolver` (Standard internal Spring exceptions).

---

## 5. Internal Mechanics (🔴 Advanced Level)

### RFC 7807: Problem Details
Modern APIs should follow the RFC 7807 standard. Spring Boot 3+ provides the `ProblemDetail` class to make this easy.
- **Properties**: `type`, `title`, `status`, `detail`, `instance`.
- **Why?**: It provides a standard key-value structure that every industrial-grade client library understands.

### The `/error` Endpoint Mechanics
When an exception isn't caught by any resolver, Spring MVC forwards the request to `/error`.
- **`BasicErrorController`**: This is the default bean that handles the `/error` path.
- **`ErrorAttributes`**: The component that extracts the timestamp, status, and message from the request context.

---

## 6. Under the Hood

### Exception Translation
In Spring Data, the `@Repository` annotation triggers a **`PersistenceExceptionTranslationPostProcessor`**. 
- **The Magic**: This automatically converts low-level database-specific exceptions (like `SQLException` from Hibernate) into Spring's unified **`DataAccessException`** hierarchy.

---

## 7. Real-World Use Cases

- **E-Commerce Checkout**: Converting a "Payment Declined" exception into a polite 402 Payment Required response with a link to the "Update Card" page.
- **Security Triage**: Capturing all `AccessDeniedException`s and returning a 403 while logging the attempt for the security team.

---

## 8. Production & Performance Considerations

- **Stack Trace Cost**: Generating a stack trace is expensive for the JVM. 
  - **Architect Tip**: For high-volume business exceptions (like `InvalidInputException`), override the `fillInStackTrace()` method to return `this` (making it a "No-Stack" exception) to save CPU cycles.
- **Log Levels**: Don't log `WARN` or `ERROR` for every 404 User Not Found. This will bloat your logs. Use `INFO` or `DEBUG` for user journey errors and `ERROR` only for system failures.

---

## 9. Architect-Level Best Practices

- **Never Swallow Exceptions**: If you catch it, log it (or re-throw it). A silent failure is a production nightmare.
- **Include a Trace ID**: Always include the W3C `traceparent` or Sleuth `trace-id` in the error response. This allows support to find the exact logs in seconds.
- **Layered Exceptions**: Define a `BaseApiException` for your project and have all functional exceptions extend it.

---

## 10. Common Mistakes & Anti-Patterns

- **Returning Stack Traces to Clients**: #1 Security mistake. Hackers can use the trace to see your library versions and finding vulnerabilities.
- **Using Exceptions for Control Flow**: Don't use `throw new UserFoundException()` to return a user. Use a normal `return`. Exceptions are for... **Exceptional** cases.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`server.error.include-stacktrace=always`**: Enable this ONLY in your `dev` profile to see traces in the JSON response.
- **`HandlerExceptionResolver` Trace**: Enable trace logs for `org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver` to see why a specific handler wasn't picked.

---

## 12. Comparisons

### @ControllerAdvice vs. @ResponseStatus
| Feature | @ControllerAdvice | @ResponseStatus |
| :--- | :--- | :--- |
| **Centralization** | High (one place for all) | Low (scattered on classes) |
| **Logic** | Full Java logic allowed | Constant status only |
| **Response Body** | Custom DTOs | Default body only |
| **Recommendation** | **Architect Standard** | OK for simple internal APIs |

---

## 13. Interview Questions

### 🟢 Basic
1. What is the purpose of `@ControllerAdvice`?
2. What is the difference between a Checked and Unchecked exception?

### 🟡 Intermediate
1. How do you return a custom JSON body when an exception occurs?
2. What is the benefit of the `ProblemDetail` class in Spring Boot 3?

### 🔴 Advanced
1. Describe the internal chain of `HandlerExceptionResolver`.
2. How does the `BasicErrorController` work in Spring Boot?

### 🔥 Tricky
1. If an exception occurs inside a `Filter` (before it reaches the DispatcherServlet), will `@ControllerAdvice` catch it? (No. Filters are outside the MVC context).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You are building a system where 50 microservices need a consistent error format. How do you implement this? (Create a shared `starter-error-handler` library with a common `@ControllerAdvice`).
2. **Performance**: Your app is processing 1 million small validation errors per minute. The CPU is at 90%. How do you optimize the exception handling? (Disable stack trace generation for validation exceptions).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Exception Handling is **Customer Service** for your API.
- **Architect Mindset**: Expect failure. Design the error path as carefully as the success path.
- **Production Reminder**: Never reveal implementation details in error messages. "Something went wrong" is better than a Hibernate query dump.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 2: Chapter 10**
