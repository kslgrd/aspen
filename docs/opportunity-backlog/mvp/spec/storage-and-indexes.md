# Storage and Indexes (MVP Spec)

This document specifies the storage layers required for the MVP, focusing on the boundary between authoritative encrypted data and derived, disposable indexes.

Key invariants:
- Authoritative data is append-only ops + blobs (see [`0001-authoritative-data-is-append-only-ops-and-blobs.md`](../../../design-decisions/0001-authoritative-data-is-append-only-ops-and-blobs.md)).
- SQLite is derived local state only (see [`0002-sqlite-is-derived-local-state-only.md`](../../../design-decisions/0002-sqlite-is-derived-local-state-only.md)).

---

## Storage layers

### 1) Authoritative store (encrypted)

Per space (Inbox only in MVP):
- **Operation log**: append-only, encrypted, signed.
- **Blob store**: encrypted, content-addressed blobs referenced by ops.

The store must support:
- atomicity: ops that reference blobs must not leave the system in a corrupted state if a crash occurs mid-write
- replay: iterate operations in a stable order sufficient to rebuild derived state

### 2) Derived store (SQLite)

SQLite contains:
- materialized views needed for timeline rendering
- full-text search indexes (FTS) or a pointer to an external index
- any local-only bookkeeping required for incremental projection

SQLite must be:
- disposable
- rebuildable from the authoritative store alone

---

## Timeline projection model (MVP)

The MVP timeline is the “ground truth view of what happened” (see [`mvp-overview.md`](../mvp-overview.md)).

Derived timeline events should support:
- multiple events for the “same” underlying source over time (re-share → new event)
- stable ordering in the UI (reverse chronological)

Open question:
- What is the exact event model and ordering rule that is deterministic under future multi-device concurrency?
  - (Likely ties into [`crdt-surface-and-conflict-semantics.md`](../problems/crdt-surface-and-conflict-semantics.md).)

---

## Rebuildability (MVP success criterion)

Acceptance requirement (see [`mvp-overview.md`](../mvp-overview.md)):
- Deleting SQLite and restarting reconstructs the same timeline.

This implies:
- Derived state must be reproducible from authoritative ops + blobs.
- Projection must be idempotent (replaying the same ops produces the same effective derived state).

Tracked unknowns:
- What is safe/acceptable to store in SQLite from a security and privacy posture?
- How do we design projections so they can be rebuilt efficiently and deterministically?

See:
- [`sqlite-projection-and-rebuild.md`](../problems/sqlite-projection-and-rebuild.md)
