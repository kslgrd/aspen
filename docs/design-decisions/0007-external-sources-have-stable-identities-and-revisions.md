# ADR-0007: External sources have stable identities and revision streams

## Status
Accepted (revised)

## Context

Aspen is designed to:
- work alongside existing tools
- integrate files, documents, and web content without replacing them
- preserve context and history over time
- keep authoritative data append-only and encrypted

Many items Aspen works with are external:
- files on disk
- URLs and web pages
- documents managed by other tools

These external sources may:
- change over time
- be re-shared intentionally
- disappear
- need to publish meaningful change events

Treating external sources as one-off imports or ephemeral captures fails to meet these requirements.

---

## Decision

External sources are modeled as **stable items with append-only revision streams**.

Each external source:
- has a stable identity
- accumulates revisions over time
- publishes events when re-shared or observed
- may persist summaries and/or snapshots depending on user intent

Re-sharing the same source is meaningful and always results in a new revision.

---

## Source identity

Each external source defines a **source key** that identifies “the same thing” over time.

Examples:
- Files: canonical path (plus volume identifier where applicable)
- URLs: canonicalized URL

The source key is used to:
- detect re-sharing of the same source
- append new revisions instead of creating duplicates
- associate history with a single logical item

Source keys are advisory but are expected to be stable.

---

## Reference vs Snapshot modes

External sources support two explicit modes that reflect user intent.

### Reference (default)
“Keep the link.”

- Aspen stores the canonical source identifier.
- On share or explicit refresh, Aspen may fetch the source and extract a **bounded summary**:
  - title
  - excerpt or extracted text
  - detected metadata (author, date, site)
  - fetch timestamp
  - extractor version
- The extracted summary is persisted via authoritative revisions (and blobs if needed).
- Full source bytes are **not** captured.

Properties:
- Searchable without refetching
- Low storage footprint
- No promise of permanence
- If the source disappears, that’s acceptable

Reference mode preserves *context*, not content.

---

### Snapshot
“Take snapshot.”

- Aspen captures a point-in-time snapshot of the source.
- Snapshot bytes are stored as encrypted blobs.
- A revision references both the snapshot blobs and extracted summaries.

Properties:
- Preserves content even if the source changes or disappears
- Higher storage cost
- Intended for archival or long-term reference

Snapshot mode preserves *content*, not just context.

---

## Revisions

A **source revision** represents an observation or capture of a source at a point in time.

Properties:
- append-only
- authored by a device
- belongs to exactly one source
- includes a reason (e.g. `reshare`, `filesystem_change`, `manual_snapshot`)
- may reference:
  - extracted summary blobs
  - snapshot blobs (snapshot mode only)

Revisions are authoritative events and participate in sync.

---

## File system sources

For file-backed sources:

- The filesystem remains authoritative for editing.
- Aspen records observations of files, not filesystem state.
- In reference mode:
  - Aspen stores metadata and extracted summaries only.
- In snapshot mode:
  - Aspen captures file contents into encrypted blobs.

Aspen does not attempt to synchronize the filesystem itself.

---

## URL and web sources

For URL-backed sources:

- The URL defines identity.
- Re-sharing always appends a revision.
- In reference mode:
  - Aspen persists a searchable summary from the last fetch.
- In snapshot mode:
  - Aspen persists a point-in-time capture.

Re-fetching is explicit, not continuous.

---

## Relationship to links

External sources are items, not links.

- Source revisions describe how a source changes over time.
- Links describe relationships between sources and other items.

Re-sharing a URL creates a new **source revision**, not a link mutation.

---

## Privacy and permissions

- External sources belong to a space.
- Revisions inherit the space’s permission and encryption model.
- Persisted summaries and snapshots are visible to all space members.

Aspen does not silently archive full content in reference mode.

---

## Consequences

### Positive

- **Storage control**
  - Users choose when content is archived.
- **Clear intent**
  - “Keep link” vs “Take snapshot” matches expectations.
- **Searchability**
  - Summaries are persistent and indexable.
- **Workflow compatibility**
  - Existing tools remain the source of truth.

### Negative

- **Staleness**
  - Reference-mode summaries may become outdated.
- **Complexity**
  - Multiple modes require clear UI and documentation.

These tradeoffs are accepted.

---

## Alternatives considered

### Always snapshot everything
Rejected:
- unacceptable storage growth
- unnecessary for most workflows
- discourages casual linking

### Never persist extracted content
Rejected:
- forces refetching
- breaks offline search
- undermines local-first guarantees

### Continuous background crawling
Rejected:
- privacy concerns
- unpredictable behavior
- excessive network activity

---

## Related documents

- ADR-0001: Authoritative data is append-only operations and blobs
- ADR-0006: Links are versioned entities with revision history
- `docs/architecture/data-model.md`
- `docs/architecture/sync-model.md`

---

## Notes

Aspen models **what was observed and when**, not **what exists now**.

Reference mode preserves context.
Snapshot mode preserves content.
Both are intentional.