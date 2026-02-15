# Optional: The End of NullPointerExceptions?

> **Part 5: Modern Java**  
> **Level:** Principal Engineer  
> **Status:** Present

---

## 0. Learning Objectives

*   **Developer**: Why `null` checks are bad.
*   **Senior**: When NOT to use Optional (Fields, Parameters).
*   **Architect**: Designing APIs that return Optional.

---

## 1. The Design Intent

To provide a clear **return type** for methods that might not return a value.
*   **Before**: `public User find(id)` returns `null`? Or throws Exception? Unclear.
*   **After**: `public Optional<User> find(id)`. The signature screams "This might be empty".

---

## 2. API Usage

### 2.1 Creation
*   `Optional.of("value")`: Throws NPE if value is null. Use when you know it's present.
*   `Optional.ofNullable(val)`: Handles null gracefully. Returns `Optional.empty()`.

### 2.2 Consumption
*   **Bad**: `if (opt.isPresent()) get()`. This is just a verbose null check.
*   **Good**: `opt.ifPresent(user -> print(user))`.
*   **Better**: `User u = opt.orElseThrow(() -> new NotFoundException())`.

---

## 3. Anti-Patterns (When NOT to use)

1.  **As Field**: `class User { Optional<Address> address; }`.
    *   **Cons**: Not Serializable. Extra memory overhead (Optional wrapper object).
    *   **Fix**: Use `null` for fields, return `Optional` in getters.
2.  **As Parameter**: `void method(Optional<String> s)`.
    *   **Cons**: Callers must wrap `Optional.of(s)`. Annoying.
    *   **Fix**: Overload the method. `method(String s)` and `method()`.

---

## 4. Summary & Architect Takeaways

1.  **Return Types Only**: Use Optional primarily for return types.
2.  **Stream Integration**: `stream.findFirst()` returns `Optional`. Perfect synergy.
3.  **Primitive Optionals**: `OptionalInt` exists. Use it to avoid boxing.

---
*Next Chapter: The Module System (JPMS).*
