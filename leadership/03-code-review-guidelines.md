# Code Review Guidelines

> **Part 1: Process**  
> **Target Audience:** Every Engineer  
> **Status:** Approved

---

## 0. The Philosophy
*   **Code is Liability**: Less code is better.
*   **Quality > Speed**: But Speed matters too. (Google standard: < 4 hours turnaround).
*   **It's a Conversation**: Not a lecture.

---

## 1. Author Checklist (Before submitting PR)
1.  **Self-Review**: Did you read your own diff? (Catches 50% of bugs).
2.  **Tests**: Did you add Unit Tests? Do they pass?
3.  **Description**: "Fixes bug" is bad. Explain *Why* and *How*.
4.  **Screenshots**: If UI change, attach Before/After images.
5.  **Small PRs**: < 400 lines. If larger, split it.

---

## 2. Reviewer Checklist (Hierarchy of Needs)

### Level 1: Correctness (Priority: Critical)
*   Does it work?
*   Are there race conditions?
*   Security flaws? (SQL Injection, IDOR).

### Level 2: Design
*   Is this the right place for this code?
*   Is it over-engineered?
*   Does it duplicate existing utils?

### Level 3: Readability
*   Are variable names clear? (`x` vs `userIndex`).
*   Are comments helpful? (Explain *Why*, not *What*).

### Level 4: Style (Priority: Low)
*   Indentation, brackets, spaces.
*   **Automate this**: Use a Linter (Prettier/Checkstyle). Humans shouldn't argue about spaces.

---

## 3. Communication Standards

### Phrasing
*   ❌ "You forgot to close the connection." (Accusatory).
*   ✅ "Does this connection need to be closed?" (Socratic).

### Nitpicks
*   Label them. "Nit: typo in comment". (Signals: "Fix this if you want, but don't block merge").

---

## 4. The "LGTM" Trap
*   **LGTM**: "Looks Good To Me".
*   Don't just rubber-stamp.
*   If you don't understand the code, say "I am not qualified to review the Auth logic, adding @SecurityTeam".

---

## 5. Speed
*   Unreviewed PRs rot.
*   Reviewing code is **your job**. It is not "something you do when you have time".
*   **Rule**: diverse your context to review code 2x a day (Morning/Afternoon).
