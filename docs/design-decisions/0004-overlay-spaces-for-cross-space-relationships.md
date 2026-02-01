# ADR-0004: Cross-space relationships are stored in overlay spaces

## Status
Accepted

## Context

Aspen supports:
- space-level permissions only
- end-to-end encryption
- peer-to-peer replication
- multiple apps sharing one data substrate

Users must be able to:
- link items across spaces
- integrate data between apps
- keep relationships private when appropriate

A naïve approach—storing cross-space links inside content spaces—creates unavoidable privacy leaks:
- the existence of a link is visible to all space members
- link counts and timing leak metadata
- targets may be inferred even if encrypted

Aspen requires a way to express relationships **without leaking their existence**.

---

## Decision

All cross-space relationships are stored in **overlay spaces**.

An overlay space is a normal space with a special purpose:
- it contains relationship records (e.g. links)
- it references items in other spaces
- it has its own permissions and encryption key

Content spaces never contain cross-space relationship data.

---

## Definition

- **Content space**: stores records and blobs that constitute primary content.
- **Overlay space**: stores records that describe relationships between items in one or more content spaces.

Overlay spaces follow the same rules as content spaces:
- space-level permissions only
- end-to-end encrypted
- append-only authoritative data
- replicated via CRDTs

---

## Consequences

### Positive

- **No existence leakage**
  - Users without access to an overlay cannot infer that a relationship exists.
- **Clean permission boundaries**
  - Relationship visibility is explicit and intentional.
- **Composable integrations**
  - Apps can create private or shared relationship layers without modifying content.
- **Uniform model**
  - Overlays reuse existing space semantics; no special-case ACLs.

### Negative

- **Indirection**
  - Relationships are not colocated with content.
- **Client-side composition**
  - Backlinks and graph views must be assembled locally.
- **More spaces**
  - Users may accumulate multiple overlays.

These tradeoffs are accepted.

---

## Implications for links

- A link is a first-class record stored in an overlay space.
- Links reference items via opaque `Ref`s.
- Links have stable identities and append-only revision history.
- Re-sharing the same logical link creates a new link revision.

Links are never inferred or synthesized implicitly.

---

## Backlinks and discovery

- Backlinks for cross-space relationships are computed client-side.
- Only overlays the user can decrypt are considered.
- Content spaces remain unaware of overlays.

This preserves privacy while enabling rich local views.

---

## Sharing semantics

- Private overlays: visible only to the user.
- Shared overlays: visible to a defined group.
- Bridge overlays: shared by users who have access to multiple content spaces.

Sharing a content space does **not** implicitly share overlays.

---

## Alternatives considered

### Storing encrypted links in content spaces
Rejected:
- leaks existence and timing metadata
- breaks privacy guarantees
- undermines space-level permission model

### Per-record permissions
Rejected:
- complex UX
- difficult cryptographic enforcement
- reintroduces metadata leakage

### Implicit global graph
Rejected:
- violates privacy by default
- incompatible with local-first operation

---

## Related documents

- ADR-0003: Permissions are defined at the space level only
- `docs/architecture/data-model.md`
- `docs/architecture/security-model.md`

---

## Notes

Overlay spaces are essential to Aspen’s privacy model.

Any design that places cross-space relationships in content spaces is incorrect by definition.