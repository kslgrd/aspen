# ADR-0008: Derived intelligence is local-only and non-authoritative

## Status
Accepted

## Context

Aspen aims to provide powerful ways to explore and understand information, including:
- search
- ranking and relevance
- vector similarity
- spatial and graph-based views
- automatic layout and clustering

These capabilities are valuable, but they introduce risks if treated as authoritative or shared:
- privacy leakage
- nondeterministic behavior across devices
- large sync payloads
- hidden coupling between apps and infrastructure

Aspen already distinguishes between authoritative data and derived state.
This decision formalizes that distinction for all “intelligent” features.

---

## Decision

All **derived intelligence** in Aspen is **local-only and non-authoritative**.

Derived intelligence includes, but is not limited to:
- full-text search indexes
- vector embeddings
- similarity rankings
- inferred relationships
- graph projections and layouts
- relevance scores and heuristics

Derived intelligence:
- is computed locally
- is never synchronized
- is always rebuildable from authoritative data
- must not affect authoritative state directly

---

## Rationale

### Privacy

Derived intelligence often encodes semantic meaning.
Syncing it would:
- leak information about content and intent
- expand the attack surface
- blur trust boundaries

Keeping it local preserves Aspen’s privacy guarantees.

---

### Correctness and determinism

Derived intelligence:
- depends on heuristics
- evolves as algorithms change
- may differ across platforms or versions

Treating it as authoritative would break:
- deterministic replay
- predictable sync behavior
- cross-device consistency guarantees

---

### Architectural clarity

By enforcing a strict boundary:
- authoritative data remains simple and verifiable
- “smart” features remain optional and replaceable
- apps can innovate without corrupting shared state

This keeps Aspen extensible over time.

---

## Implications

### Search
- Search indexes are built locally from decrypted authoritative data.
- Indexes may be deleted and rebuilt at any time.
- Search results may differ slightly across devices.

---

### Vector embeddings
- Embeddings are computed locally.
- Embeddings are stored locally.
- Different devices may use different models or versions.

This is acceptable.

---

### Graph and spatial views
- Layouts and projections are derived.
- View state may be persisted locally or as user-owned records.
- Derived relationships are not promoted to authoritative links automatically.

---

### Automation and suggestions
- Suggestions may be generated from local intelligence.
- Applying a suggestion always results in an explicit authoritative operation.
- Intelligence proposes; users or apps decide.

---

## What is allowed to be persisted

Derived intelligence **may** persist locally:
- cached scores
- layout coordinates
- clustering results
- UI state

It **must not** be treated as shared truth.

---

## What is explicitly forbidden

- Syncing embeddings or indexes
- Inferring links and writing them authoritatively without explicit action
- Treating derived scores as durable facts
- Making authoritative behavior depend on derived state

Any such design violates Aspen’s core model.

---

## Consequences

### Positive

- **Strong privacy guarantees**
- **Clear sync semantics**
- **Algorithmic freedom**
- **Low coupling between apps**

### Negative

- **Inconsistent experiences**
  - Different devices may surface different suggestions.
- **Rebuild cost**
  - Intelligence must be recomputed after data loss.

These tradeoffs are accepted.

---

## Alternatives considered

### Syncing derived intelligence
Rejected:
- privacy risks
- version skew
- nondeterministic convergence

### Server-side intelligence
Rejected:
- requires trusted infrastructure
- violates local-first principles
- centralizes insight

### Promoting intelligence to authoritative state
Rejected:
- breaks auditability
- introduces hidden mutations
- undermines user agency

---

## Related documents

- ADR-0001: Authoritative data is append-only operations and blobs
- ADR-0002: SQLite is derived local state only
- ADR-0007: External sources have stable identities and revision streams
- `docs/architecture/sync-model.md`

---

## Notes

Aspen separates **facts** from **interpretations**.

Facts are shared.
Interpretations are local.