# Chapter 8: Spring MVC Internals

## 0. Learning Objectives

- **🟢 Beginner**: Understand the request-response cycle and the role of `@Controller` and `@RequestMapping`.
- **🟡 Professional**: Master the `DispatcherServlet` flow, understand View Resolvers, and learn how to use `Interceptor`s for cross-cutting web concerns.
- **🔴 Architect**: Deep dive into the `HandlerMapping` and `HandlerAdapter` architecture, understand the `RequestBodyadvice` and `ResponseBodyAdvice` hooks, and optimize the Servlet container threading model for high-traffic APIs.

---

## 1. Why This Topic Exists

### Real-World Business Problem
When building a web application, you need a way to route an HTTP request (URL, Method, Headers) to a specific piece of Java code. Without a framework, you would have to write hundreds of "if-else" statements inside a raw Java `Servlet`.

### Technical Limitations Solved
- **The "Dispatcher" Pattern**: Spring MVC provides a single entry point (The DispatcherServlet) that manages the complex task of routing, data binding, and view rendering.
- **Support for Multiple Formats**: It allows the same code to return HTML, JSON, or XML based on the client's request headers (Content Negotiation).

---

## 2. Big Picture Architecture View

Spring MVC is built on top of the standard **Java Servlet API**.

### Interaction with Other Modules
- **Core Container**: MVC controllers are just Beans managed by the container.
- **Security**: Spring Security sits as a "Filter" in front of the MVC dispatcher.
- **Validation**: JSR-303 validation is triggered during the data-binding phase of MVC.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **MVC (Model-View-Controller)**: A design pattern that separates Data (Model), User Interface (View), and Logic (Controller).
- **Endpoint**: A specific URL path that handles requests (e.g., `/api/users`).

### Simple Explanation
Think of a **Post Office**. 
1. The **DispatcherServlet** is the "Front Desk". It receives all mail.
2. The **HandlerMapping** is the "Sorting Machine". it looks at the address (URL) and decides which department should handle it.
3. The **Controller** is the "Department Worker". They process the request and put the answer in an envelope (the Model).
4. The **ViewResolver** is the "Output Format". It decides if the answer should be sent as a Letter (HTML) or a Data-package (JSON).

