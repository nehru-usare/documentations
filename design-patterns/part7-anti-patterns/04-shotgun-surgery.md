# Shotgun Surgery

> **Part 7: Anti-Patterns**  
> **Severity:** 🟠 High  
> **Status:** Death by a Thousand Edits

---

## 1. The Symptom
You want to add a `currency` field to the Price.
You have to edit:
1.  `DatabaseSchema.sql`
2.  `Product.java`
3.  `ProductDTO.java`
4.  `PriceCalculator.java`
5.  `InvoiceGenerator.java`
6.  `PaymentGateway.java`

You touch **10 files** for **1 requirement**.
If you miss one, the system crashes.

---

## 2. Cause: Divergent Change
This is the opposite of the God Object.
*   **God Object**: Too much responsibility in 1 place.
*   **Shotgun Surgery**: 1 responsibility scattered across 10 places.

Usually caused by **primitive obsession** (passing `BigDecimal amount` everywhere instead of a `Money` object) or Copy-Paste coding.

---

## 3. The Fix

### 1. Introduce Parameter Object
If you pass `(String uuid, String name, String email)` to 5 methods, create a `UserProfile` object.

### 2. Group Functionality
If `InvoiceGenerator` and `PaymentGateway` both calculate tax, move tax calculation to a `TaxService`.

### 3. Replace Primitives with Objects
Instead of `BigDecimal`, use `Money` class that contains `Amount` and `Currency`. You update logic in `Money` class once.

---

## 4. Architect Takeaway
*   **Cohesion**: Code that changes together should stay together.
