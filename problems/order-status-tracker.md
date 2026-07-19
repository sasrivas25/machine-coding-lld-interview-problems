# Food Delivery Order Status Tracker

Difficulty: Easy. Core topic: state machines.

The Swiggy / DoorDash-style order lifecycle — one of the most common warm-up rounds, and the cleanest possible test of one skill: implementing a state machine properly. It is rated easy, which is exactly why it is expected airtight, not just working.

## Scenario

You maintain the order-tracking service of a food delivery backend. Status updates, cancellations, and history queries all exist — and the lifecycle is full of holes. Orders skip states and move backwards, cancellation succeeds from states where it should be refused (and fails where it should work), the status history misses transitions or records rejected ones, and listings come back in unstable order.

## Requirements

- Orders move placed → accepted → preparing → out for delivery → delivered. No skips, no backward moves.
- Cancellation is legal only from specific states and refused cleanly elsewhere.
- Terminal states are terminal: nothing leaves delivered or cancelled.
- History is an append-only record of every successful transition — no more, no fewer.
- Rejected updates and unknown orders produce distinct, clear errors.
- Listings and histories are deterministically ordered.

## Edge cases to handle

- Backward moves (delivered → preparing)
- Skipped states
- Cancelling after delivery; double-cancelling
- Status updates for unknown orders
- History after a rejected transition (must be unchanged)

## What interviewers look for

Whether you write the transition table before writing code, and funnel every status change through one enforcement point that consults it, rejects violations, and appends to history in the same step. Free-string statuses and scattered if-checks at call sites are what this round is designed to expose.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/food-delivery-order-status-tracker-coding-problem
