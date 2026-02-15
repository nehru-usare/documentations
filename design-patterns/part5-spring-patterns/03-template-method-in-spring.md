# Template Method in Spring

> **Part 5: Patterns Inside Spring**  
> **Status:** Boilerplate Killer

---

## 1. The `XXXTemplate` Pattern
Whenever you see a class ending in `Template` in Spring, it is the **Template Method Design Pattern**.
*   `JdbcTemplate`
*   `RestTemplate` (and `WebClient`)
*   `JmsTemplate`
*   `MongoTemplate`
*   `TransactionTemplate`

---

## 2. Example: `JdbcTemplate`

### Without Template (Boilerplate)
```java
Connection con = null;
PreparedStatement ps = null;
try {
    con = dataSource.getConnection(); // 1. Acquire
    ps = con.prepareStatement("SELECT..."); // 2. Prepare
    ResultSet rs = ps.executeQuery(); // 3. Execute
    while (rs.next()) { ... } // 4. Map
} catch (SQLException e) {
    // 5. Handle Exception
} finally {
    // 6. Close everything safely
    if (ps != null) ps.close(); 
    if (con != null) con.close();
}
```

### With `JdbcTemplate`
```java
jdbcTemplate.query("SELECT...", new RowMapper<User>() {
    @Override
    public User mapRow(ResultSet rs, int rowNum) {
        // Pure Business Logic
        return new User(rs.getString("name"));
    }
});
```

### How it works
`JdbcTemplate` implements the **Fixed Steps** (Open, Catch, Close).
It delegates the **Variable Step** (`mapRow`) to you via a Callback Interface.
*   *Note*: Technically, Spring uses **Callback** mechanism here, which is the functional equivalent of Template Method using Composition instead of Inheritance.

---

## 3. `TransactionTemplate`
Programmatic Transaction Management uses this.
```java
transactionTemplate.execute(status -> {
    // Your DB code here
    // If exception, Spring Rollbacks.
    return result;
});
```

---

## 4. Architect Takeaway
*   **Don't reinvent the wheel**. If you find yourself writing `try-catch-finally` blocks repeatedly for a resource, wrap it in a Template class.
