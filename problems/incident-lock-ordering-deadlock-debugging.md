# Incident Debugging: Ledger Transfers Freeze in Production

Difficulty: Hard. Core topic: lock ordering, deadlock.

The 3 AM pager round. Transfers stopped completing, the queue backed up, readiness timed out, and the only artefact you have is a thread dump the watchdog grabbed before the operator hit restart. Read it, name the deadlock, fix the source — and keep unrelated transfers concurrent.

## Scenario

At 03:12 UTC the ledger service stopped completing transfers. Requests kept arriving and queueing — depth went from under five to forty-eight in thirty seconds — the readiness probe timed out, and the on-call engineer restarted the pod. Hours later it froze again. Before the first restart the watchdog captured a full thread dump, bundled as `logs/incident.log`; it names which threads are stuck, which lock each holds, and which lock each awaits.

You are given the frozen source: a `LedgerService` with per-account locks via an injectable `LockManager`, a two-account `transfer(from, to, amount)`, and a single-account `deposit`. The bundled tests deterministically reproduce the freeze — two threads, one running `transfer(A, B)` and one `transfer(B, A)`, each take their first lock, meet at a barrier, then reach for the second. On the starter code that locks up every time and fails with a DEADLOCK message after a bounded timeout. Sequential transfers, same-direction concurrency, and single-account deposits already pass.

## Requirements

- Read `logs/incident.log` and locate the root cause in the source.
- Make the forced opposite-order scenario complete instead of freezing.
- Keep balances correct under concurrent transfers.
- The mixed-order stress workload must finish.
- Preserve per-account locking so unrelated transfers still run concurrently.
- Do not edit the tests or the log; `verify.sh` must print PASS.

## Edge cases to handle

- Two transfers touching the same pair of accounts in opposite directions
- A transfer whose two accounts, when ordered, collide with another in-flight pair
- Single-account deposits that must not regress
- Correctness of final balances after the stress workload, not just liveness
- Bounded timeouts so the suite reports DEADLOCK rather than hanging

## What interviewers look for

Whether the candidate reads the dump before touching code, identifies the acquire-order cycle as the root cause, and imposes a consistent global lock order rather than papering over it with a timeout, a global lock, or a retry loop. Full marks keep fine-grained concurrency intact and verify balances, proving the fix addresses the deadlock's cause and not just its symptom.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/incident-lock-ordering-deadlock-debugging
