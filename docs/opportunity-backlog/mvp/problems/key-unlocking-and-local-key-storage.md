# Problem: Key unlocking and local key storage UX

**Problem statement:**  
When I use Aspen day-to-day on a single device, I want encryption to be real but unobtrusive,
so I can trust that my data is private without constantly "doing security," but key storage
and unlock behavior can easily become either insecure by default or too annoying to use.

**Evidence**

- The MVP explicitly requires: "Key management must be real, not mocked." (mvp-overview)
- The key-management model lists multiple unlock approaches and open questions (OS secure storage vs passphrase; background behavior when locked).
- Even single-device behavior sets expectations and shapes future migration to multi-device and collaboration.

---

## Opportunities

- How might we provide real, space-scoped encryption with a low-friction unlock experience, without implying guarantees we can’t keep or baking in unsafe defaults?

---

## Hypotheses

- We believe using OS secure storage by default (when available) will achieve a transparent unlock experience
  without compromising the E2EE model.
- We believe supporting an explicit "lock Aspen" action plus a session-based unlock will achieve clarity
  about security state without constant prompts.
- We believe a passphrase fallback (when OS secure storage is unavailable) will achieve portability
  while keeping the model honest.

**Success metrics**

- Users can complete a normal capture/search/link flow without encountering cryptography concepts.
- The app clearly communicates when the store is locked/unlocked (no silent failure modes).
- No plaintext key material is written to disk outside of OS-protected storage.

---

## Possible Concepts (MVE)

- **Hypothesis:** OS secure storage default
  **MVE:** Implement identity/device/space key storage using OS keychain APIs on one platform
  **Effort:** medium
  **Signal:** end-to-end flow works without prompts while remaining "real"

- **Hypothesis:** Session-based unlock + explicit lock
  **MVE:** Add "locked" state transitions and verify background behavior is sane
  **Effort:** medium
  **Signal:** no confusing "why can’t I search?" moments; clear user control

- **Hypothesis:** Passphrase fallback
  **MVE:** Implement a passphrase-encrypted local KEK and test cold-start + restart behavior
  **Effort:** medium
  **Signal:** consistent behavior when keychain is unavailable

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- This is the first place where "real security" intersects UX in MVP.
- We should be explicit about what locking means (e.g., search disabled? timeline still visible?)
  to avoid drifting into insecure convenience.

## Links

- [mvp-overview.md](../mvp-overview.md)
- [key-management.md](../../../architecture/key-management.md)
- [security-model.md](../../../architecture/security-model.md)
