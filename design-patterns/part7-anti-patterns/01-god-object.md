# God Object (The Blob)

> **Part 7: Anti-Patterns**  
> **Severity:** 🔴 Critical  
> **Status:** The All-Knowing Class

---

## 1. The Symptom
You open `OrderManager.java`.
It has **5,000 lines** of code.
It imports:
*   `java.sql.*`
*   `javax.mail.*`
*   `com.stripe.*`
*   `org.apache.pdfbox.*`

It handles:
1.  Saving Orders.
2.  Sending Emails.
3.  Charging Credit Cards.
4.  Generating PDFs.

---

## 2. Why is this Bad?
1.  **Testing Nightmare**: To test "PDF Generation", you have to mock the Database and the Payment Gateway.
2.  **Concurrency Issues**: Everyone on the team is editing this one file. Merge conflicts daily.
3.  **Memory Leak**: If this object holds state, it holds *everything*.

---

## 3. The Fix: Decomposition
Apply **Single Responsibility Principle (SRP)**.

### Break it Down
*   `OrderRepository` (SQL stuff).
*   `EmailService` (Mail stuff).
*   `PaymentService` (Stripe stuff).
*   `PdfGenerator` (PDF stuff).

### The "Manager" Trap
Beware of classes named `Manager`, `Processor`, or `System`. These names are vague and encourage God Objects.
*   **Bad**: `UserManager`
*   **Good**: `UserRegistrar`, `UserAuthenticator`, `UserProfileUpdater`.

---

## 4. Architect Takeaway
*   **Line Limit**: If a class exceeds 500 lines, it is suspect. If it exceeds 1000 lines, it is guilty.
*   **Injection Limit**: If a constructor has > 7 dependencies, it is doing too much.
