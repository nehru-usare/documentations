# Advanced Git Commands

> **Part 2: Mastery**  
> **Difficulty:** ⭐⭐⭐⭐ (Productivity)  
> **Status:** Rebased

---

## 0. Learning Objectives
*   **Beginner**: Fixing a "Broken" repo.
*   **Developer**: Using `bisect` to find which commit broke the build.
*   **Architect**: Cleaning huge repositories.

---

## 1. Core Concepts (🟢 Beginner Level)

### The Safety Net
*   **Reflog**: Git records every time HEAD changes.
*   `git reflog`.
*   Allows you to undo `git reset --hard`.

---

## 2. Architecture Breakdown (The Commands)

### 1. Git Bisect (The Debugger)
*   **Problem**: Bug introduced sometime in last 100 commits.
*   **Manual**: Check out randomly? No.
*   **Algorithm**: Binary Search.
*   **Flow**:
    ```bash
    git bisect start
    git bisect bad HEAD
    git bisect good v1.0
    # Git checks out middle commit.
    # You test.
    git bisect good  # or bad
    # Git jumps to next half.
    ```
*   **Result**: Finds the culprit in O(log N) steps.

### 2. Git Rerere (Reuse Recorded Resolution)
*   If you fix the same merge conflict 5 times (during long rebase), enable this.
*   Git "remembers" how you solved file A.
*   Auto-applies fix next time.

### 3. Git Blame (Forensics)
*   `git blame -L 10,20 file.py`.
*   Shows who wrote lines 10-20 and *in which commit*.

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. Interactive Rebase (`-i`)
*   `git rebase -i HEAD~3`.
*   **Pick**: Keep commit.
*   **Squash**: Merge into previous commit.
*   **Drop**: Delete commit.
*   **Edit**: Pause rebase to change content of old commit.
*   *Power Tool for clean history*.

---

## 4. Scenarios

### The "Oops, I committed a Password"
1.  **Stop**: Don't push.
2.  `git reset --soft HEAD~1`. (Undo commit, keep changes).
3.  Remove password.
4.  Commit again.

### The "I pushed the Password"
1.  **Panic**: It's in history.
2.  **Fix**: `git filter-repo --path secrets.txt --invert-paths` (Rewrites ALL history).
3.  **Force Push**: `git push --force`. (Breaks everyone else's repo).

---

## 5. Security Considerations

### 1. Force Push (`--force`)
*   Destructive.
*   **Better**: `--force-with-lease`.
*   Checks if remote has changed since you last fetched. Prevents overwriting colleague's work.

---

## 6. Real Production Lessons

### "The Merge Hell"
*   Feature branch is 3 months old.
*   Main has moved on.
*   **Fix**: Don't Merge main into feature. **Rebase** feature onto main frequently. Keep diff small.

---

## 7. Interview Questions

### Basic
1.  What is `git stash`?
2.  Difference between Rebase and Merge.
3.  What does `git cherry-pick` do?

### Intermediate
1.  How to find a bug introduced 2 weeks ago? (Bisect).
2.  Recover a deleted branch. (Reflog -> Checkout SHA -> Create Branch).
3.  What is `git add -p`? (Patch mode - stage partial file).

### Advanced
1.  Explain `git worktree`. (Checkout multiple branches in different folders from SAME repo).
2.  How to combine 10 commits into 1? (Squash).
3.  Difference between `git fetch` and `git pull`. (Pull = Fetch + Merge).

---

## 8. Summary & Architect Takeaways

1.  **Reflog is your best friend**.
2.  **Bisect saves hours**.
3.  **Never Force Push** to shared branches (main/develop).
