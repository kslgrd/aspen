# ADR-0006: Links are versioned entities with revision history

## Status
Accepted

## Context

Aspen supports:
- multiple apps sharing a single data substrate
- cross-space relationships via overlay spaces
- append-only authoritative data
- CRDT-based synchronization
- history, auditability, and diffing

Links in Aspen are first-class entities, not derived edges.

Real-world use cases require that:
- sharing the same link more than once is meaningful
- changes to a link should publish events
- previous states of a link can be inspected and compared
- concurrent updates converge deterministically

Treating links as mutable rows or deduplicated edges fails to meet these requirements.

---

## Decision

Links in Aspen are modeled as **versioned entities with append-only revision history**.

- A logical link has a stable identity.
- Changes to a link are represented as new revisions.
- Re-sharing the same logical link creates a new revision.
- All link history is preserved.

There is no in-place mutation of links.

---

## Link identity

A logical link is identified by a canonical **link key**, derived from:

- overlay space identifier
- source `Ref`
- target `Ref`
- link kind (if applicable)

The link key uniquely identifies “the same relationship.”

Directionality rules are defined per link kind.

---

## Link revisions

A **link revision** represents a change to a link over time.

Properties:
- append-only
- authored by a device
- encrypted
- belongs to exactly one logical link
- may update metadata (label, notes, weight, confidence, etc.)
- may include a reason (e.g. `reshare`, `edit`, `auto-update`)

Revisions are authoritative events.

---

## Re-sharing semantics

Sharing the same logical link again:

- computes the same link key
- appends a new link revision
- emits a change event

If the payload is identical to the current revision, the platform may treat the operation as idempotent and suppress redundant revisions, unless the user explicitly intends to publish a change.

This balances correctness with noise reduction.

---

## History and diffing

Because link history is preserved:

- previous revisions can be inspected
- diffs can be computed client-side
- timelines of relationship changes can be presented

Diffing is an application concern; the platform preserves the data required to do so.

---

## Concurrency and convergence

Concurrent updates to the same logical link are resolved via CRDT rules applied to the “current revision” pointer.

- All revisions remain in history.
- A deterministic rule selects the effective revision.
- Conflicts may be surfaced in UI but replicas converge.

Link revision convergence follows the same principles as record revision convergence.

---

## Privacy and overlays

All links and link revisions are stored **only in overlay spaces**.

This ensures:
- no leakage of relationship existence
- no inference of cross-space structure by unauthorized users
- consistent permission enforcement

Content spaces are unaware of links.

---

## Consequences

### Positive

- **Semantic clarity**
  - Links represent relationships over time, not static edges.
- **Auditability**
  - Relationship changes are inspectable and explainable.
- **Eventfulness**
  - Re-sharing and edits publish meaningful change events.
- **CRDT compatibility**
  - Converges cleanly under concurrent edits.
- **Privacy preservation**
  - Overlay model remains intact.

### Negative

- **Increased data volume**
  - Link history accumulates.
- **More complex modeling**
  - Applications must reason about revisions.

These tradeoffs are accepted.

---

## Alternatives considered

### Mutable link records
Rejected:
- loses history
- breaks auditability
- ambiguous sync semantics

### Deduplicated edges with timestamps
Rejected:
- conflates identity and state
- insufficient for diffing
- brittle under concurrency

### Derived links only
Rejected:
- cannot support explicit re-sharing semantics
- loses user intent

---

## Related documents

- ADR-0001: Authoritative data is append-only operations and blobs
- ADR-0004: Cross-space relationships are stored in overlay spaces
- `docs/architecture/data-model.md`
- `docs/architecture/sync-model.md`

---

## Notes

Links are not just connections; they are evolving statements.

Treating them as versioned entities is required to model real user behavior accurately.