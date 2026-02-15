# Architecture Decision Records (ADR)

> **Part 1: Process**  
> **Target Audience:** Senior Engineers & Architects  
> **Status:** Decided

---

## 0. The Problem
*   **Amnesia**: Why did we choose Mongo over Postgres 2 years ago? ("I think Bob liked it?").
*   **Re-litigation**: New hires constantly question old decisions without context.
*   **Invisibility**: Decisions happen in chats/meetings, nowhere written down.

---

## 1. The Solution: ADR
An **Architecture Decision Record** is a short text file (Markdown) stored *in the code repo*.
It captures a single, significant architectural decision.

### The Lifecycle
1.  **Proposed**: Pull Request open. Team discusses.
2.  **Accepted**: PR merged. This is now the law.
3.  **Superseded**: A new ADR replaces this one.

---

## 2. The Golden Template
Copy this to `doc/adr/001-template.md`.

```markdown
# ADR 001: Use PostgreSQL for User Data

**Status**: Accepted  
**Date**: 2026-02-15  
**Author**: Nehru Usare  

## Context
We need a database to store User Profiles. 
We expect relational data (Users have Orders).
We need strong consistency (ACID) for billing.

## Options Considered
1.  **PostgreSQL**: Strong ACID, relational, free.
2.  **MongoDB**: Flexible schema, good for rapid dev.
3.  **Cassandra**: High write throughput, but overkill for our scale.

## Decision
We chose **PostgreSQL**.

## Justification
*   We need ACID compliance for billing (Mongo 4.0 has transactions, but SQL is stricter).
*   Our data is highly relational. Join performance is critical.
*   Team is already familiar with SQL.

## Consequences
*   **Positive**: Data integrity is guaranteed.
*   **Negative**: Schema migrations (Liquibase) are required for every change.
```

---

## 3. Best Practices
1.  **Keep it Short**: 1-2 pages max. If it's 20 pages, it's a Design Doc, not an ADR.
2.  **Number Them**: `001-setup.md`, `002-database.md`.
3.  **Store with Code**: `docs/adr/`.
4.  **Immutable**: Once accepted, don't edit it. Creating a new ADR (`005-move-to-mongo.md`) that says "Supersedes 001".

---

## 4. When to write an ADR?
*   Should we use Library A or Library B? (**Yes**).
*   Should we rename this variable? (**No**).
*   Should we move to Microservices? (**YES**).
