# Inventory Overselling Under Concurrency

Difficulty: Hard. Core topic: row locking, idempotent reservations.

The flash-sale database round. Sequential tests pass, but overlapping reservations confirm more stock than exists and retries create multiple rows for one idempotency key.

## Scenario

A warehouse service stores total, available, and reserved quantities in PostgreSQL and records each reservation attempt. Multiple application processes reserve the same hot SKU concurrently, while the upstream order service retries slow requests with the same idempotency key.

The starter implementation reads availability, decides in application code, and writes later. Concurrent transactions observe the same old value and overwrite each other. The idempotency check is also a read-before-insert race, allowing duplicate logical reservations.

## Requirements

- Never confirm more quantity than the product's physical stock.
- Keep available and reserved counters consistent with confirmed reservations.
- Resolve one idempotency key to exactly one reservation row.
- Return the same outcome to all concurrent retries.
- Reject unsatisfied requests atomically, across multiple processes.

## Edge cases to handle

- Many callers competing for the final unit
- Concurrent retries of the same request
- A retry racing with a first attempt that has not committed
- Rejected reservations leaving inventory unchanged
- Rollback after a reservation row is tentatively created

## What interviewers look for

Whether correctness is enforced at the database boundary with transactions, row-level locking or an atomic conditional update, and a unique constraint for idempotency. An in-process mutex cannot coordinate separate service instances and is not a valid fix.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/inventory-overselling-under-concurrency
