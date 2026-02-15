# Git Internals (The .git Directory)

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Deep Dive)  
> **Status:** Committed

---

## 0. Learning Objectives
*   **Beginner**: Where Git stores my code.
*   **Developer**: Recovering a "Lost Commit" using internals.
*   **Architect**: Understanding DAG (Directed Acyclic Graph).

---

## 1. Core Concepts (🟢 Beginner Level)

### "Content Addressable Filesystem"
*   Git is a key-value store.
*   **Key**: SHA-1 Hash of content.
*   **Value**: Compressed content (zlib).

### The `.git` Directory
*   `objects/`: The database.
*   `refs/`: Bookmarks (Branches, Tags).
*   `HEAD`: Current location.
*   `index`: Staging area.

---

## 2. Architecture Breakdown (The Objects)

### 1. Blob (Binary Large Object)
*   Stores **File Content**. (No filename, no permissions).
*   `git hash-object -w file.txt`.

### 2. Tree
*   Directory listing.
*   Maps `Filename` -> `Blob SHA`.
*   Also stores permissions (`100644`).

### 3. Commit
*   Snapshot of state.
*   Points to a **Tree** (Root directory).
*   Points to **Parent Commit(s)**.
*   Metadata (Author, Message, Date).

### 4. Tag
*   Points to a Commit (Annotated Tag).
*   Immutable.

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. HEAD (The Pointer to a Pointer)
*   `cat .git/HEAD` -> `ref: refs/heads/main`.
*   `cat .git/refs/heads/main` -> `<SHA-1 of latest commit>`.
*   **Detached HEAD**: HEAD points directly to a SHA-1, not a Branch Ref.

### 2. Packfiles
*   Git doesn't store 1000 copies of a file if you change 1 line.
*   It runs `git gc`.
*   Compresses objects into **Packfiles** (Deltas).
*   `objects/pack/*.pack`.

---

## 4. Scaling Considerations

### Monorepo Performance
*   Millions of Objects = Slow `git status`.
*   **Solution**:
    1.  **Scalar / VFS for Git**: Virtual filesystem (Microsoft).
    2.  **Sparse Checkout**: Only checkout `frontend/` folder.
    3.  **Shallow Clone**: `git clone --depth 1`.

---

## 5. Security Considerations

### 1. Signing Commits (GPG)
*   Anyone can set `user.name = "Linus Torvalds"`.
*   **Fix**: GPG Sign keys. Shows "Verified" badge on GitHub.

### 2. Secrets in History
*   Deleting a file in a new commit generally *keeps* it in history.
*   Attackers use `git log -p | grep password`.
*   **Fix**: BFG Repo-Cleaner or `git filter-repo`.

---

## 6. Real Production Lessons

### "The Dangling Commit"
*   You did `git reset --hard HEAD~1`. Your work is gone?
*   **No**. The Commit Object still exists in `.git/objects`.
*   It is just "Dangling" (No Ref points to it).
*   Use `git fsck --lost-found` or `git reflog` to revive it.

---

## 7. Interview Questions

### Basic
1.  Where is the local repo? (`.git` folder).
2.  What is a SHA-1? (40 char hash).
3.  Difference between `git add` and `git commit` regarding internals? (Add creates Blob/Index. Commit creates Commit Obj).

### Intermediate
1.  Explain `git gc`.
2.  What is a "Detached HEAD"?
3.  Difference between Soft, Mixed, and Hard Reset.

### Advanced
1.  How does Git calculate the SHA of a directory? (It doesn't. It hashes the Tree object).
2.  Why is Rebase "dangerous" on public branches? (Rewrites History -> Changes SHAs).
3.  Manually create a commit without using `git commit` (Using `git write-tree`, `git commit-tree`).

---

## 8. Summary & Architect Takeaways

1.  **Git is immutable**: You rarely "delete" data, you just stop pointing to it.
2.  **Refs are cheap**: A branch is just a 41-byte text file containing a Hash. Create them freely.
3.  **Reflog is Life**: It records where HEAD *was*. It saves you from bad merges.
