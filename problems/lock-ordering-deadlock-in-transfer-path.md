# Lock Ordering Deadlock in the Transfer Path

Difficulty: Hard. Core topic: deadlock, lock ordering, concurrency.

The runtime-diagnostics round. A ledger locks source then destination, so two transfers running in opposite directions between the same pair of accounts wedge the service permanently — most visibly during end-of-day netting.

## Scenario

A settlement ledger holds balances in cents, and a transfer moves value between accounts. Because a transfer must debit and credit as a single visible step, it holds both account locks for its duration: take the source lock, take the destination lock, apply the movement, release both.

Sequential traffic has been fine for months. Settlement operations now report that the service intermittently stops completing transfers altogether. It does not crash, logs no errors, and never recovers on its own — the process has to be restarted. The stall is far more likely during end-of-day netting, when the same pairs of accounts settle against each other in both directions at once.

A capture from a wedged process ships with the problem: a full thread dump and a record of which account locks were held and which workers were still waiting at that moment. Every language's capture points at the same structure in its own dialect.

Four tests pass today. Three fail, and they describe exactly what operations reports: opposing transfers never settle, worker threads stay blocked past a bounded deadline, and the behaviour does not change when the workload is scaled.

## Requirements

- Make all seven tests pass without weakening the ledger's guarantees.
- Keep balances correct and keep invalid transfers rejected.
- Keep transfers touching disjoint accounts running concurrently — a single global ledger lock fails this.
- Keep the fix in `src/`; do not change `tests/`, `verify.sh`, or the workload constants.

## Edge cases to handle

- Self-transfer, where source and destination are the same account
- A deterministic total order over accounts for lock acquisition
- Opposing transfers between the same pair under sustained load
- Releasing both locks on the rejection path
- Scaling the workload without changing behaviour

## What interviewers look for

Whether you read the dump, identify the cycle, and impose a consistent global ordering instead of reaching for a coarse lock or a timeout-and-retry. A full-marks answer orders acquisition by a stable account key, handles the same-account case explicitly, and preserves concurrency between unrelated account pairs.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/lock-ordering-deadlock-in-transfer-path
