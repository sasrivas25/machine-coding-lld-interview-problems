# Webhook Event Idempotency and Ordering

Difficulty: Hard. Core topic: idempotency, event ordering.

The at-least-once webhook round. Deliveries retry and per-account events arrive out of order, so the processor must dedup by event ID and apply each account's sequence in order — or retries double-apply balance changes and an early future event corrupts account state.

## Scenario

You maintain a customer integration that receives account-lifecycle webhooks from an external billing provider. Delivery is at-least-once and per-account events may arrive out of order. The starter processor handles simple in-order batches, so it demos cleanly, but production shows the two classic failures: a provider retry re-applies a balance change because the event was not recognized as a duplicate, and a later sequence number applied before its missing predecessor leaves the account in an incorrect state.

You fix the Python processor so it is safe under retries and reordering. Duplicates must be acknowledged without side effects, stale events ignored, genuinely future events deferred until their predecessors arrive, and events that can be ordered within the received batch applied deterministically.

## Requirements

- Every event is idempotent by event ID — a duplicate is acknowledged with no side effect.
- Each account's events are applied strictly in sequence order.
- Stale events (already superseded) are ignored.
- Future events (predecessor not yet applied) are deferred, then applied once the gap fills.
- Events that become orderable within the received batch are applied deterministically.

## Edge cases to handle

- The same event ID delivered two or three times.
- A later sequence arriving before an earlier one for the same account.
- A batch that fills an earlier gap, unblocking deferred events.
- Interleaved events across multiple accounts.
- An event stale relative to already-applied state.

## What interviewers look for

Whether idempotency and ordering are treated as two separate guarantees layered correctly: a dedup set keyed by event ID for exactly-once side effects, and per-account sequence tracking that defers rather than misapplies out-of-order events. A full-marks answer keeps each account's applied position explicit, drains newly-orderable deferred events after every batch, and makes replaying the whole stream a no-op.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/webhook-event-idempotency-and-ordering-coding-problem