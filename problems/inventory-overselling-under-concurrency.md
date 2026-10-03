# Inventory Overselling Under Concurrency

Difficulty: Hard. Core topic: atomic reservation, idempotency, hold expiry.

The database-engineering round. A reservation service reads availability and writes the hold in two separate steps, so a flash sale oversells stock, retried requests double-count, and expired holds never give their capacity back.

## Scenario

A multi-warehouse storefront reserves stock before checkout, then confirms, cancels, or expires each hold. During a flash sale, customers were charged for items that were not in stock, availability counters drifted negative, and SKUs marked sold out still had physical stock on the shelf.

The starter service has four distinct defects. It checks availability and inserts the reservation non-atomically, so two concurrent checkouts can both pass the check. It ignores the client idempotency key, so a retried request creates a second reservation. It never returns expired holds to available stock, so capacity leaks away over the sale. And it lets the same reservation be confirmed twice.

The test suite drives genuine concurrency — barriers to release threads simultaneously and bounded joins — so a fix that only works single-threaded will fail.

## Requirements

- Never let reserved quantity exceed on-hand quantity, under any interleaving.
- Create at most one reservation per idempotency key; a retry returns the original.
- Make confirm idempotent, and reject confirmation of a hold whose reservation has already expired.
- Release capacity consistently on both cancel and expiry.
- Keep availability correct per warehouse, not just in aggregate.
- Do not edit the tests or `verify.sh`.

## Edge cases to handle

- Two concurrent reservations for the last remaining unit
- A retry arriving while the first attempt is still in flight
- Expiry racing a confirm for the same hold
- Cancel after expiry has already released the capacity
- Confirm called twice with the same reservation id

## What interviewers look for

Whether check-then-write becomes a single atomic operation, and whether the idempotency key is enforced by a unique constraint rather than a prior read. A full-marks answer treats expiry as a state transition that must release stock exactly once, and keeps every invariant true while threads collide on the same SKU.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/inventory-overselling-under-concurrency
