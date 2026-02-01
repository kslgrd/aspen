# Key Management

This document defines Aspen’s key-management model at a conceptual level.

It covers:
- what keys exist
- what they protect
- where keys are stored
- how keys are unlocked
- how keys move between devices
- what happens when keys are lost

This is **not** a crypto spec. Algorithms, KDF parameters, and protocol wire formats are out of scope.

---

## Goals

Key management must support:

- **End-to-end encryption**: infrastructure cannot read user data.
- **Local-first**: a single device works fully offline.
- **Frictionless use**: daily usage should not feel like “doing security.”
- **Multi-device**: adding a device should be possible without a required server.
- **Explicit trust**: identity continuity matters; trust resets are visible.
- **No operator access**: there must be no design path where Aspen (as a service) can access user keys or data.

---

## Threat model (key-management specific)

### In scope
- Prevent service operators / relays from decrypting user data
- Prevent unauthorized peers from reading space contents
- Detect identity changes after first trust (continuity)
- Ensure only authorized devices can author accepted operations

### Out of scope
- A fully compromised device (malware, root access)
- A malicious collaborator with legitimate access
- Retroactive revocation of already-replicated content

These constraints must be reflected in UX language.

---

## Key types

### 1) User identity key
A long-lived key pair representing a person.

- Used to establish a stable identity across spaces and devices
- Used to authorize device keys
- Changes rarely and triggers explicit security events (ADR-0005)

**Notes**
- Treat identity key changes as meaningful events.
- Recovery is hard by design; avoid silent resets.

---

### 2) Device keys
A key pair per device install.

- Used to sign authored operations
- Used to authenticate sync traffic
- Authorized by signatures from the user identity key (ADR-0005)

**Notes**
- Device keys rotate more often than identity keys.
- Losing a device should not require identity reset.

---

### 3) Space keys
Key material per space (including overlay spaces).

- Used to encrypt and decrypt **all authoritative data** in that space:
  - ops
  - revisions
  - membership events
  - blobs
- Membership implies access to space keys (ADR-0009)

**Notes**
- Space keys are the cryptographic enforcement mechanism for space-level permissions (ADR-0003).

---

### 4) Local storage key (optional, device-local)
A key used to encrypt sensitive local state at rest on the device, such as:
- cached decrypted material
- key material envelopes
- local derived state if desired

This is separate from E2EE and exists to reduce risk from disk theft or offline extraction.

---

## Key hierarchy and envelopes

Aspen should use an envelope model:

- Space keys are encrypted (“wrapped”) for each authorized device.
- A device stores only wrapped space keys plus the information required to unwrap them.
- Unwrapping requires access to device-local secrets and/or OS-secure storage.

At a conceptual level:

- **Identity key** authorizes devices
- **Device key** proves authorship
- **Space key** decrypts space content
- **Device-local secret** protects space key envelopes at rest

---

## Where keys live (at rest)

Key material must be stored so that:

- adding a device is possible
- daily use is low friction
- a stolen disk does not trivially expose everything

### Recommended storage strategy (by platform capability)

1) **Use OS secure storage when available** (preferred)
- macOS Keychain
- Windows DPAPI / Credential Locker
- iOS Keychain / Secure Enclave where available
- Android Keystore where available

2) **Fallback to passphrase-based protection**
- user supplies an Aspen passphrase
- a derived key encrypts (“wraps”) local key envelopes

3) **Hybrid**
- OS secure storage holds the key-encryption-key (KEK)
- passphrase is a secondary factor (optional)

**Important**
Aspen must not depend on any single mechanism universally. Devices vary.

---

## Unlocking keys (runtime)

Aspen must support multiple unlock models.

### Model A: Transparent unlock (best UX)
- OS-secure storage unlocks automatically (e.g., logged-in user session).
- No prompt for normal use.

**When this is acceptable**
- The platform is relying on OS-level account security and full-disk encryption.

