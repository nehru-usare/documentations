# Date & Time API: Mastering Temporal Logic

> **Part 6: Advanced I/O & Standards**  
> **Level:** Principal Engineer  
> **Status:** Timely

---

## 0. Learning Objectives

*   **Developer**: Why `java.util.Date` is mutable (and bad).
*   **Senior**: Converting between Timezones correctly.
*   **Architect**: Storing time in Databases (UTC vs Local).

---

## 1. The Old World (`java.util.Date`)

*   **Mutable**: You can change the time of a Date object. Not thread-safe.
*   **Confusing**: Year starts at 1900. Month 0 is January.
*   **Advice**: **Burn it**. Never use it in new code.

---

## 2. The New World (`java.time`) - Java 8

Immutable. Thread-safe. Domain-Driven.

### 2.1 The Core Types
1.  **Instant**: A point on the timeline (UTC). "Unix Timestamp". Usage: **Logs, Database Storage**.
2.  **LocalDate**: "2026-02-15". No time, no zone. Usage: **Birthdays**.
3.  **LocalTime**: "14:30". No date, no zone. Usage: **Alarm Clock**.
4.  **LocalDateTime**: "2026-02-15T14:30". No zone. Usage: **Bad Idea** (usually). Only use if the event happens "at 2 PM wherever you are" (Holiday).
5.  **ZonedDateTime**: "2026-02-15T14:30+05:30[Asia/Kolkata]". Full context. Usage: **UI Display**, **Calendar Invitations**.

---

## 3. Best Practices

### 3.1 Storage (Database)
*   **Rule**: Always store in **UTC**.
*   **Type**: Use `Instant` in Java -> `TIMESTAMP WITH TIME ZONE` (Postgres).
*   **Why**: If you store Local Time, and the server moves to a different timezone (or Daylight Savings hits), your data is corrupted.

### 3.2 Display (UI)
*   Convert `Instant` (UTC) to `ZonedDateTime` (User's Zone) **only at the edge** (API response or UI rendering).

---

## 4. Summary & Architect Takeaways

1.  **Clock Dependency**: Don't use `Instant.now()`. Inject a `Clock` bean `clock.instant()`. Allows testing (Time travel).
2.  **Period vs Duration**:
    *   `Period`: "3 Months" (Calendar based).
    *   `Duration`: "90 Days * 24 Hours" (Physics based).
3.  **ZoneId**: Use "Asia/Kolkata", not "IST" (IST is ambiguous - Irish Standard Time vs India Standard Time).

---
*Next Chapter: Security Evolution.*
