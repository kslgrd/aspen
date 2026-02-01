# Deletion Semantics

This document defines what “deletion” means in Aspen.

Aspen is built on append-only, replicated, end-to-end encrypted data. As a result, deletion cannot mean “erase all traces everywhere.” This document makes those constraints explicit and defines the semantics Aspen supports instead.

This is a **conceptual model**, not an implementation spec.

---

## Goals

Deletion in Aspen must:

- be understandable to users
- respect append-only and CRDT constraints
- avoid lying about guarantees
- preserve auditability where appropriate
- minimize unintended data leakage
- work consistently in shared spaces

---

## Core principle

**Deletion in Aspen is semantic, not physical.**

Deletion expresses *intent* (“this should no longer be considered active or visible”), not an attempt to rewrite history or reclaim all storage.

---

## What deletion means

When an item is deleted in Aspen:

- a **deletion operation** is appended to the space’s authoritative log
- the item is marked as deleted in derived views
- the item no longer appears in normal UI flows
- existing links to the item are treated as inactive or historical

The original data remains part of the replicated history.

---

## What deletion does *not* mean

Deletion does **not**:

- remove historical operations from the log
- guarantee storage reclamation on all devices
- revoke access to data already replicated to authorized peers
- prevent offline peers from retaining old data
- silently alter the past

Any UI or documentation must avoid implying these guarantees.

---

## Deletion vs redaction

Aspen distinguishes between two concepts:

### Deletion
- expresses that an item is no longer active
- preserves history
- reversible in principle (via additional operations)

### Redaction
- attempts to limit future access to sensitive content
- may involve:
  - encrypting new content under rotated keys
  - suppressing display of historical payloads
- **cannot** guarantee full erasure in a P2P system

Redaction is a stronger signal than deletion, but still forward-looking.

---

## Deleting in shared spaces

In a shared space:

- deletion is visible to all space members
- deletion intent applies uniformly
- permissions determine *who may delete*, not *what deletion means*

A deleted item remains part of the shared history, but is no longer part of the active working set.

---

## Relationship to permissions and roles

- Editors may delete items they can write.
- Owners may:
  - delete items
  - rotate space keys
  - remove members

Deletion does not bypass permission rules.

---

## External sources (ADR-0007)

Deleting an external source item:

- removes the item from active views
- does not modify or delete the underlying file or URL
- does not imply deletion of snapshots already captured

For snapshot mode:
- snapshots remain in history unless explicitly redacted via key rotation

For reference mode:
- persisted summaries remain historical records

---

## Links and relationships (ADR-0006)

Deleting an item:

- does not delete link history
- causes links to reference a deleted target
- links may be:
  - hidden
  - marked as historical
  - surfaced only in audit views

Deleting a link:
- appends a deletion revision to that link’s history
- does not remove prior link revisions

---

## User experience principles

Deletion UX must:

- be explicit about scope (“delete from this space”)
- avoid implying global erasure
- distinguish delete vs remove-from-view
- avoid panic-inducing language
- provide “soft delete” affordances where appropriate

Example language:
- “Remove from this space”
- “Hide from view”
- “Delete (keeps history)”

Avoid:
- “Erase permanently”
- “Delete forever”

---

## Storage and compaction

Deletion does not immediately reclaim storage.

Storage reclamation may occur via:
- local compaction
- snapshot pruning
- user-controlled cleanup policies

Compaction is a **separate concern** and must never change the logical history of the space.

---

## Threat model implications

Deletion does not protect against:
- malicious collaborators
- compromised devices
- already-exported data

This must be stated plainly.

---

## Non-goals

- Guaranteed global erasure
- Legal compliance semantics (e.g., “right to be forgotten”)
- Invisible history rewriting
- Centralized enforcement

Aspen prioritizes correctness and honesty over false assurances.

---

## Open questions

- Should “undo delete” be a first-class concept or just another operation?
- How should deleted items appear in graph and spatial views?
- Should deletion events be suppressible in derived intelligence?
- How should redaction be surfaced in UI?

These questions should be resolved alongside application UX design.

---