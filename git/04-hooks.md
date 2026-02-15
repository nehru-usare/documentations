# Git Hooks (Client-side vs Server-side)

> **Part 2: Mastery**  
> **Difficulty:** ⭐⭐⭐ (Automation)  
> **Status:** Hooked

---

## 0. Learning Objectives
*   **Beginner**: Preventing "WIP" commits.
*   **Developer**: Enforcing Linting before Push.
*   **Architect**: Enforcing Commit Message standards globally.

---

## 1. Core Concepts (🟢 Beginner Level)

### The Interceptors
*   Git Hooks are scripts that run **automatically** before/after Git events.
*   Location: `.git/hooks/`.
*   Language: Bash, Python, Ruby (Any executable).

---

## 2. Architecture Breakdown (The Lifecycle)

### 1. Client-Side Hooks (Run on Developer's Laptop)
*   **pre-commit**: Runs before commit message editor opens.
    *   *Use Case*: Linting (`eslint`), formatting (`prettier`), checking for secrets.
*   **commit-msg**: Runs after message is typed.
    *   *Use Case*: Enforce "JIRA-123: Description" format.
*   **pre-push**: Runs before `git push`.
    *   *Use Case*: Run Unit Tests (prevent breaking build).

### 2. Server-Side Hooks (Run on Remote/GitHub Enterprise)
*   **pre-receive**: Runs when push arrives. Can reject the *entire* push.
    *   *Use Case*: Block pushes to `main`. Compliance checks.
*   **post-receive**: Runs after push is accepted.
    *   *Use Case*: Trigger Jenkins/Deployment.

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. Sharing Hooks
*   Problem: `.git/hooks` is **not** committed to the repo. (Security feature).
*   **Fix 1**: **Husky** (Node.js). Configures hooks in `package.json`.
*   **Fix 2**: **pre-commit** (Python framework). Config file `.pre-commit-config.yaml`.
*   **Fix 3**: `git config core.hooksPath .githooks` (Change hooks dir to a committed folder).

---

## 4. Trade-Off Analysis

| Hook | Speed | Security |
| :--- | :--- | :--- |
| **Client-Side** | Fast (Local) | Bypassable (`--no-verify`) |
| **Server-Side** | Slow (Network) | Enforceable (Strict) |

---

## 5. Scaling Considerations

### The "Slow Commit"
*   If `pre-commit` runs full test suite (5 mins), Developers will bypass it.
*   **Rule**: Hooks must be < 5 seconds.
*   Run heavy tests in CI, not hooks.

---

## 6. Real Production Lessons

### "The Accidental Secret"
*   **Scenario**: Dev commits AWS Key.
*   **Prevention**: `pre-commit` hook using `detect-secrets` or `gitleaks`.
*   Blocks the commit instantly.

---

## 7. Interview Questions

### Basic
1.  Where are hooks stored?
2.  Can you bypass a hook? (Yes, `git commit --no-verify`).
3.  Difference between Client and Server hooks.

### Intermediate
1.  How to share hooks with the team? (Husky/pre-commit).
2.  Write a bash script to reject commits without "JIRA-" in message.
3.  Why doesn't Git clone hooks? (Security - avoid running arbitrary code).

### Advanced
1.  Design a `pre-receive` hook to block large binary files (>10MB).
2.  How to debug a failing hook? (Echo to stderr).
3.  Explain the execution order of `pre-commit`, `prepare-commit-msg`, `commit-msg`.

---

## 8. Summary & Architect Takeaways

1.  **Automate Compliance**: Don't rely on humans to remember Linting.
2.  **Husky is Standard**: For JS/TS projects, Husky is mandatory.
3.  **Don't Annoy Devs**: If hooks are flaky/slow, they will be disabled.
