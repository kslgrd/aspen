# ADR-0002: SQLite is derived local state only

## Status
Accepted

## Context

Aspen requires:

- local-first operation
- peer-to-peer sync
- end-to-end encryption
- CRDT-based convergence
- support for multiple apps sharing a single data substrate

The system needs fast local queries for:

- UI rendering
- search
- filtering
- graph traversal
- spatial layouts

SQLite is an obvious and practical tool for this purpose.

However, treating SQLite as authoritative or synchronizing database files directly would conflict with Aspen’s core constraints.

---

## Decision

SQLite databases in Aspen are **derived local state only**.

SQLite may be used to:

- materialize views of authoritative data
- support fast local queries
- power search, indexing, and UI responsiveness

SQLite must **never** be treated as:

- a source of truth
- a unit of synchronization
- an object replicated between devices

All SQLite data must be:

- rebuildable from authoritative operations and blobs
- discardable without data loss

---

## Consequences

### Positive

- **Clear authority boundary**
  - There is exactly one source of truth: append-only ops and blobs.
- **Safe concurrency**
  - No risk of database-level merge conflicts.
- **CRDT alignment**
  - SQLite never needs to reason about causality or ordering.
- **Simpler encryption model**
  - Only authoritative data requires strict encryption guarantees.
- **Multi-app isolation**
  - Apps can build their own views without corrupting shared state.

### Negative

- **Rebuild cost**
  - Indexes and views must be reconstructed after data loss or schema changes.
- **Implementation complexity**
  - The platform must maintain projection logic.
- **Delayed availability**
  - Large datasets may require time to rehydrate indexes.

These tradeoffs are accepted.

---

## Enforcement

The following rules apply:

- SQLite files are never synchronized.
- SQLite files are never shared between devices.
- SQLite files may be deleted at any time without data loss.
- No app may assume SQLite state is complete or up-to-date.
- All authoritative writes must originate as operations, not SQL mutations.

Any feature that requires syncing SQLite state is incorrectly designed.

---

## Implications for search and indexing

- Full-text search indexes are derived from decrypted authoritative data.
- Vector embeddings are computed locally and stored locally.
- Embeddings are never synced, as they leak semantic information.
- If a device is compromised, derived indexes are considered expendable.

Search correctness depends on replaying authoritative history, not on database replication.

---

## Implications for platform APIs

- Platform APIs expose **intent-based operations**, not SQL access.
- Apps request data via:
  - subscriptions
  - queries over materialized views
- Apps must tolerate partial or rebuilding indexes.

The platform controls projection and indexing.

---

## Alternatives considered

### Syncing SQLite files
Rejected:
- unsafe under concurrent writes
- opaque conflict resolution
- difficult to encrypt safely
- tightly couples apps to storage internals

### Treating SQLite as authoritative
Rejected:
- incompatible with CRDT-based sync
- violates append-only invariant
- encourages hidden state mutation

### Multiple authoritative databases
Rejected:
- creates ambiguity
- breaks deterministic replay
- complicates reasoning and debugging

---

## Related documents

- ADR-0001: Authoritative data is append-only operations and blobs
- `docs/architecture/data-model.md`
- `docs/architecture/sync-model.md`

---

## Notes

SQLite is a performance optimization, not a design primitive.

Any architecture that relies on SQLite for correctness is incompatible with Aspen.