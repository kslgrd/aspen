# MVP System Decomposition

This document decomposes the MVP into buildable systems and defines responsibilities and boundaries.

It is derived from:

- [`mvp-overview.md`](../mvp-overview.md)
- [`overview.md`](../../../architecture/overview.md)
- [`0011-technical-foundations.md`](../../../design-decisions/0011-technical-foundations.md)

---

## MVP invariants (non-negotiable)

1) **Authoritative data is encrypted end-to-end**
- All authoritative data is encrypted at rest under space keys.
- Authoritative data is append-only (ops + blobs). (ADR-0001)

2) **SQLite is derived and disposable**
- SQLite is only projections + indexes.
- Deleting SQLite and restarting reconstructs the same timeline and search results (modulo non-authoritative ranking). (ADR-0002)

3) **Local-first**
- All primary flows (capture, browse timeline, search) work offline.

4) **Platform boundary**
- Apps never bypass the platform to read/write authoritative storage or keys. (See [`overview.md`](../../../architecture/overview.md).)

---

## Systems (MVP)

### 1) macOS App (UI + OS integration)

**Responsibilities**
- Capture surfaces:
  - paste URL when the app is active
  - drag-and-drop local files
- Primary UI:
  - a timeline view
  - an incremental free-text search box that filters the timeline
- Present “locked/unlocked” state clearly (when keys are not available).

**Out of scope**
- background capture (share sheets, extensions)
- collaboration UX
- advanced query languages

**Spec**
- [`macos-app.md`](macos-app.md)

**Related problems**
- [`capture-surfaces.md`](../problems/capture-surfaces.md)
- [`key-unlocking-and-local-key-storage.md`](../problems/key-unlocking-and-local-key-storage.md)

---

### 2) Platform Kernel (portable library)

**Responsibilities**
- Identity bootstrap (single user)
- Device identity (single device)
- Inbox space bootstrap (space keys)
- Authoritative storage:
  - append-only ops
  - encrypted blob storage
- Derived state:
  - projections into SQLite
  - full-text search indexes (in SQLite, or alongside it)
- Subscriptions / change notifications (app updates reactively).

**MVP constraints**
- Single device, no sync transport required — but the storage and semantics must not block sync later.

**Spec**
- [`platform.md`](platform.md)

**Related problems**
- [`platform-api-and-ffi-boundary.md`](../problems/platform-api-and-ffi-boundary.md)
- [`record-schema-and-versioning.md`](../problems/record-schema-and-versioning.md)
- [`crdt-surface-and-conflict-semantics.md`](../problems/crdt-surface-and-conflict-semantics.md)

---

### 3) Authoritative Store (ops + blobs)

**Responsibilities**
- Persist encrypted operation log entries per space.
- Persist encrypted blobs (e.g., extracted summaries) content-addressed.
- Provide atomic append + blob reference semantics.
- Provide replay for rebuild.

**Spec**
- [`storage-and-indexes.md`](storage-and-indexes.md)

---

### 4) Derived State (SQLite projections + search)

**Responsibilities**
- Materialize “timeline events” from ops.
- Maintain a local-only search index over:
  - extracted text (URL summaries, file extracted text)
  - metadata (title, type, timestamps, paths, URLs)
- Rebuild deterministically from authoritative data.

**Spec**
- [`storage-and-indexes.md`](storage-and-indexes.md)

**Related problems**
- [`sqlite-projection-and-rebuild.md`](../problems/sqlite-projection-and-rebuild.md)

---

### 5) Ingestion Pipelines (reference-mode capture)

**Responsibilities**
- URL capture:
  - fetch page
  - extract bounded summary (and minimal metadata)
  - store summary as encrypted blob
  - append timeline event (new capture or re-share)
- File capture:
  - persist canonical path + metadata
  - extract text where possible
  - append timeline event (new capture or re-share)

**Spec**
- [`ingestion.md`](ingestion.md)

**Related problems**
- [`url-normalization.md`](../problems/url-normalization.md)
- [`path-identity.md`](../problems/path-identity.md)
- [`url-fetch-and-summary-extraction.md`](../problems/url-fetch-and-summary-extraction.md)
- [`file-text-extraction.md`](../problems/file-text-extraction.md)

---

## Cross-cutting concerns (MVP)

### Observability and debugging
- Structured logs (at least for capture + indexing + rebuild).
- A “rebuild derived state” action (dev-only is fine) to prove disposability.

### Safety rails
- Never leak decrypted authoritative payloads into derived state unless explicitly allowed by the threat model.
- Avoid caching plaintext blob contents on disk (beyond SQLite projections; clarify what’s acceptable). See [`sqlite-projection-and-rebuild.md`](../problems/sqlite-projection-and-rebuild.md).
