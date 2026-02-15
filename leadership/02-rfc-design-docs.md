# RFC Process & Technical Design Docs (TDD)

> **Part 1: Process**  
> **Target Audience:** Tech Leads  
> **Status:** Under Review

---

## 0. The Problem
*   **Coding First**: Developer spends 2 weeks writing code, then Architect says "This won't scale".
*   **Wasted Effort**: 2 weeks lost.
*   **The Fix**: Write the Design Doc *first*.

---

## 1. The RFC Process (Request for Comments)
1.  **Draft**: Author writes the TDD (Technical Design Doc).
2.  **Review**: Team reads it mostly async (Google Docs / GitHub PR).
3.  **Meeting**: Only if needed to resolve conflicts.
4.  **Decision**: Approved / Rejected / Changes Requested.
5.  **Code**: Coding starts ONLY after Approval.

---

## 2. Structure of a Design Doc
Based on Google/Amazon standards.

### 1. Title & Metadata
*   Author, Reviewers, Status.

### 2. Overview (Abstract)
*   1 paragraph summary. What are we building?

### 3. Goals & Non-Goals
*   **Goals**: "Support 10,000 Concurrent Users".
*   **Non-Goals**: "We will NOT support Mobile app in v1". (Avoid scope creep).

### 4. Proposed Solution
*   **Architecture Diagram** (Mermaid/Draw.io).
*   **API Interface** (OpenAPI spec snippet).
*   **Data Model** (Schema).

### 5. Alternatives Considered
*   "We considered using Redis for caching, but chose Hazelcast because..."
*   Shows you did your homework.

### 6. Cross-Cutting Concerns
*   **Security**: AuthN/AuthZ? Data Encryption?
*   **Observability**: What metrics will we track?
*   **Cost**: Will this cost $10 or $10,000/month?

---

## 3. The "Review" Etiquette

### For Authors
*   **Don't take it personally**: Comments are on the *idea*, not you.
*   **Be explicit**: Ask "I am unsure about the Database choice, please focus review there".

### For Reviewers
*   **Nitpicks last**: Don't complain about spelling if the architecture is fundamentally flawed.
*   **Ask Questions**: "What happens if the Queue is full?" (Don't just say "This is bad").

---

## 4. Disagree and Commit
*   You argued for Solution A.
*   The team chose Solution B.
*   **Action**: You commit 100% to making Solution B successful. You do not sabotage it or say "I told you so".
