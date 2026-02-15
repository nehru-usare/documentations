# Branching Strategies

> **Part 2: Best Practices**  
> **Difficulty:** ⭐⭐⭐ (Process)  
> **Status:** Merged

---

## 0. Learning Objectives
*   **Beginner**: Why everyone working on `main` is chaos.
*   **Developer**: Following the team workflow vs Feature Branches.
*   **Architect**: Choosing a strategy for 100+ developers.

---

## 1. Core Concepts (🟢 Beginner Level)

### The Main Branch
*   The "Source of Truth".
*   Should always be deployable.
*   Protected (No direct push).

---

## 2. Architecture Breakdown (The Strategies)

### 1. Git Flow (Legacy / Enterprise)
*   **Branches**:
    *   `main` (Prod).
    *   `develop` (Integration).
    *   `feature/*` (Dev work).
    *   `release/*` (Prep for Prod).
    *   `hotfix/*` (Emergency).
*   **Pros**: Strict control. Good for infrequent releases (Version 1.0, 2.0).
*   **Cons**: Complexity. "Merge Hell" between develop and release. Only for Waterfall/Slow Agile.

### 2. GitHub Flow (Web / Simple)
*   **Branches**:
    *   `main` (Prod).
    *   `feature/*`.
*   **Flow**: Key concept is **Pull Request**.
    1.  Branch off main.
    2.  Commit.
    3.  Open PR.
    4.  Merge to main -> **Auto Deploy**.
*   **Pros**: Simple. CI/CD friendly.
*   **Cons**: Bad for native apps (iOS) that need version numbers.

### 3. Trunk-Based Development (Modern / FAANG)
*   **Branches**:
    *   `main` (The Trunk).
*   **Flow**:
    *   Developers commit directly to main (or short-lived PRs).
    *   **Feature Flags**: Unfinished code is hidden behind a flag.
*   **Pros**: No Merge Hell. Continuous Integration in truest sense.
*   **Cons**: Requires high test coverage / Senior team.

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. Feature Flags vs Long Branches
*   Long Branch: Code diverges. Merging takes days.
*   Feature Flag: Code is in main, but `if (flags.enableNewUI)` is false.
*   *Trunk Based works because of Feature Flags*.

---

## 4. Trade-Off Analysis

| Strategy | Speed | Safety | Complexity |
| :--- | :--- | :--- | :--- |
| **Git Flow** | Low | High | High |
| **GitHub Flow**| High | Med | Low |
| **Trunk Based**| Very High | Low (Needs Tests) | Med |

---

## 5. Scaling Considerations

### Monorepo Strategy
*   Google/Facebook use Trunk Based.
*   With 10,000 commits a day, branching is too slow.
*   Robotic Merge Queue (e.g., "Kodiak" or "MergeQueue") ensures main never breaks.

---

## 6. Real Production Lessons

### "The Release Freeze"
*   Git Flow often leads to "Code Freeze" periods where `develop` is locked.
*   Developers sit idle.
*   **Fix**: Move to Trunk Based.

---

## 7. Interview Questions

### Basic
1.  Describe GitHub Flow.
2.  What is a Pull Request?
3.  Where do you branch `hotfix` from? (Main, typically).

### Intermediate
1.  Git Flow vs Trunk Based.
2.  What is a Feature Flag?
3.  How to handle a merge conflict?

### Advanced
1.  Discuss Semantic Versioning (SemVer) with Git Flow.
2.  How to implement Trunk Based Dev in a regulated industry (Banking)? (Pair Programming acts as Code Review).
3.  Design a release pipeline for a mobile app using Git Tags.

---

## 8. Summary & Architect Takeaways

1.  **Start Simple**: Use GitHub Flow.
2.  **Avoid Git Flow**: Unless you sell boxed software.
3.  **Merge Often**: The longer a branch lives, the harder it is to merge.
