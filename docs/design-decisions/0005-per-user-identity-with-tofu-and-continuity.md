# ADR-0005: Per-user identity with TOFU and continuity

## Status
Accepted

## Context

Aspen supports:
- peer-to-peer collaboration
- end-to-end encryption
- no required backend services
- long-lived shared spaces
- frictionless sharing

To collaborate safely, peers must be able to answer:
- *Who am I sharing with?*
- *Is this still the same person I trusted before?*

Centralized identity providers are incompatible with Aspen’s goals.
Per-space identities create unnecessary friction and trust duplication.

A per-user identity model is required, but it must remain usable without a trusted authority.

---

## Decision

Aspen uses a **per-user cryptographic identity**, established via **Trust On First Use (TOFU)** and maintained through **identity continuity**.

- Each user has a long-lived identity key.
- Identity is global across all spaces and apps.
- The first time an identity is encountered, it is accepted on faith (TOFU).
- Once accepted, the identity key is pinned locally.
- Subsequent interactions are verified against the pinned identity.

Identity continuity is enforced explicitly. Silent trust resets are not allowed.

---

## Identity scope

- Identity is **per user**, not per space.
- A user’s identity remains stable across:
  - devices
  - spaces
  - apps

This allows:
- verify once, trust many
- consistent mental models of collaboration
- reuse of trust across contexts

---

## Device model

- Each device has its own cryptographic keys.
- Devices act on behalf of a user.
- Device keys are authorized by being signed by the user’s identity key.

A device is trusted if:
- its device key is valid, and
- it is signed by the pinned user identity.

This enables:
- multiple devices per user
- device rotation without re-verification
- recovery from device loss without identity loss

---

## Identity continuity guarantees

Once an identity is pinned:

- If the identity key remains the same, trust continues automatically.
- If the identity key changes, this is treated as a **security event**.

Default behavior on identity change:
- surface a clear warning
- block sensitive actions involving that identity
- require explicit user re-verification

Identity changes are never silently accepted.

---

## UX implications

The security model must be visible but not intimidating:

- First-time sharing may proceed without verification.
- Verification is encouraged but optional.
- Identity changes are surfaced clearly and require user intent.

Users should never be asked to understand cryptographic details to make safe decisions.

---

## Security properties

This model provides:

- Protection against impersonation *after* first contact
- Stable trust relationships over time
- No reliance on centralized identity services
- Compatibility with fully peer-to-peer operation

It does **not** protect against:
- impersonation at first contact
- compromised local devices
- malicious collaborators with legitimate access

These are accepted tradeoffs.

---

## Relationship to encryption and permissions

- Identity keys authenticate authorship of operations.
- Space membership is bound to authorized device keys.
- Permission enforcement relies on both identity and role.

Identity answers *who*; spaces answer *what*.

---

## Alternatives considered

### Per-space identities
Rejected:
- repeated verification burden
- fractured trust model
- poor UX for multi-space collaboration

### Centralized identity provider
Rejected:
- required backend dependency
- trust concentration
- incompatibility with privacy goals

### Anonymous or ephemeral identities
Rejected:
- no continuity
- poor collaboration UX
- difficult recovery and auditing

---

## Related documents

- ADR-0003: Permissions are defined at the space level only
- ADR-0004: Cross-space relationships are stored in overlay spaces
- `docs/architecture/security-model.md`
- `docs/architecture/data-model.md`

---

## Notes

Per-user identity is foundational to safe, low-friction collaboration in Aspen.

If this decision is reversed, most sharing and trust assumptions must be revisited.