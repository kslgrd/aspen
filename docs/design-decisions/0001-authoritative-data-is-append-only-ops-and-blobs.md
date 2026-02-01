# ADR-0001: Authoritative data is append-only operations and blobs

## Status
Accepted

## Context

Aspen is a local-first, end-to-end encrypted platform that supports:

- offline operation
- peer-to-peer sync
- CRDT-based collaboration
- multiple apps sharing a single data substrate

These requirements imply:

- devices must be able to operate independently
- replicas must converge without centralized coordination
- data must be verifiable, replayable, and mergeable
- encryption boundaries must be clean and enforceable

Traditional approaches such as syncing databases, mutable state, or derived indexes do not satisfy these constraints reliably.

A clear distinction between **authoritative data** and **derived state** is required.

---

## Decision

All authoritative data in Aspen is represented as:

1. **Append-only operations**
   - Immutable
   - Encrypted
   - Signed by device keys
   - Scoped to a space
   - CRDT-compatible

2. **Encrypted blobs**
   - Immutable
   - Content-addressed
   - Stored separately from operations
   - Referenced by operations

There is no other source of truth.

Any “update” to data is modeled as the addition of a new operation and/or blob.

---

## Consequences

### Positive

- **Deterministic replay**
  - A replica can reconstruct state by replaying operations.
- **Reliable sync**
  - Anti-entropy reconciliation is straightforward.
- **CRDT compatibility**
  - Concurrent edits can be merged without coordination.
- **Auditability**
  - History is preserved by construction.
- **Encryption clarity**
  - Only authoritative data needs to be encrypted and replicated.
- **Multi-app safety**
  - Apps cannot corrupt shared state by mutating derived views.

### Negative

- **Higher conceptual overhead**
  - Developers must think in terms of revisions and events.
- **Storage growth**
  - History accumulates unless compacted.
- **Derived state complexity**
  - Materialized views and indexes must be rebuilt or maintained incrementally.

These costs are accepted as necessary tradeoffs.

---

## What is explicitly not authoritative

The following are *derived state* and must never be treated as sources of truth:

- SQLite databases
- Materialized record views
- Full-text search indexes
- Vector embeddings
- Graph layouts or projections
- Caches of any kind

Derived state may be discarded and rebuilt at any time.

---

## Implications for implementation

- No SQLite file is ever synchronized.
- No mutable state is shared between devices.
- Deletion is modeled as a semantic operation, not physical removal.
- Compaction, if implemented, must preserve logical history.
- Sync protocols operate only on operations and blobs.

Any design that attempts to bypass this model is incorrect by definition.

---

## Alternatives considered

### Syncing mutable databases
Rejected:
- breaks under concurrent writes
- opaque to CRDT semantics
- difficult to encrypt and reason about safely

### Server-authoritative state
Rejected:
- violates local-first and privacy constraints
- introduces mandatory infrastructure
- weakens user data ownership

### Eventual migration to logs later
Rejected:
- creates early technical debt
- encourages accidental reliance on mutable state

---

## Related documents

- [`docs/architecture/overview.md`](../architecture/overview.md)
- [`docs/architecture/data-model.md`](../architecture/data-model.md)
- [`docs/architecture/sync-model.md`](../architecture/sync-model.md)
- [`docs/architecture/security-model.md`](../architecture/security-model.md)

---

## Notes

This decision is foundational.

If it is ever reversed, the architecture of Aspen must be reconsidered from first principles.