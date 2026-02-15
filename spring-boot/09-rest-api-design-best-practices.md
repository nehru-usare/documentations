# Chapter 9: REST API Design Best Practices

## 0. Learning Objectives

- **🟢 Beginner**: Understand the 4 main HTTP methods (GET, POST, PUT, DELETE) and the basics of resource-based URLs.
- **🟡 Professional**: Master status code selection, HATEOAS, API versioning strategies, and idempotent operations.
- **🔴 Architect**: Design for cross-cutting concerns like pagination, filtering, sorting, "Richardson Maturity Model" level 3, and the design of long-running asynchronous API patterns.

---

## 1. Why This Topic Exists

### Real-World Business Problem
When multiple teams (Mobile, Frontend, Third-party partners) use your API, inconsistency is a disaster. If one team uses `/getUsers` and another uses `/api/v1/user-list`, the cognitive load increases, and integration becomes brittle.

### Technical Limitations Solved
- **Predictability**: REST provides a standardized way to interact with a system, meaning developers skip the "How do I use this?" phase.
- **Cacheability**: Proper use of HTTP GET and ETag headers allows the internet's infrastructure (CDNs, Bowsers) to cache your data automatically, saving your servers from unnecessary load.

---

## 2. Big Picture Architecture View

REST is an **Architectural Style**, not a protocol. In the Spring ecosystem, it sits between your internal business Domain and the outside Web.

### Interaction with Other Modules
- **HATEOAS**: Allows the API to provide "Navigation Links" back to the client.
- **Security**: Dictates how tokens (JWT/OAuth) are passed via headers (`Authorization: Bearer ...`).

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Resource**: An object (like a User, a Product, or an Invoice) that has a unique URL.
- **HTTP Method**: The "Verb" (Action) you perform on that resource.

### Simple Explanation
Think of a **Library**.
- **GET `/books`**: List all books.
- **GET `/books/123`**: View details of a specific book.
- **POST `/books`**: Donate a new book to the library.
- **PUT `/books/123`**: Update the information for a book.
- **DELETE `/books/123`**: Remove a torn book from the catalog.

### Minimal Working Example
```java
// A resource-based endpoint
@GetMapping("/users/{id}")
public UserDto getUser(@PathVariable Long id) {
    return userService.findById(id);
}
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Status Code Selection
- **200 OK**: Success.
- **201 Created**: Successful POST / creation.
- **204 No Content**: Successful DELETE.
- **400 Bad Request**: Client-side validation fail.
- **401 Unauthorized**: No login.
- **403 Forbidden**: Logged in, but no permission for this specific resource.
- **404 Not Found**: Resource doesn't exist.

### Idempotency
- **GET, PUT, DELETE** should be idempotent. (Repeating the request 10 times has the same effect as once).
- **POST** is NOT idempotent. (Repeating it creates 10 objects).

### API Versioning
1. **URI Versioning**: `/api/v1/users` (Most popular, highly visible).
2. **Header Versioning**: `X-API-Version: 2` (Cleaner URLs but harder to debug in a browser).

---

## 5. Internal Mechanics (🔴 Advanced Level)

### Richardson Maturity Model
- **Level 0**: POX (Plain Old XML) over HTTP. One URL for everything.
- **Level 1**: Resources. Different URLs for different things.
- **Level 2**: HTTP Verbs. Correct use of GET/POST/PUT.
- **Level 3**: **HATEOAS**. The API returns links advising the client on what it can do next.

### Filtering, Sorting, and Pagination
Standardizing these across an enterprise is critical.
- **Pagination**: Use `page` and `size` parameters. Spring Data handles this via `Pageable`.
- **Filtering**: Use Query Parameters: `/users?role=admin&status=active`.
- **Sorting**: `/users?sort=name,desc`.

---

## 6. Under the Hood

### Spring HATEOAS Support
Spring Boot provides a starter for HATEOAS. It wraps your DTOs in an `EntityModel` which allows you to add `Links`.
- **Internal**: It uses a `LinkDiscoverer` to help clients traverse your API without hardcoding complex URL paths.

---

## 7. Real-World Use Cases

- **Public Developer Platform**: An API like Stripe or GitHub that must be perfectly consistent to maintain developer trust.
- **Microservices Backbone**: Reliable communication between a "Checkout Service" and an "Inventory Service".

---

## 8. Production & Performance Considerations

- **Partial Updates (PATCH)**: Instead of sending the whole 50-field User object for a name change, use PATCH to only send the `name` field. This saves bandwidth and reduces DB write contention.
- **ETags & Conditional GETs**: The server sends a hash of the resource. The client sends it back in the next request. If the data hasn't changed, the server returns **304 Not Modified**, saving the body transfer completely.

---

## 9. Architect-Level Best Practices

- **Plural Nouns**: Use `/users`, not `/user`.
- **Nouns, not Verbs**: `/orders/{id}/cancel` is a verb. Better: `POST /orders/{id}/cancellation` or `PATCH /orders/{id} { "status": "CANCELLED" }`.
- **Consistency**: If one endpoint uses snake_case, they ALL must. If one uses `ISO-8601` for dates, they ALL must.

---

## 10. Common Mistakes & Anti-Patterns

- **Tunneling everything through POST**: Using POST for everything because it's "easier". This breaks caching and predictability.
- **Ignoring 4xx and returning 200 with error body**: Returning `200 OK` with `{ "error": "User not found" }` is a crime against REST. Use `404`.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **cURL / Postman**: The ultimate tools for manual verification.
- **OpenAPI (Swagger)**: Automatically generates documentation and an interactive UI for your API.
- **Trace Logs**: Log the Request Path, Method, and Status code for every single call in production.

---

## 12. Comparisons

### PUT vs. PATCH
| Feature | PUT | PATCH |
| :--- | :--- | :--- |
| **Replacement** | Full resource replacement | Partial update |
| **Idempotent** | Yes | Generally no (depends on logic) |
| **Payload Size** | Larger | Smaller |
| **Implementation** | Easier | Slightly more complex |

---

## 13. Interview Questions

### 🟢 Basic
1. What are the 4 main HTTP verbs?
2. What is the difference between 401 and 403 status codes?

### 🟡 Intermediate
1. What does it mean for an operation to be "Idempotent"?
2. Explain the benefits of API Versioning.

### 🔴 Advanced
1. Describe the Richardson Maturity Model.
2. How do you implement HATEOAS in a Spring Boot application?

### 🔥 Tricky
1. Is it okay to send a body in a GET request? (Technically possible in HTTP spec, but forbidden by most servers/proxies and highly discouraged).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You are building a "Search" API. It has 50 possible filters. If you put them all in the URL query string, the URL becomes too long (>2k chars). What do you do? (Use `POST /search` with a JSON body, acknowledging it is a "Functional POST" rather than a RESTful create).
2. **Performance Optimization**: Your mobile app needs to load a list of 1,000 products. The JSON is 2MB. How do you optimize the API for the mobile client? (Implement Pagination, Field Filtering, and enable Gzip compression).

---

## 15. Summary & Key Takeaways

- **Core Insight**: A REST API is a **Language**. If the language is inconsistent, the conversation fails.
- **Architect Mindset**: Documentation is part of the API. If it's not documented in Swagger/OpenAPI, it doesn't exist.
- **Production Reminder**: Use ETag headers for your read-heavy resources. Your cache is your best friend in production.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 2: Chapter 9**
