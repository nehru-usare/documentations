# MVC Pattern in Spring

> **Part 5: Patterns Inside Spring**  
> **Status:** The Web Standard

---

## 1. The MVC Pattern
*   **Model**: The Data (POJO).
*   **View**: The Presentation (JSP, Thymeleaf, JSON).
*   **Controller**: The Logic (Handles request, updates Model, selects View).

---

## 2. Spring MVC Flow (Front Controller)
Spring implements MVC using the **Front Controller** Pattern (`DispatcherServlet`).

1.  **Request** hits `DispatcherServlet`.
2.  **HandlerMapping**: Looks up which `@Controller` method matches the URL.
3.  **Controller**: Executes logic. Returns "Model" (Data) and "View Name".
4.  **ViewResolver**: Converts "View Name" (e.g., "home") to physical file (`/WEB-INF/home.jsp`).
5.  **Response**: Rendered HTML sent to user.

---

## 3. REST APIs (`@RestController`)
Modern Spring uses `@RestController`.
*   It combines `@Controller` and `@ResponseBody`.
*   **Model**: The Object returned by the method.
*   **View**: Serialized JSON (Jackson).
*   **ViewResolver**: `HttpMessageConverter` (converts Object -> JSON).

---

## 4. Architect Takeaway
*   **Keep Controllers Thin**: Controllers should only handle HTTP concerns (Params, Headers). They should delegate ALL logic to Services.
*   **Separation**: Don't return Entities from Controllers. Return DTOs (Data Transfer Objects).
