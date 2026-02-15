# Chapter 34: MockMvc and RestAssured for API Testing

## 0. Learning Objectives

- **🟢 Beginner**: Understand the purpose of `MockMvc` and how to perform a simple `GET` request in a test.
- **🟡 Professional**: Master the use of **JSONPath** for result verification, simulating different HTTP methods, and using **RestAssured** for end-to-end API testing against a real running server.
- **🔴 Architect**: Deep dive into the `MockMvc` request cycle internals, understand the "Filter Simulation" logic, and design a standardized **API contract testing** suite that ensures front-end compatibility.

---

## 1. Why This Topic Exists

### Real-World Business Problem
You've written a REST Controller. How do you know if it returns the right JSON structure? How do you know if it handles the "Page Not Found" case with a 404 status? Testing just the "Service" class isn't enough; you must test the **HTTP Layer** (Serialization, URL Mapping, Status Codes).

### Technical Limitations Solved
- **No Network Required**: `MockMvc` allows you to test your Controllers without actually opening a network port or starting a full Tomcat server. It "Simulates" the HTTP request inside the Spring container.
- **True End-to-End**: `RestAssured` goes one step further by calling your API over the real network, which is perfect for final verification before shipping.

---

## 2. Big Picture Architecture View

API testing sits at the very **Edge** of your application.

### Interaction with Other Modules
- **Spring MVC**: `MockMvc` calls the `DispatcherServlet` directly.
- **Jackson**: API tests verify that your Java objects are correctly converted to JSON.
- **Spring Security**: Crucial for testing if endpoints are blocked for unauthorized users.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **MockMvc**: A Spring-provided tool that mimics a web server.
- **JSONPath**: A query language (like SQL for JSON) used to find specific values in a response.
- **RestAssured**: An external library for writing clean, fluent API tests over real HTTP.

### Simple Explanation
Think of a **Drive-Thru Menu**.
- **MockMvc**: You are the manager of the restaurant. You stand *behind the counter* and pretend someone just ordered a burger. You see if the kitchen makes it correctly. No real cars are involved. (Fast).
- **RestAssured**: You are a real customer. You drive your car to the window, place an order, and see if the food you get is correct. (Real world).

### Minimal Working Example
```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired MockMvc mockMvc;

    @Test
    void shouldReturnUser() throws Exception {
        mockMvc.perform(get("/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("John"));
    }
}
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Fluency in MockMvc
- **Perform**: The action (`get`, `post`, `put`, `delete`).
- **Expect**: The verification (`status`, `header`, `content`, `jsonPath`).
- **JSONPath Tricks**:
  - `$.[0].id`: The ID of the first item in a list.
  - `$.length()`: Total number of items in the list.

### RestAssured (End-to-End)
When using `@SpringBootTest(webEnvironment = RANDOM_PORT)`, use RestAssured:
```java
given()
    .port(port)
    .contentType(ContentType.JSON)
when()
    .get("/api/orders")
then()
    .statusCode(200)
    .body("total", equalTo(100.0f));
```

---

## 5. Internal Mechanics (🔴 Advanced Level)

### MockMvc Request Lifecycle
1. `mockMvc.perform()` creates a `MockHttpServletRequest`.
2. It passes this request to the `DispatcherServlet`.
3. The `DispatcherServlet` goes through the standard mapping, interceptors, and controller logic.
4. The result is captured in a `MvcResult` object (including the `Handler`, `ModelAndView`, and `ResolvedException`).
**Architect Note**: Unlike a real server, `MockMvc` does NOT execute your Servlet Filters by default unless you configure it `.apply(SecurityMockMvcConfigurers.springSecurity())`.

### Testing File Uploads
You can simulate a multipart request easily:
```java
mockMvc.perform(multipart("/upload").file(new MockMultipartFile("file", "test.txt", "text/plain", "data".getBytes())))
       .andExpect(status().isCreated());
