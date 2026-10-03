# Outbox Publishes Phantom and Duplicate Events

Difficulty: Hard. Core topic: transactional outbox, exactly-once delivery.

The database-engineering round. The order write and the outbox write live in different transactions, and several publishers claim the same rows — so downstream sees events for orders that were never committed, and sees the real ones more than once.

## Scenario

Placing an order must record the order and enqueue an `order.placed` event for downstream consumers. Events are written to an `outbox` table and delivered by a publisher that moves unpublished rows into `published_events` and marks them published. Several publisher instances run concurrently.

Consumers see two classes of defect. There are events for orders that do not exist, produced when an order fails validation after its event has already been enqueued. And the same event is delivered several times whenever more than one publisher is running.

Serialising down to a single publisher is not an acceptable fix; publishing has to stay concurrent.

## Requirements

- An event must exist if and only if its order was committed.
- A rejected order must leave no trace in the outbox.
- Deliver each outbox row exactly once, even with concurrent publishers.
- Keep batches bounded and publishing idempotent on retry.
- Do not modify the tests or `verify.sh`.

## Edge cases to handle

- Validation failing after the event row is inserted
- Two publishers polling the same batch simultaneously
- A publisher crashing between moving a row and marking it published
- Re-running a batch that was already fully published
- Ordering requirements within a single order's events

## What interviewers look for

Whether the order and its event share one transaction, and whether the claim step is a real claim rather than a read followed by a hopeful update. A full-marks answer accepts either a blocking claim or a skip-locked claim, keeps the batch bounded, and can explain why "exactly once" here means exactly-once effects rather than exactly-once delivery attempts.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/transactional-outbox-implementation
