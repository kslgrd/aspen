# Sync Topology

This document describes Aspen’s synchronization model at a conceptual level:  
how devices find each other, how data moves, and what guarantees are (and are not) provided.

This is **not** a wire protocol specification. It defines constraints, expectations, and invariants that implementations must respect.

---

## Goals

Aspen’s sync model must:

- support local-first operation
- work without any required backend service
- scale from one device to many devices and peers
- tolerate offline and intermittent connectivity
- preserve end-to-end encryption
- avoid central authority or coordination
- align with CRDT-based convergence

---

## Core principle

**Synchronization is space-scoped, peer-to-peer, and opportunistic.**

There is no global sync state and no single “source of truth” node.

---

## Unit of synchronization

The **space** is the unit of synchronization (ADR-0009).

- Devices synchronize data **per space**
- A device participates in sync for a space if and only if:
  - it is a member of the space
  - it possesses valid space keys
- There is no partial synchronization within a space

Derived state (indexes, embeddings, layouts) is never synchronized (ADR-0008).

---

## Peer model

Each device is a peer.

Peers may be:
- personal devices owned by one user
- devices belonging to collaborators
- intermittently connected
- behind NATs or firewalls

Peers are treated symmetrically.  
There are no special “primary” or “leader” nodes.

---

## Connectivity assumptions

Aspen makes **minimal assumptions** about connectivity:

- peers may be offline for long periods
- peers may connect directly (LAN, Bluetooth, local network)
- peers may connect indirectly via relays
- peers may never all be online at the same time

The system must converge eventually when connectivity exists.

---

## Transport neutrality

Aspen’s sync logic is **transport-agnostic**.

Possible transports include:
- direct TCP/UDP connections
- local network discovery
- WebRTC data channels
- encrypted relay servers
- store-and-forward mailboxes

The transport:
- does not need to be trusted
- must not have access to plaintext
- must not influence authority or ordering

Sync correctness must not depend on transport behavior.

---

## Relay and mailbox servers (optional)

Aspen may use optional infrastructure to improve connectivity.

Such services may act as:
- rendezvous points
- encrypted message relays
- temporary mailboxes for offline peers

Constraints:
- payloads are end-to-end encrypted
- relays do not authenticate authorship
- relays do not enforce permissions
- relays do not maintain authoritative state

Using a relay is a performance optimization, not a requirement.

---

## Sync strategy (conceptual)

Sync proceeds via **anti-entropy** per space.

At a high level:
1) Peers exchange summaries of what they know for a space.
2) Missing operations are requested.
3) Operations are verified, decrypted, and applied locally.
4) CRDT rules ensure convergence.

No peer is assumed to have a complete or up-to-date view at all times.

---

## Ordering and causality

- Operations are appended locally.
- Global total ordering is not required.
- Causal ordering is respected where relevant.
- Conflicting concurrent operations converge deterministically.

Time synchronization between peers is not required.

---

## Identity and trust during sync

Before accepting data from a peer:

- the peer’s device key must be authorized (ADR-0005)
- the peer must be a member of the space
- operations must be signed correctly

Sync is gated by trust, not connectivity.

---

## Handling many connections

Aspen anticipates scenarios with:
- many peers in a space
- bursty connectivity
- partial overlap of online sets

Design expectations:
- peers sync opportunistically
- no requirement for full mesh connectivity
- eventual convergence without coordination storms
- tolerance for slow or lagging peers

Efficiency is important, but correctness and privacy come first.

---

## Failure modes

Aspen must tolerate:

- peers going offline mid-sync
- duplicate delivery of operations
- delayed delivery
- reordering
- dropped messages
- relay failure

Failure must degrade availability, not correctness.

---

## What sync does *not* do

Sync does **not**:

- guarantee immediate consistency
- guarantee delivery deadlines
- resolve social conflicts
- retroactively revoke data
- hide trust changes

Any UI implying these guarantees is incorrect.

---

## Relationship to user experience

Users should experience sync as:

- automatic when possible
- invisible most of the time
- predictable when trust changes occur
- resilient to network conditions

Users should not need to understand topology to use Aspen safely.

---

## Non-goals

- Centralized coordination
- Global clocks or ordering
- “Always online” assumptions
- Server-enforced permissions
- Strong delivery guarantees

Aspen favors resilience and user control over strict coordination.

---

## Open questions

- How much peer metadata is shared during discovery?
- What heuristics govern sync frequency and batching?
- How should relay infrastructure be described to users?
- Should users be able to opt out of relays entirely?

These questions should be resolved alongside protocol design.

---