# Lock Leaked on the Exception Path

Difficulty: Hard. Core topic: critical sections, exception safety, thread dumps.

The runtime-diagnostics round. A settlement service takes a lock explicitly and releases it only on success, so the first rejected record wedges every request that follows — no error, no timeout, nothing.

## Scenario

A settlement service applies incoming requests one at a time. A single lock serialises them because the downstream ledger cannot tolerate two concurrent applications. Most requests settle normally. Some are rejected: a malformed or unbalanced request raises a validation error back to its caller, which is expected and handled.

Support reports that the service runs normally until one bad request arrives. The bad request is rejected correctly, with a sensible error naming the failing record. From that moment every subsequent request hangs — no error, no timeout, no log line. They simply never return. A restart clears it until the next bad request.

The service is not slow or busy. It is doing nothing at all.

A capture from a stuck process ships with the problem: a full thread dump plus a record of which requests settled, which failed, and which never returned. Read it carefully — the obvious hypothesis does not survive it. The Java capture in particular shows the runtime's own deadlock detector reporting nothing.

Four tests pass today. Four fail: a valid record after a rejected one never settles, most records never settle, worker threads are still blocked past a bounded deadline, and the ledger total is short by the value of every record that never completed.

## Requirements

- Release the lock on every exit path, including the rejection path.
- Keep rejecting the poisoned request, still naming the failing record, still not counting it as settled — swallowing the error breaks passing tests.
- Keep requests serialised.
- Make all eight tests pass.
- Keep the fix in `src/`; do not change `tests/`, `verify.sh`, or the workload constants.

## Edge cases to handle

- Validation failing after the lock is taken but before any mutation
- An exception raised inside the release path itself
- Re-entrant application on the same thread
- Workers blocked past the bounded deadline in the test harness
- The ledger total reconciling exactly after the fix

## What interviewers look for

Whether the fix is structural — scoped acquisition that cannot be bypassed — rather than an extra release in one catch block. A full-marks answer notes why no deadlock is reported (there is no cycle, just an abandoned lock), keeps rejection semantics intact, and reconciles the ledger total as proof.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/lock-leaked-on-exception-path
