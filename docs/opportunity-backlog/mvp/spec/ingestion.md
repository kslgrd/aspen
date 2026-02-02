# Ingestion Pipelines (MVP Spec)

MVP item types (see [`mvp-overview.md`](../mvp-overview.md)):
- URLs (reference mode only; store extracted summary as encrypted blob)
- Local files (reference mode only; store path + metadata + extracted text where possible)

This document specifies the stages and contracts for ingestion in MVP.

---

## Capture contract (applies to all surfaces)

All capture surfaces should call a single platform capture API and share semantics.

At minimum, a capture request includes:
- item kind: URL | file
- raw input: raw URL or file path(s)
- surface metadata:
  - surface: paste | drag-drop
  - timestamp (device-local)
  - (optional) app focus context

Capture produces:
- a new authoritative event (append-only)
- derived state updates (timeline + search)

See: [`capture-surfaces.md`](../problems/capture-surfaces.md)

---

## URL ingestion (reference mode)

### Stages

1) Normalize URL identity (dedupe key)
- Store the raw URL always.
- Compute a normalized form for identity / re-share detection.
  - [`url-normalization.md`](../problems/url-normalization.md)

2) Fetch page
- Fetch the URL content using a robust client (timeouts, redirects, user-agent policy).

3) Extract metadata + bounded summary
- Title
- Optional: site name, published date (best-effort)
- A bounded text summary for indexing and offline viewing

4) Persist authoritative data
- Store extracted summary as an encrypted blob.
- Append an operation representing “URL captured” (new item) or “URL re-shared” (new event).

5) Update derived indexes
- Project into SQLite (timeline row + search index).

Tracked unknowns:
- [`url-fetch-and-summary-extraction.md`](../problems/url-fetch-and-summary-extraction.md)

---

## File ingestion (reference mode)

### Stages

1) Resolve file identity (dedupe key)
- Persist canonical path + stable metadata.
- Decide how (or whether) to track identity beyond path for future rename/move.
  - [`path-identity.md`](../problems/path-identity.md)

2) Extract metadata
- filename
- filesystem timestamps
- size
- file type / UTType

3) Extract text where possible
- Best-effort extraction (PDF, plain text, etc.).
- Do not store file bytes in MVP.

4) Persist authoritative data
- Append an operation representing “file referenced” (new item) or “file re-shared” (new event).
- If extracted text is stored authoritatively, store it as an encrypted blob.

5) Update derived indexes
- Project into SQLite (timeline row + search index).

Tracked unknowns:
- [`file-text-extraction.md`](../problems/file-text-extraction.md)
