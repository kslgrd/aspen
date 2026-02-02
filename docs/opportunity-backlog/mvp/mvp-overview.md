# MVP Overview

This document describes the **minimum viable product** for Aspen. The goal is to validate Aspen’s core architectural invariants while delivering something that is immediately usable.

Per [ADR 0011](../../design-decisions/0011-technical-foundations.md) we'll start by establishing the core platform, as well as an extremely basic macOS app.

---

## MVP Goal

Prove that we can capture URL's and files in a **private, local-first connective layer** over the things you already use, without compromising:

- correctness
- privacy
- future multi-device and multi-app expansion

---

## Default mental model

### The Inbox

The MVP revolves around a single, always-private space:

**Inbox**

- Created automatically on first launch
- Private by default
- Never shared
- Cannot be deleted
- Acts as the capture surface for everything

Users should never have to think about spaces in the MVP, other spaces are a future concern.

---

## Supported item types (MVP)

### URLs
- Stored in **reference mode only**
- On add:
  - fetch page
  - extract bounded summary
  - persist summary as encrypted blob
- Re-adding the same URL:
  - appends a new revision event
  - publishes intent (“this matters again”)

### Local files
- Stored as **references**, not copies
- Persist:
  - canonical path
  - metadata
  - extracted text where possible
- No file bytes captured in MVP

---

## Primary UI: Add items + Timeline + Search

The entire MVP UI can be described as:

> **A timeline, filtered by a powerful free-text search box.**

### Adding items

- Drag and drop for files
- Paste a URL (command + v) when the app window is active

### Timeline

- Shows items in reverse chronological order
- Includes:
  - URLs
  - file references
- Each entry represents an **event** (creation, reshare, update), not a mutable object
- Items may appear multiple times over time as revisions occur

The timeline is the ground truth view of “what happened.”

---

### Search

- Free-text search filters the timeline
- Search is incremental and fast
- Search operates over:
  - extracted summaries from URLs/files
  - basic metadata (title, type, timestamps)

Search is:
- local-only
- derived
- non-authoritative

No advanced query language is required for MVP.

---

## What the MVP deliberately avoids

The MVP **does not include**:

- linking
- shared spaces
- collaboration
- permissions UI
- device pairing
- snapshot/archive mode
- background crawling
- auto-refreshing URLs
- AI-assisted actions
- complex graph UI

All of these are designed for, none are required to validate the core.

---

## Encryption and storage (MVP scope)

- All authoritative data is encrypted end-to-end
- Encryption is space-scoped (Inbox has its own keys)
- SQLite is used only for:
  - projections
  - search indexes
- SQLite is disposable and rebuildable

Key management must be real, not mocked.

---

## Identity and devices (MVP scope)

- Single user
- Single device
- Real identity key
- Real device key
- No recovery flows yet

The model must *support* more later, but not implement it yet.

---

## Success criteria

The MVP is successful if:

- Deleting SQLite and restarting reconstructs the same timeline
- Re-adding the same URL produces a new timeline event
- Linking and re-linking produces visible history
- Search works offline
- All data remains private and local
- No architectural shortcuts are taken that block future expansion

Performance and polish are secondary.

---

## What this MVP proves

If this MVP works, it proves that:

- Aspen’s append-only model is viable
- “Reference-first” external sources are useful
- A timeline-first UI is sufficient and powerful
- The platform can support multiple apps later
- Privacy and usability are not in conflict

Only after this is proven does it make sense to build:
- linking
- collaboration
- spatial canvases
- additional apps
- sync infrastructure

---

## Non-goals

- Being competitive with existing tools
- Supporting every workflow
- Appearing “finished”
- Solving collaboration yet

The MVP is a foundation, not a destination.
