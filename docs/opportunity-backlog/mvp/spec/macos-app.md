# macOS App (MVP Spec)

The MVP app is an “Inbox + Timeline + Search” UI (see [`mvp-overview.md`](../mvp-overview.md)) with minimal capture surfaces:

- Paste a URL when the app window is active
- Drag-and-drop local files

This document defines the UX and platform interactions required for MVP.

---

## UX flows (MVP)

### First launch

Goal: app becomes usable without configuration while keeping security “real”.

- Platform bootstraps identity + device + Inbox space.
- App shows a timeline (empty state) and a search box.
- If keys are locked/unavailable, the app shows a clear locked state and offers unlock.

Related:
- [`key-unlocking-and-local-key-storage.md`](../problems/key-unlocking-and-local-key-storage.md)

### Capture a URL (paste)

- User focuses the app window.
- User presses Command+V.
- App attempts to parse clipboard as URL.
  - If valid URL: call platform `capture_url(...)`
  - If not a URL: no-op or show subtle validation (TBD)
- Timeline updates with a new event row (optimistic UI allowed if platform supports it).

### Capture file(s) (drag/drop)

- User drags files onto the app window.
- App calls platform `capture_file(path...)` for each file (or batch API).
- Timeline updates with new events.

### Search

- Search box filters the timeline incrementally as the user types.
- Search results come from derived local indexes; the UI should avoid implying that search is authoritative.

---

## UI requirements (MVP)

- One window: search box + timeline list.
- Timeline rows show:
  - title (URL title or file name)
  - type indicator (URL vs file)
  - timestamp (capture time)
- Clicking a row is optional for MVP; if supported, show a minimal detail view (metadata + extracted summary/text).

---

## Open questions (tracked as problems)

- Capture surface consistency and semantics
  - [`capture-surfaces.md`](../problems/capture-surfaces.md)
- Locked/unlocked UX (key storage, session behavior)
  - [`key-unlocking-and-local-key-storage.md`](../problems/key-unlocking-and-local-key-storage.md)