```

---

## 6. Under the Hood

### Serialization Validation
One of the most common production bugs is a "Missing Field" in JSON. API tests are your first line of defense. By asserting on the exact structure using `jsonPath`, you ensure that a developer hasn't accidentally renamed a field in a DTO.

---

## 7. Real-World Use Cases

- **Contract Enforcement**: Ensuring your API always returns `{"id": 1, "status": "ACTIVE"}` so the React/Angular frontend doesn't break.
- **Error Scenario Simulation**: Verifying that giving a negative ID to your API returns a `400 Bad Request` with a meaningful error message.

---

## 8. Production & Performance Considerations

- **Logging**: Use `.andDo(print())` during development to see the full request/response in the console. **Remove it for CI/CD** as it slows down the tests and pollutes the logs.
- **Parallelism**: RestAssured tests are slower because of network overhead. Run them in a separate "Integration Test" phase rather than the main "Unit Test" phase.

---

## 9. Architect-Level Best Practices

- **Use @WebMvcTest for Speed**: Dedicated controller tests are 10x faster than full `@SpringBootTest`.
- **Abstract Common Setup**: Create a `BaseApiTest` class that configures the Object Mapper, authentication, and headers to keep your tests clean.
- **Test the Content Type**: Always verify the `Content-Type: application/json` header. If your API accidentally returns plain text, the client-side parser will fail.

---

## 10. Common Mistakes & Anti-Patterns

- **Hardcoding IDs**: Don't check for `"id": 1`. Check for `"id": notNullValue()`. Database-generated IDs are unpredictable.
- **Ignoring the Body**: Only checking `status().isOk()` without checking the payload. An empty JSON `{}` also returns a 200 OK!

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`Handler not found`**: You likely forgot to include the Controller in your `@WebMvcTest` declaration.
- **`JsonPath Exception`**: The structure of your JSON doesn't match your query. Print the output using `.andDo(print())` to see the actual JSON.

---

## 12. Comparisons

### MockMvc vs. RestAssured
| Feature | MockMvc | RestAssured |
| :--- | :--- | :--- |
| **Port Usage** | No (Simulated) | Yes (Real Port) |
| **Speed** | Fast | Slower (Network) |
| **Environment** | Spring Context | Any Web Server |
| **Security** | Simulated Filters | Real Security Chain |
| **Recommendation** | **Layered Controller Test** | **End-to-End Flow Test** |

---

## 13. Interview Questions

### 🟢 Basic
1. What does `jsonPath` do in a MockMvc test?
2. How do you check if a response code is 404?

### 🟡 Intermediate
1. Difference between `@WebMvcTest` and `@SpringBootTest`?
2. How do you simulate a "POST" request with a JSON body in MockMvc?

### 🔴 Advanced
1. How does MockMvc simulate the DispatcherServlet request cycle?
2. What are the benefits of using RestAssured for contract testing?

### 🔥 Tricky
1. If your Controller uses `@Async`, will a standard MockMvc test pass? (No, you must use `MvcResult` and `mockMvc.perform(asyncDispatch(mvcResult))` to wait for the async part to finish).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You are building a "Public API" used by 1,000 external companies. How do you ensure you never accidentally break their integrations? (Implement **RestAssured Contract Tests**. These tests should run against every build and verify that all mandatory fields are present and the JSON structure is identical to the previous version).
2. **Performance**: Your "Integration Tests" take 10 minutes to run because they all start a full Tomcat server. How do you optimize? (Migrate most of those tests to `@WebMvcTest`. This allows you to test the HTTP mapping and logic without the overhead of starting the entire application).

---

## 15. Summary & Key Takeaways

- **Core Insight**: API tests are **Public Promises**.
- **Architect Mindset**: Don't just test the "Happy Path." Test the "Validation Failed," "Unauthorized," and "Server Error" paths.
- **Production Reminder**: A 200 OK doesn't mean the app works. Verify the data inside the JSON.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 7: Chapter 34**
