# Golden Hammer

> **Part 7: Anti-Patterns**  
> **Severity:** 🟡 Medium  
> **Status:** Cognitive Bias

---

## 1. The Symptom
> "I know **Kafka**. Let's use Kafka for inter-process communication on this laptop."

> "I know **NoSQL**. Let's store this highly relational financial ledger in MongoDB."

> "I know **Microservices**. Let's split this 2-screen CRUD app into 15 services."

---

## 2. Why it happens
1.  **Comfort Zone**: Developers prefer using what they know over what is best.
2.  **Resume Driven Development**: Developers pick tech to look good on their CV, not to solve the business problem.
3.  **Ignorance**: Not knowing that a simpler tool exists (e.g., using a distributed cache when a simple `HashMap` would suffice).

---

## 3. The Fix
1.  **Evaluate Alternatives**: Force the team to propose at least 2 options for every architectural decision. (e.g. SQL vs NoSQL).
2.  **Proof of Concept (PoC)**: Build a small prototype with the "Boring Technology". It often wins.
3.  **KISS Principle**: Keep It Simple, Stupid.

---

## 4. Famous Golden Hammers
*   **XML/JSON**: Storing config, data, logic, and UI definitions all in XML.
*   **Regex**: Trying to parse HTML with Regex. (Don't).
*   **Design Patterns**: Forcing a Visitor Pattern when a simple loop would do.
