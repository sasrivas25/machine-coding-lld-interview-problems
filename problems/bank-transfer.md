# Concurrent Bank Transfer

Difficulty: Medium. Core topic: deadlock prevention, atomicity.

A staple of backend interviews because it forces you to reason about several hard things at once: atomicity, deadlocks, race conditions, and idempotency. This is not a data-structures question — it tests whether you can take a service that looks correct on a single thread and make it correct when many transfers run in parallel, including two transfers touching the same two accounts in opposite directions.

## Scenario

You maintain a service that moves money between accounts. A transfer debits a source, credits a destination, and lands in a ledger. The happy path works when requests arrive one at a time. Under concurrency it breaks: two opposing transfers (A→B and B→A) deadlock waiting on each other's locks, the balance check happens at the wrong moment so accounts get overdrawn, failed transfers leave partial changes behind, and retried requests apply the same transfer twice.

## Requirements

- The balance check and both mutations happen in one critical section: no overdrafts, no partial transfers.
- Deadlock-free: when a transfer needs both account locks, acquire them in a stable global order (e.g. lower account id first) — without collapsing everything onto one global lock.
- A failed transfer leaves both accounts and the ledger unchanged.
- Idempotency: a retried request returns the original result instead of moving money again.
- Validation up front: reject same-account transfers and non-positive amounts before touching the money path.
- Transfer history is consistent on both accounts, with exactly one ledger entry per completed transfer.

## Edge cases to handle

- A→B and B→A running simultaneously
- A transfer racing the source account's other outgoing transfers past the balance
- The same idempotency key arriving twice, including concurrently
- Transfers involving unknown or inactive accounts

## What interviewers look for

Lock ordering as the deadlock answer — a stable global order so no waiting cycle can form — and the discipline inside the critical section: re-check the idempotency key, confirm accounts, verify funds before debiting, move the money, write one ledger entry. Getting concurrency wrong here means lost or duplicated money, which is why fintech loops love it.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/concurrent-bank-transfer-coding-problem
