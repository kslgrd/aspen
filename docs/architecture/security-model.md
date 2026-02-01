# Security Model

This document describes Aspen’s security model at a conceptual level.

It defines:

- what Aspen is designed to protect,
- what it explicitly does not attempt to protect against,
- and the trust assumptions built into the system.

This is a **model of guarantees**, not an implementation guide.

---

## Design goals

Aspen’s security model is designed to:

- protect user data from service operators and infrastructure
- allow collaboration without trusting a central authority
- preserve identity continuity over time
- minimize accidental privacy leaks
- remain understandable to users without security expertise

Aspen prioritizes **clarity and correctness** over maximal theoretical security.

---

## Threat model

### In scope

Aspen is designed to protect against:

- curious or malicious service operators
- compromised sync relays
- passive network observers
- accidental data exposure through sync or integration
- unauthorized peers attempting to inject data
- identity impersonation after initial trust is established

### Out of scope

Aspen does **not** attempt to protect against:

- a fully compromised local device
- malicious collaborators who already have legitimate access
- users intentionally sharing data they should not
- retroactive revocation of data already replicated
- global traffic analysis or nation-state adversaries

These constraints are intentional.

---

## Trust assumptions

Aspen makes the following assumptions explicit:

- The user controls their own devices.
- The user may choose to trust other users.
- First contact between users is performed on faith (TOFU).
- Trust, once established, must be stable and verifiable over time.

No always-on trusted third party is assumed.

---

## Identity model

### Per-user identity

- Each user has a long-lived cryptographic identity.
- Identity is global across all spaces and apps.
- Identity keys change rarely and intentionally.

Aspen treats a user identity as the anchor of trust.

---

### Trust On First Use (TOFU)

- The first time a user encounters another user’s identity, it is accepted on faith.
- The identity key is pinned locally.
- Subsequent interactions are verified against the pinned identity.

TOFU is a conscious tradeoff: it minimizes friction, but allows first-contact impersonation.

Aspen mitigates this with identity continuity.

---

### Identity continuity

Once an identity is pinned:

- If the identity key remains the same, trust continues.
- If the identity key changes, this is treated as a **security event**.

Default behavior on identity change:

- surface a warning to the user
- block sensitive actions involving that identity
- require explicit re-verification

Silent trust resets are not allowed.

---

## Device trust

### Device keys

- Each device generates its own cryptographic keys.
- Devices act on behalf of a user.
- Devices sign all authored operations.

### Device authorization

- Device keys are authorized by being signed by the user’s identity key.
- A device is trusted if:
  - its device key is valid, and
  - it is signed by a pinned user identity.

This allows:

- device rotation
- multiple devices per user
- continuity without repeated verification

---

## Encryption model

### End-to-end encryption

All authoritative data is encrypted end-to-end:

- record revisions
- link revisions
- blobs
- membership events

No service or relay can read user data.

---

### Space keys

- Each space has its own encryption key material.
- Space keys encrypt all authoritative data within that space.
- Access to a space key implies access to all content in that space.

This enforces space-level permissions cryptographically.

---

### Key distribution

- Space keys are never shared in plaintext.
- Invites distribute encrypted key material to authorized devices.
- Devices must prove possession of the corresponding private keys.

Key distribution does not require a trusted server.

---

## Permissions and enforcement

### Space-level permissions

Permissions are defined only at the space level:

- Viewer: read access
- Editor: read and write access
- Owner: administrative access

There are no per-record permissions.

---

### Cryptographic enforcement

- All operations are signed by device keys.
- Peers accept operations only if:
  - the device key is authorized for the space, and
  - the operation type is permitted by the device’s role.

Unauthorized operations are rejected during replication.

---

## Cross-space privacy

### Overlay spaces

Cross-space relationships are modeled via overlay spaces.

Security properties:

- relationships are encrypted under the overlay’s key
- users without access to an overlay cannot infer the existence of links
- content spaces remain unaware of cross-space relationships

This prevents accidental metadata leakage.

---

## History and revocation

### Append-only history

- All authoritative data is append-only.
- Changes are modeled as revisions.
- History is preserved for audit, diffing, and sync correctness.

---

### Revocation semantics

When access is revoked:

- future content is protected
- previously replicated content cannot be clawed back

Aspen does not attempt to provide retroactive secrecy.

This limitation is surfaced clearly in the user experience.

---

## Recovery and loss

### Key loss

If a user loses their identity key:

- their identity continuity is broken
- collaborators must explicitly re-verify a new identity

Recovery mechanisms may exist, but identity resets are treated as meaningful events.

---

## Metadata leakage

Aspen acknowledges unavoidable metadata leakage:

- timing of sync
- approximate data volume
- number of peers in a space

The system minimizes metadata exposure but does not claim perfect secrecy.

---

## Security boundaries summary

Aspen draws clear boundaries:

- Infrastructure is not trusted with data.
- Users explicitly trust other users.
- Spaces define visibility and access.
- Overlays prevent relationship leakage.
- Identity continuity preserves trust over time.

Anything outside these boundaries is a deliberate tradeoff.