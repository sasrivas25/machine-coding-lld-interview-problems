# Warranty RMA Return Authorization System

Difficulty: Medium. Core topic: state machine, transactional consistency.

The returns-desk round. A return can only open while the purchase is still under warranty, it advances one stage at a time, and when it resolves as a refund the money and the restock have to agree. Skip a stage or let a refund and inventory drift apart and the ledger stops matching the warehouse.

## Scenario

You inherit a partially implemented warranty RMA backend. The models, repositories, and services exist, and the visible tests describe opening returns only under warranty, advancing each RMA one stage at a time, resolving an approved return as refund, replacement, or rejection, keeping refunds and restock consistent, and listing returns scoped to their owning customer.

The behavior is off in the places that matter for consistency. Returns open past the warranty window, RMAs skip stages in their lifecycle, a refund and its inventory restock can land out of sync, and listings leak returns across customers. Several visible tests fail until the eligibility checks, state transitions, and transactional resolution are correct.

## Requirements

- Open a return authorization only while the purchase is still under warranty.
- Advance each RMA through its lifecycle one stage at a time — no skipping.
- Resolve an approved return as a refund, a replacement, or a rejection.
- Keep the refund and the inventory restock consistent with each other.
- List return authorizations only for the customer who owns them.
- Preserve the existing public method contracts; do not modify the tests.

## Edge cases to handle

- A return requested exactly at the warranty window boundary
- An RMA advanced two stages in a single call
- A refund that succeeds while restock does not, or the reverse
- A rejection that must leave inventory and refunds untouched
- A listing query that must not return another customer's RMAs

## What interviewers look for

Whether the lifecycle is a genuine one-step-at-a-time state machine, whether the warranty check gates opening rather than resolution, and whether refund-plus-restock is treated as one atomic outcome so the two never diverge. The strongest answers scope every read to its owning customer and keep rejections free of side effects.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/warranty-rma-return-authorization-system