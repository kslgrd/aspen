# Device Lifecycle and Recovery

This document describes how devices participate in Aspen, how trust is established and maintained over time, and what recovery realistically means in a local-first, end-to-end encrypted system.

This is a **conceptual model**, not a protocol specification.

---

## Goals

Device lifecycle management must:

- support multiple devices per user
- preserve identity continuity (ADR-0005)
- avoid central authorities
- make trust transitions visible
- be honest about what can and cannot be recovered
- avoid pretending to offer SaaS-style “account recovery”

---

## Core concepts

### User identity vs device
Aspen distinguishes between:
- **who you are** (user identity)
- **where you act from** (devices)

A user may have:
- zero or more devices
- devices added or removed over time
- devices with different trust states

Identity is long-lived.  
Devices are replaceable.

---

## Device states

Each device known to a user has a state:

- **Active**  
  Authorized to author operations and participate in sync.

- **Revoked**  
  No longer trusted to author new operations.

- **Unknown**  
  A device claiming an identity that has not been authorized.

State is evaluated locally per peer.

---

## Adding a new device

Adding a device is an explicit trust operation.

### High-level flow

1) New device generates a device key pair.
2) New device presents its device key to an existing trusted device.
3) Existing device verifies intent (user confirmation).
4) Existing device:
   - signs the new device key using the user identity key
   - provides encrypted space key envelopes for authorized spaces
5) New device becomes active.

This process must be:
- explicit
- user-visible
- resistant to silent MITM

---

## First device bootstrap

The first device:
- generates the user identity key
- implicitly authorizes itself
- becomes the root of trust

This is the only moment when identity is created without verification.

---

## Trust On First Use (TOFU)

When encountering another user identity for the first time:

- the identity is accepted provisionally (TOFU)
- the identity key is pinned locally
- future changes to that identity key are treated as security events

TOFU minimizes friction while preserving continuity.

---

## Revoking a device

Revocation is a **forward-looking** action.

### Effects

- Operations signed by the revoked device are rejected going forward.
- The revoked device does **not** receive new space key material.
- Other devices may rotate space keys to prevent future access.

### Limits

- Revocation does not erase data already replicated.
- Revocation does not prevent offline misuse of cached data.
- Revocation is not retroactive.

These limits must be reflected in UX.

---

## Space key rotation after revocation

After revoking a device or removing a member:

- Space owners may rotate the space key.
- New space keys are distributed to remaining authorized devices.
- Revoked devices cannot decrypt new content.

This is optional but recommended for sensitive spaces.

---

## Device loss

### Losing a single device

If at least one trusted device remains:
- revoke the lost device
- rotate space keys if needed
- add a replacement device via pairing

This is the **happy path**.

---

### Losing all devices (identity loss)

If all devices holding the identity key are lost:

- identity continuity is broken
- collaborators cannot distinguish a legitimate recovery from impersonation
- previously shared spaces cannot be rejoined automatically

Aspen must treat this as an unrecoverable event **by default**.

This is not a bug; it is a consequence of decentralization and end-to-end encryption.

---

## Optional recovery mechanisms

Aspen may support **explicit, user-managed recovery options**, such as:

- encrypted recovery key / seed phrase
- offline recovery kit generated at setup
- split-key or social recovery schemes (future)

Constraints:
- recovery must be opt-in
- recovery must be explained clearly
- recovery material must never be silently escrowed

If recovery is enabled, users must understand the tradeoff.

---

## Device trust UI principles

Device-related UX must follow these rules:

- show devices by meaningful labels (name, last seen)
- surface changes clearly (new device, revoked device)
- avoid constant prompts
- warn only when trust boundaries change
- never auto-accept identity or device changes

Security should be visible, not noisy.

---

## Relationship to sync

- Devices participate in sync only if active.
- Sync does not imply trust; trust gates sync.
- Offline devices may reappear later and must be re-evaluated.

---

## Non-goals

- Perfect recovery in all cases
- Retroactive revocation of data
- Invisible trust resets
- “Reset my account” semantics

Aspen chooses honesty over false reassurance.

---

## Open questions

- What pairing UX is acceptable for non-technical users?
- Should device approval be per space or global?
- How aggressively should space keys be rotated by default?
- How much device metadata is appropriate to persist and share?

These questions should be revisited before finalizing onboarding UX.

---