### Minimal Working Example
```java
@RestController
@RequestMapping("/api")
public class HelloController {
    @GetMapping("/greet")
    public String sayHello() {
        return "Hello World!";
    }
}
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### The DispatcherServlet Flow
1. **Receive Request**: HTTP request arrives at `DispatcherServlet`.
2. **Find Handler**: `HandlerMapping` finds the controller method.
3. **Execute Interceptors**: `HandlerInterceptor.preHandle()` runs.
4. **Invoke Controller**: `HandlerAdapter` calls the controller method.
5. **Handle result**: Controller returns a `ModelAndView` or raw JSON.
6. **View Rendering**: If HTML, the `ViewResolver` resolves the template.
7. **Complete**: `afterCompletion()` runs and the response is sent.

### HandlerInterceptors
Perfect for logging, auth checks, or performance timing across all web requests.
```java
public class MyInterceptor implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request, ...) {
        // Run before controller
        return true; 
    }
}
```

---

## 5. Internal Mechanics (🔴 Advanced Level)

### HandlerMapping vs. HandlerAdapter
- **`HandlerMapping`**: The "Router". It maps a request to a **`HandlerMethod`**.
- **`HandlerAdapter`**: The "Invoker". Since handlers can be anything (Methods, Classes, even older Servlet types), the adapter provides a uniform way to call them.

### Data Binding and Argument Resolvers
How does `@PathVariable` or `@RequestBody` work?
- Spring uses **`HandlerMethodArgumentResolver`**. There are about 30+ built-in resolvers. 
- For `@RequestBody`, it uses an **`HttpMessageConverter`** (like Jackson) to read the input stream.

### Request/Response Advice
- **`RequestBodyAdvice`**: Allows you to intercept and modify the JSON body *before* it reaches the controller (e.g., for decryption).
- **`ResponseBodyAdvice`**: Allows you to modify the result *after* the controller finishes but before it's sent (e.g., adding common metadata to every JSON response).

---

## 6. Under the Hood

### Autoconfiguration of MVC
Spring Boot automatically registers the `DispatcherServlet` at `/` and configures a `ContentNegotiationManager`.
- It automatically adds Jackson (for JSON) if the library is on the classpath.
- It configures `StaticResourceHttpRequestHandler` to serve files from `/static` or `/public`.

---

## 7. Real-World Use Cases

- **Versioned APIs**: Using custom `HandlerMapping` to route `/v1/users` and `/v2/users` to different controller versions based on a header.
- **Server-Side Rendering (SSR)**: Using Thymeleaf + Spring MVC for high-SEO e-commerce sites.

---

## 8. Production & Performance Considerations

- **Thread-Per-Request**: In the standard MVC model (Tomcat), each request blocks a thread. Under extremely high load (10k+ concurrent requests), you will hit a **Thread Pool Exhaustion** bottleneck.
- **Object Serialization**: Large JSON objects take CPU and memory to serialize. Use **`MappingJackson2HttpMessageConverter`** optimizations like "filtering" to only send necessary fields.

---

## 9. Architect-Level Best Practices

- **Avoid logic in Controllers**: Keep controllers slim. They should only handle HTTP concerns (mapping, status codes) and delegate to Services.
- **Use `@RestController` for APIs**: It automatically adds `@ResponseBody` to every method, making your code cleaner.
- **Content Negotiation**: Don't hardcode `.json` in your URLs. Let the client use the `Accept` header.

---

## 10. Common Mistakes & Anti-Patterns

- **Returning Entities Directly**: Never return your JPA `@Entity` classes from a controller. Use **DTOs** (Data Transfer Objects). Returning entities leads to security leaks (lazy loading sensitive fields) and tight coupling with the DB.
- **Heavy Logic in Interceptors**: Since interceptors run for every request, a slow DB call in an interceptor will kill your entire application's performance.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`logging.level.org.springframework.web=DEBUG`**: Shows the exact mapping decisions.
- **404 Not Found**: Check the "Conditions" report to see if your controller bean was even created.
- **415 Unsupported Media Type**: Usually means you sent JSON but didn't set `Content-Type: application/json` or your POJO is missing a default constructor for Jackson.

---

## 12. Comparisons

### MVC vs. WebFlux (High Level)
| Feature | Spring WebMVC (Servlet) | Spring WebFlux (Reactive) |
| :--- | :--- | :--- |
| **I/O Model** | Blocking | Non-blocking |
| **Concurrency** | Thread-per-request | Event Loop |
| **Dependencies** | Standard Java/JPA | Reactive Drviers/R2DBC |
| **Best For** | Standard Apps | High-scale / Streaming |

---

## 13. Interview Questions

### 🟢 Basic
1. What is the role of `DispatcherServlet`?
2. Mention the difference between `@Controller` and `@RestController`.

### 🟡 Intermediate
1. Describe the journey of a request in Spring MVC.
2. What is a `ViewResolver`?

### 🔴 Advanced
1. How does Spring MVC find the correct `HandlerMapping` among many?
2. What is an `HttpMessageConverter` and how do you add a custom one?

### 🔥 Tricky
1. If two controllers have the exact same `@RequestMapping` value, what happens during startup? (Startup fails with an `IllegalStateException` due to ambiguous mapping).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You are building an API that needs to support both JSON and XML. How do you implement this without changing the controller logic? (Content Negotiation).
2. **Production Incident**: Your API is returning 200 OK, but the body is empty, even though the service is returning data. Where do you look? (Check `ResponseBodyAdvice` or Jackson's `@JsonIgnore` / Getter access).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Spring MVC is the **Translator** between HTTP and Java.
- **Architect Mindset**: Design for **Content Independence**. Your business logic shouldn't care if the caller wants JSON or HTML.
- **Production Reminder**: Monitor your **Servlet Thread Pool**. If `tomcat_threads_busy` is always high, you have a blocking-I/O bottleneck.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 2: Chapter 8**
