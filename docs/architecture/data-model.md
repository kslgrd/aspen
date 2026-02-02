# Data Model

This document defines the core data model used by Aspen.

It describes the **conceptual entities**, their responsibilities, and the invariants they must obey. It does not define storage schemas, encodings, or APIs.

---

## Design goals

The data model must:

- Support local-first operation
- Enable end-to-end encrypted replication
- Work with CRDT-based synchronization
- Avoid permission and privacy leaks
- Support multiple apps sharing one substrate
- Allow history, revisioning, and diffing

These goals drive the shape of the model more than convenience.

---

## Core entities

### User
A **user** represents a long-lived human identity.

- Users have a stable cryptographic identity.
- Users may own multiple devices.
- Identity continuity matters more than anonymity.

User identity is global across all spaces.

---

### Device
A **device** represents a single installation of Aspen.

- Each device has its own key material.
- Devices act on behalf of a user.
- Devices sign operations.
- Devices may come and go without invalidating user identity.

Devices are the unit of authorship in the system.

---

### Space
A **space** is the primary unit of:

- visibility
- permissions
- encryption
- replication

Properties:

- Every authoritative object belongs to exactly one space.
- Spaces define who can read and/or write data.
- Permissions are coarse and space-level only.

Space roles:

- Viewer
- Editor
- Owner

There are no per-record or per-field permissions.

---

### Overlay Space
An **overlay space** is a special kind of space whose contents describe relationships between items in other spaces.

Key properties:

- Overlays use the same permission and encryption model as spaces.
- Overlays prevent leaking the existence of cross-space relationships.
- Users without access to an overlay cannot infer that a relationship exists.

Overlay spaces are how Aspen models:

- cross-space links
- private relationship layers
- shared connection layers

---

### Record
A **record** is a typed, app-owned object stored in a space.

Properties:

- Records are namespaced by app.
- Records have stable identifiers.
- Records may reference other records via `Ref`s.
- Records are immutable; changes create new revisions.

Examples:

- Note
- Opportunity
- Graph layout
- Decision
- Task

Aspen does not interpret record semantics.

---

### Record Revision
A **record revision** represents a change to a record over time.

Properties:

- Append-only
- Authored by a device
- Belongs to exactly one record
- May reference new blobs
- May carry metadata describing the reason for change

Revisions allow:

- history
- diffing
- sync events
- conflict resolution

“Updating” a record always means appending a revision.

---

### Blob
A **blob** is encrypted binary content.

Properties:

- Content-addressed
- Immutable
- Stored separately from records
- Referenced by records or revisions

Examples:

- File snapshots
- Screenshots
- Web captures
- Large text payloads

Blobs are never modified in place.

---

### Ref
A **Ref** is a stable reference to a record.

A Ref includes:

- space identifier
- record identifier
- record type

Refs are opaque identifiers.
They do not imply access or visibility.

---

### Link
A **link** is a relationship between two Refs.

Properties:
- Stored only in overlay spaces
- Directional or non-directional (depending on kind)
- Has a stable identity
- Is versioned via revisions

Links are first-class entities, not derived edges.

---

### Link Revision
A **link revision** represents a change to a link.

Properties:

- Append-only
- Belongs to a single logical link
- May update metadata (label, weight, notes, etc.)
- Emits change events even when the same link is “shared again”

Sharing the same logical link multiple times creates new revisions, not duplicates.

---

## Identity and source keys

Some records represent external sources (e.g. files or URLs).

These may define a **source key**:

- canonical file path
- canonical URL

Source keys are used for:

- deduplication
- detecting “the same thing shared again”
- appending new revisions instead of creating new records

Source keys are advisory, not authoritative.

---

## Authoritative vs derived data

Only the following are authoritative and replicated:

- record revisions
- link revisions
- blobs
- space membership events

Everything else is derived:

- materialized record state
- search indexes
- vector embeddings
- graph projections

Derived data may be discarded and rebuilt at any time.

---

## Invariants

The data model enforces the following invariants:

- All authoritative data is append-only.
- Every authoritative object belongs to exactly one space.
- All replicated data is encrypted.
- Cross-space relationships never leak existence.
- History is preserved; mutation is modeled as revision.
- SQLite is never a source of truth.

Breaking any of these invariants indicates a design error.
