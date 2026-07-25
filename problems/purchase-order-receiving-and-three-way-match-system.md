# Purchase Order Receiving and Three-Way Match System

Difficulty: Hard. Core topic: cumulative receipts, three-way match.

The procurement-backend round. Partial receipts that accumulate against order lines, an over-receipt tolerance that has to be a percentage not a hard cap, and a three-way match that only pays what was ordered and actually received. The workflow is stubbed; the aggregation is wrong.

## Scenario

You are working on a partially implemented purchase order receiving and invoice matching backend. Purchase orders carry multiple order lines; goods receipts arrive in parts and accumulate against each line; an over-receipt tolerance permits a little slack; PO status transitions follow cumulative receipts; and invoices are matched three ways against ordered price and received quantity. Several visible tests fail because some repository and service logic is incomplete or incorrect — receipts don't accumulate correctly, the tolerance is mis-applied, statuses transition at the wrong points, or invoices match against the wrong quantities.

Read the models, repositories, services, and visible tests, infer the intended behaviour, and fix the implementation. Do not rewrite from scratch, do not change public method contracts, and do not modify the tests.

## Requirements

- Track cumulative received quantity per order line across multiple receipts.
- Enforce over-receipt tolerance as a percentage of the ordered quantity.
- Drive purchase order status transitions from cumulative receipts.
- Match invoices three ways: ordered price and received quantity must agree.
- Reject receipts and invoices that violate line-level constraints.
- Existing public contracts and tests remain untouched.

## Edge cases to handle

- Several partial receipts that together reach or exceed the ordered quantity
- A receipt landing right at, just under, and just over the tolerance percentage
- A line fully received while others on the same PO are still open
- An invoice quantity that exceeds what was actually received
- Status transitioning to fully received only when every line qualifies

## What interviewers look for

Whether cumulative state is reconstructed from the receipt history rather than tracked ad hoc, and whether the tolerance is computed as a percentage of the order line, not a fixed number. Full marks show status transitions derived from aggregate receipts and a three-way match that refuses to pay for quantity that was never received — parent-child validation and aggregate reconciliation handled as invariants.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/purchase-order-receiving-and-three-way-match-system
