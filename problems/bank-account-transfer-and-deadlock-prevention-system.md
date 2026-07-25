# Bank Account Transfer and Deadlock Prevention System

Difficulty: Hard. Core topic: concurrency, deadlock prevention.

The concurrency round with money on the line. Two accounts, two locks, two threads transferring in opposite directions — and if you grab the locks in the order the arguments happen to arrive, the whole service seizes. The trick is fine-grained locking that still can't deadlock.

## Scenario

You are handed a backend for bank transfers that mostly works and occasionally freezes or corrupts a balance. Transfers lock the two accounts involved and mutate them, but the locking order follows the call, not a stable rule, so opposite-direction transfers can each hold one lock and wait forever for the other. Validation and mutation drift apart, letting a balance slip negative under concurrency, and a retried transfer can move money twice.

The fix must keep per-account locking so unrelated transfers still run in parallel — a single global lock is off the table.

## Requirements

- Balances must never go negative.
- Both accounts are locked in a stable global order by account id, then validated and mutated in one critical section.
- Same-account transfers and non-positive amounts are rejected.
- An idempotency retry returns the original successful transfer without moving money again.
- A failed transfer leaves both accounts exactly as they were.
- Unrelated transfers proceed concurrently; no one global lock.

## Edge cases to handle

- `transfer(A, B)` and `transfer(B, A)` racing at the same instant
- A transfer whose amount exceeds the source balance
- Self-transfer and zero/negative amounts
- The same transfer request replayed after a success
- A validation failure mid-transfer that must roll nothing back because nothing changed yet

## What interviewers look for

Whether the candidate reaches for a total order over lock acquisition rather than luck or a coarse lock, and keeps the check-and-move inside one critical section so the balance invariant holds under contention. Full marks show idempotency and atomicity treated as first-class: a retry is not a second transfer, and a rejected transfer is a no-op, not a partial one.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/bank-account-transfer-and-deadlock-prevention-system
