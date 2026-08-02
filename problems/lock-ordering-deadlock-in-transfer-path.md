# Lock-Ordering Deadlock in a Transfer Path

Difficulty: Hard. Core topic: thread dumps, global lock ordering.

The classic two-resource deadlock round, made deterministic. Opposite-direction transfers each hold one account lock and wait forever for the other.

## Scenario

A ledger transfer locks the source account and then the destination so no caller can observe a half-applied debit/credit. When `A → B` and `B → A` overlap, each thread acquires its source first, forming a circular wait. Production freezes during bilateral netting; the dump identifies the held and requested locks.

The repair must keep per-account concurrency. A single global ledger lock prevents the cycle but discards the design's parallelism.

## Requirements

- Make opposite-direction transfers complete without deadlock.
- Preserve atomic debit and credit behavior.
- Keep unrelated account pairs concurrent.
- Reject invalid transfers exactly as before.
- Release both locks safely on every path.

## Edge cases to handle

- Transfers in both directions over the same pair
- Self-transfer behavior
- Three transfers whose account pairs overlap
- Insufficient funds after locks are acquired
- Exceptions while both resources are held

## What interviewers look for

Whether you derive a stable global order from account identity, acquire both locks in that order regardless of transfer direction, and release in a structured scope. Retries, sleeps, and larger timeouts make the schedule less likely but leave the cycle possible.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/lock-ordering-deadlock-in-transfer-path
