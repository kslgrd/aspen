# Sync Model

This document describes Aspen’s synchronization model at a conceptual level.

It explains how data moves between devices, how conflicts are resolved, and what guarantees the system provides under partial connectivity.

This is a model of **replication semantics**, not a network protocol specification.

---

## Design goals

Aspen’s sync model is designed to:

- work without any required server
- support offline-first usage
- allow many peers to collaborate in shared spaces
- preserve privacy through end-to-end encryption
- converge deterministically under concurrent edits
- tolerate unreliable and intermittent connectivity

Correctness and clarity take precedence over minimizing implementation complexity.

---

## Local-first as a foundation

Aspen is local-first by construction:

- every device maintains a complete local copy of the data it is authorized to access
- all writes occur locally first
- synchronization is replication, not remote execution

A device must function fully without network access.

---

## Replication unit

The fundamental unit of replication is **authoritative data**, consisting of:

- append-only operation logs (CRDT ops)
- encrypted blobs
- space membership events

Derived data (SQLite views, indexes, layouts) is never replicated.

---

## CRDT-based convergence

### Why CRDTs

Aspen uses CRDT-based approaches to ensure:

- concurrent edits can be merged without coordination
- devices can operate independently while offline
- replicas eventually converge when communication resumes

Operational transformation (OT) is explicitly avoided due to:

- reliance on centralized coordination
- complexity under arbitrary peer topologies

---

### Scope of CRDTs

CRDT semantics apply to:

- record revisions
- link revisions
- text content (where applicable)
- pointers to “current” revisions

CRDTs are used to resolve **state**, not intent.

---

## Append-only logs

### Operation logs

Each space maintains an append-only log of operations:

- operations are immutable
- operations are ordered causally, not globally
- operations are signed by device keys
- operations are encrypted under the space key

Examples of operations:

- append record revision
- append link revision
- add member to space
- revoke member role

---

### Why append-only

Append-only logs provide:

- auditability
- deterministic replay
- easy anti-entropy reconciliation
- compatibility with content-addressed storage

Deletion is modeled as a semantic operation, not physical removal.

---

## Blobs and large data

Blobs are synchronized separately from operation logs.

Properties:

- content-addressed
- encrypted
- immutable

Synchronization behavior:

- operations reference blobs by hash
- blobs may be fetched lazily
- missing blobs can be requested on demand

This allows:

- fast metadata sync
- deferred large transfers
- partial replication based on need

---

## Peer-to-peer topology

### No required servers

Aspen does not require any central server to function.

Peers may:

- discover each other directly (e.g. LAN, known addresses)
- communicate opportunistically
- remain disconnected for long periods

---

### Optional relays

Relays may exist to assist with:

- peer discovery
- NAT traversal
- message forwarding

Relays:

- cannot read data
- cannot enforce permissions
- cannot modify authoritative state

They are transport-only.

---

## Group synchronization

### Announcement vs transfer

To scale to many peers in shared spaces, Aspen distinguishes between:

- **announcements**: lightweight signals about state (e.g. “I have revision X”)
- **transfers**: direct, stateful exchanges of missing data

Announcements may be broadcast.
Transfers are always peer-to-peer and stateful.

---

### Per-peer state

Each device maintains per-peer sync state:

- last known operation identifiers
- last known space versions
- pending blob requests

This state is local-only and not replicated.

---

## Anti-entropy reconciliation

Aspen assumes messages may be lost or delayed.

Therefore:

- real-time streams are opportunistic
- periodic reconciliation is required

Reconciliation involves:

- exchanging summaries of known state
- requesting missing operations
- requesting missing blobs

Anti-entropy is what guarantees eventual consistency.

---

## Conflict handling

### Record and link updates

Conflicts arise when multiple devices update the same logical entity concurrently

Resolution:

- CRDT rules determine the “current” revision pointer
- all revisions remain in history
- conflicts may be surfaced in UI, but state converges deterministically

---

### Semantic conflicts

Aspen does not attempt to resolve semantic conflicts automatically.

Examples:

- two users editing the same paragraph in incompatible ways
- competing interpretations of link metadata

These are application-level concerns.

---

## Membership and permission sync

Space membership changes are synchronized like any other operation.

Properties:

- membership events are append-only
- membership changes are signed
- peers enforce permissions cryptographically

Unauthorized operations are rejected during replication.

---

## Sync triggers

Synchronization may be triggered by:

- peer discovery
- explicit user action
- background reconciliation
- local changes

No trigger is authoritative; they are additive.

---

## Failure modes

Aspen is designed to tolerate:

- network partitions
- peer churn
- partial replication
- delayed delivery
- duplicate messages

It does **not** guarantee:

- immediate consistency
- total ordering across all peers
- conflict-free semantics at the application level

---

## Sync invariants

The sync model enforces the following invariants:

- Local writes never depend on remote availability
- All replicated data is encrypted
- All operations are immutable
- Replicas converge given sufficient communication
- Derived state is never synchronized

Violating these invariants indicates a design error.
