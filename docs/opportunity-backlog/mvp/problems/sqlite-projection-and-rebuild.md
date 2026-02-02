# Problem: SQLite projection boundaries and rebuild semantics

**Problem statement:**  
When Aspen uses SQLite for timeline and search, I want it to be fast and convenient while staying disposable,
so we can rebuild confidently and avoid leaks, but it’s easy to accidentally let SQLite become authoritative
or to persist sensitive plaintext in ways that conflict with the security model.

**Evidence**

- ADR-0002: SQLite is derived local state only; it must be disposable and rebuildable. (See [`0002-sqlite-is-derived-local-state-only.md`](../../../design-decisions/0002-sqlite-is-derived-local-state-only.md).)
- MVP success criteria requires: deleting SQLite and restarting reconstructs the same timeline. (See [`mvp-overview.md`](../mvp-overview.md).)
- The architecture distinguishes authoritative encrypted data vs derived state, but does not yet define the exact projection boundary for:
  - extracted URL summaries
  - extracted file text
  - search indexes (FTS)

---

## Opportunities

- How might we define a crisp “authoritative vs derived” boundary for text extraction and search that preserves privacy and rebuildability?

---

## Hypotheses

- We believe storing extracted text authoritatively as encrypted blobs (and projecting only what’s needed) will achieve rebuildability
  without relying on fragile re-extraction at rebuild time.
- We believe defining a strict rule for what plaintext may exist in SQLite (and when) will achieve a security posture that is easy to reason about.

**Success metrics**

- A full rebuild from authoritative ops + blobs reproduces the same visible timeline events.
- No derived artifacts are required to recover user-visible history.
- We can explain (in docs) what plaintext exists on disk and why it is acceptable.

---

## Possible Concepts (MVE)

- **Hypothesis:** Authoritative extracted text  
  **MVE:** Store URL summary blobs authoritatively; SQLite stores only searchable text needed for UX; test rebuild by deleting SQLite  
  **Effort:** medium  
  **Signal:** rebuild passes; no re-fetch needed

- **Hypothesis:** Strict plaintext boundary  
  **MVE:** Write a “disk footprint” doc for MVP and validate it against a prototype implementation  
  **Effort:** low  
  **Signal:** no surprises during review

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- This problem is intentionally about *boundaries and semantics*, not the specific SQLite schema.
- Related to “locked/unlocked” UX: if the store is locked, what (if anything) can SQLite show/search?

## Links

- [`mvp-overview.md`](../mvp-overview.md)
- [`storage-and-indexes.md`](../spec/storage-and-indexes.md)
- [`0002-sqlite-is-derived-local-state-only.md`](../../../design-decisions/0002-sqlite-is-derived-local-state-only.md)
- [`security-model.md`](../../../architecture/security-model.md)