### Model B: Biometric-gated unlock (nice-to-have)
- Biometrics gate access to key material via OS APIs.
- Works well for “open Aspen” interactions.

**Constraints**
- Not all devices support biometrics.
- Biometrics are local-only and cannot be used for cross-device sync.
- Background sync may not be possible if biometrics are required on every access.

Use biometrics as a convenience layer on top of local key storage, not as the root of trust.

### Model C: Passphrase unlock (most portable)
- User enters a passphrase to unlock Aspen.
- Enables consistent behavior across platforms.

**Tradeoff**
- More friction.
- Needs careful UX: cache unlock for session, support “lock now,” etc.

### Practical requirement
Aspen should support:
- “Unlock once per session”
- explicit “Lock Aspen”
- reasonable timeout behavior (configurable later)

---

## Adding a device (key transfer)

Adding a device requires transferring access without a central authority.

Conceptually:

1) New device generates a device key pair.
2) Existing trusted device verifies the new device (UX step).
3) Existing device:
   - signs the new device key with the user identity key
   - encrypts space key envelopes for the new device
4) New device stores wrapped keys locally.

Transfer channels may include:
- direct P2P on LAN
- QR code out-of-band pairing
- manual transfer of an encrypted package
- optional relay mailbox (encrypted payload only)

**Constraints**
- No server should ever see plaintext keys.
- Pairing should minimize opportunities for silent MITM.

---

## Sharing a space with another user

Sharing implies sharing the space key with another identity’s devices.

Conceptually:

1) Initiator selects a target user identity (TOFU if first contact).
2) Initiator creates an invitation that includes:
   - space metadata (minimal)
   - wrapped space key material for the recipient’s devices (or for a recipient “key agreement” flow)
3) Recipient accepts and stores the space key envelope.
4) Recipient can now decrypt space content.

**Identity continuity**
If the recipient identity changes later, sharing must require explicit re-verification (ADR-0005).

---

## Key rotation and revocation

### Device revocation
- Stop accepting operations from that device key.
- Future key envelopes are not issued to that device.

**Limit**
The device may retain old space keys and old data already replicated.

### Space key rotation
- Rotate space keys to prevent future access by revoked members.
- Requires re-wrapping new space keys to remaining members.

**Limit**
Does not claw back old data.

### Identity key rotation (rare, serious)
- Treat as a security event requiring explicit re-verification by collaborators.

---

## Local disk footprint and external sources (relationship to ADR-0007)

Key management intersects storage modes:

- **Reference mode**:
  - Aspen persists extracted summaries (encrypted in the space)
  - does not necessarily store full source bytes
- **Snapshot mode**:
  - Aspen stores encrypted snapshot blobs

Key management must support large blobs efficiently:
- stream encryption/decryption
- chunking and deduplication are implementation choices, but should not change the trust model

---

## Failure and recovery

### If a device is lost
- Revoke device key.
- Rotate space keys if necessary.
- Add a new device via pairing from another trusted device.

### If all devices are lost (identity key loss)
Default assumption:
- Identity continuity is broken.
- Collaborators must re-verify a new identity.
- Previously shared spaces cannot be recovered unless recovery material exists.

Recovery mechanisms are optional and must be explicit, e.g.:
- user-generated recovery kit (encrypted seed / recovery key)
- printed phrase stored offline
- split-key schemes (future)

Do not imply “account recovery” like a SaaS product.

---

## Non-goals

- Perfect protection against a compromised device
- Retroactive revocation of already-replicated content
- Requiring biometrics universally
- A required cloud escrow or “reset password” flow
- Hiding security events from users

---

## Open questions (intentionally unresolved)

- Should Aspen encrypt all local derived state by default, or rely on OS disk encryption?
- What pairing UX is acceptable for non-technical users?
- Should there be a “recovery kit” in MVP, or explicitly defer?
- How should background sync behave when the store is locked?
- What is the minimal metadata leaked during invites and membership changes?

These questions should be resolved before building any UX that implies guarantees.

---