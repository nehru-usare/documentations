# Clean Architecture (Hexagonal/Onion)

> **Part 7: High-Level Architecture**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Code that Survives

---

## 0. Learning Objectives
*   **Beginner**: Why putting SQL in your Controller is bad.
*   **Developer**: Organizing code into Domain, Application, and Infrastructure layers.
*   **Architect**: Protecting the "Business Logic" from the "Framework" (Spring/Rails).

---

## 1. Problem Context
**Why does this exist?**
"Spaghetti Code".
*   UI calls Database directly.
*   Business Rules mixed with HTTP parsing.
*   **Legacy**: Can't switch from SQL to Mongo because SQL is everywhere.
*   **Testing**: Can't test Logic without spinning up a Database.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Hexagonal Architecture (Ports & Adapters)
*   **Core**: The Application logic (The Hexagon).
*   **Ports**: Interfaces (Input/Output).
*   **Adapters**: Implementation (Controller, JDBC Repository).

### 2. Clean Architecture (Uncle Bob)
*   Concentric Circles.
*   **Entities**: Enterprise Rules.
*   **Use Cases**: Application Rules.
*   **Adapters**: Gateways, Presenters.
*   **Frameworks**: Web, DB, Devices.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Dependency Rule
*   **Dependencies only point INWARD.**
*   The `Domain` knows NOTHING about the `Database`.
*   The `Database` depends on the `Domain`.

### Implementation
1.  **Domain**: `class User { ... }` (POJO).
2.  **Use Case**: `interface UserRepository { void save(User u); }`.
3.  **Infrastructure**: `class SqlUserRepository implements UserRepository { ... }`.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Inversion of Control (IoC)
*   Usually, Controller -> Service -> DAO -> DB Driver.
*   In Clean Arch: Controller -> Service -> **Interface** <- DAO Implementation.
*   This allows swapping the DAO without touching the Service.

### 2. Mapping Overhead
*   You will have `UserEntity` (DB), `UserDomain` (Logic), `UserDTO` (API).
*   You must map between them. `Mapper.toDomain(entity)`.
*   *Critique*: Boilerplate.
*   *defense*: Decoupling is worth the typing.

---

## 5. Trade-Off Analysis

| Feature | N-Tier (MVC) | Clean Architecture |
| :--- | :--- | :--- |
| **Speed to Market** | Fast | Slow (Boilerplate) |
| **Maintainability** | Low | High |
| **Testability** | Medium | High (Unit Test Core) |
| **Framework Lock** | High | Low |

---

## 6. Scaling Considerations

### Team Scaling
*   Clean Architecture enables parallel work.
*   Team A works on Core Logic (Use Cases).
*   Team B works on API Adapters (Controllers).
*   Team C works on DB Adapters (Postgres).

---

## 7. Failure Scenarios & Recovery

### 1. Anemic Domain Model
*   Devs use Clean Arch structure but enforce no rules in Domain Objects (Getters/Setters only).
*   Logic leaks into Service layer (Transaction Scripts).
*   **Fix**: Returns to DDD. Put logic in Entities.

---

## 8. Security Considerations

### 1. Boundary Protection
*   The Core is the "Trusted Zone".
*   All Input from Adapters (Controller) must be sanitized *before* entering the Core.

---

## 9. Performance Considerations

*   **Object Allocation**: Mapping large lists (DTO -> Domain -> Entity) burns CPU/RAM.
*   For high-performance loops, "Pragmatism" allows bypassing layers (CQRS Read side often skips Domain logic).

---

## 10. Real Production Lessons

### Framework Upgrades
*   **Scenario**: Spring Boot 2 -> 3 upgrade.
*   **Result**:
    *   **MVC App**: Broken. logic was tied to `HttpServletRequest`.
    *   **Clean App**: Core logic untouched. Only updated the Web Adapter. Safe and fast.

---

## 11. Interview Questions

### Basic
1.  What is Separation of Concerns?
2.  Why use Interfaces?
3.  What is a DTO?
4.  Why shouldn't the Entity have `@Table` annotations? (Ideally).
5.  What is Dependency Injection?

### Intermediate
1.  Explain Ports and Adapters.
2.  What is the Dependency Rule?
3.  Difference between Domain Service and Application Service.
4.  Why is mapping required?
5.  How do you handle Transactions in Clean Arch? (Unit of Work pattern / Gateway).

### Advanced
1.  Design a package structure for a Golang Clean Arch project.
2.  Critique Clean Architecture for CRUD apps. (Overkill).
3.  How does Screaming Architecture apply? (Folder names should scream "Accounting", not "Controllers").
4.  Refactor a Tight-Coupled Monolith to Clean Arch. Where to start?
5.  Compare Onion Architecture vs Hexagonal. (Mostly same concepts).

### Architect-Level
1.  "We are building a prototype. Should we use Clean Architecture?" (No. Use MVC. Refactor later if it survives).
2.  Design a Plugin system using Hexagonal Architecture.
3.  Evaluate the cognitive load of indirection vs the benefit of testability.

---

## 12. Scenario-Based System Design Problems

### 1. Design Banking Core
*   **Req**: 20 year lifespan.
*   **Choice**: **Clean Architecture**.
*   **Why**: Frameworks die. Business logic persists. Isolate the Core.

### 2. Design Marketing Blog
*   **Req**: Ship in 2 weeks.
*   **Choice**: **MVC (Django/Rails)**.
*   **Why**: CRUD. No complex rules. Don't over-engineer.

---

## 13. Summary & Architect Takeaways

1.  **The Framework is a Detail**: Keep it at arm's length.
2.  **Test the Core**: If your business logic can be tested with plain JUnit (no MockMvc, no H2), you have won.
3.  **Pragmatism**: Don't be a zealot. If it's a simple CRUD read, just call the DAO.
