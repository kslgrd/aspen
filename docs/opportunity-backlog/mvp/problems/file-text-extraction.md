# Problem: File text extraction (reference mode)

**Problem statement:**  
When I drag a local file into Aspen, I want it to become searchable and skimmable without copying the file,
so I can keep my files where they are, but extracting text reliably across file types (PDF, Office, images)
is inconsistent and can create performance, privacy, and UX edge cases.

**Evidence**

- MVP requires file references (no file bytes) and extracted text “where possible.” (See [`mvp-overview.md`](../mvp-overview.md).)
- File identity is already flagged as tricky; extraction adds another axis of ambiguity (what if content changes? what if extraction fails?).
- macOS has multiple system frameworks that could help (Quick Look, Spotlight importers), but relying on them may reduce portability.

---

## Opportunities

- How might we define a small, portable extraction surface for MVP that yields useful search without turning into a full document processing system?

---

## Hypotheses

- We believe supporting a limited set of high-signal file types first (plain text, PDF) will achieve useful MVP search
  without high complexity.
- We believe capturing extraction provenance (what extractor, when, what file metadata) will achieve debuggability and correctness
  when file bytes are not stored.

**Success metrics**

- ≥80% of dragged files yield some searchable text (in a representative corpus).
- Extraction failures are explicit (file still appears in timeline; search misses are explainable).
- No file bytes are persisted in MVP (beyond OS caches outside Aspen’s control).

---

## Possible Concepts (MVE)

- **Hypothesis:** Narrow file type support first  
  **MVE:** Implement extraction for plaintext + PDF only; measure usefulness on a 30-file corpus  
  **Effort:** medium  
  **Signal:** extraction coverage + search usefulness

- **Hypothesis:** Extraction provenance  
  **MVE:** Add extractor metadata to derived views and confirm it helps debugging  
  **Effort:** low  
  **Signal:** faster diagnosis of “why isn’t this searchable?”

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- This is separate from stable file identity across moves/renames/devices. See `path-identity.md`.
- We need to decide whether extracted text is stored authoritatively (encrypted blob) or only derived; this impacts rebuildability and privacy posture.

## Links

- [`mvp-overview.md`](../mvp-overview.md)
- [`ingestion.md`](../spec/ingestion.md)
- [`path-identity.md`](path-identity.md)
