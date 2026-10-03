# Stale Inventory After a Delivery Reordering

Difficulty: Hard. Core topic: version-aware projections, monotonic state.

The distributed-systems round. The transport promises nothing about order, the projector applies whatever arrives, and after a reconfiguration several SKUs disagree with the source of truth permanently.

## Scenario

A projection service maintains the current quantity of every SKU from a stream of change events. Each event names its entity, carries a per-entity version that increases by one with every upstream change, and supplies the new quantity. Readers query the projection continuously while the stream is being consumed, so it must be usable at every moment, not only once the stream is exhausted.

The transport delivers each event at least once and promises nothing about order.

After a network reconfiguration reordered part of the stream, the projection began to disagree with the source of truth for several SKUs, and the disagreement did not resolve on its own. The service reported no errors: it received every event, applied every event, and logged each application as a success.

The captured evidence contains the delivery trace, the ledger of projection updates with the version each wrote, the projector's log, and samples of what readers observed mid-run.

## Requirements

- Make the projection correct under arbitrary delivery order.
- Never let an entity's version move backwards.
- End with each entity at its highest-versioned event.
- Never let a mid-run reader observe a version older than one already delivered — so buffering and sorting at the end is ruled out.
- Do not modify the tests.

## Edge cases to handle

- A duplicate delivery of an event already applied
- An old event arriving after a newer one
- An entity's first event arriving out of order relative to another entity's
- Readers querying while an apply is in progress
- Equal versions delivered twice with the same payload

## What interviewers look for

Whether each apply becomes a conditional, version-gated write rather than an unconditional overwrite. A full-marks answer keeps the projection monotonic per entity, stays readable at every instant, and can explain why at-least-once plus unordered delivery makes the version comparison mandatory rather than defensive.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/out-of-order-event-application
