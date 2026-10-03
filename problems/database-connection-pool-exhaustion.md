# Connection Pool Exhausts After Failed Requests

Difficulty: Hard. Core topic: resource lifecycle, error paths, transaction hygiene.

The database-engineering round. A reporting service returns pooled connections on the happy path only, and returns some of them with a transaction still open — so a run of failed requests quietly bricks the service.

## Scenario

A reporting service borrows connections from a fixed-size pool to build customer summaries and recent-order listings. The pool hands out a connection, the service uses it, and the connection is returned for reuse.

The service runs normally for a while and then stops responding, with every request timing out while waiting for a connection. The failure correlates with requests for customers that do not exist. Database monitoring also shows pooled sessions sitting in `idle in transaction`, holding snapshots open long after their request finished.

The pool itself is shared infrastructure and must not be changed, and enlarging it is not a fix — it only moves the cliff.

## Requirements

- Return the connection on every path, including exceptions and early returns.
- Ensure a returned connection has no open transaction and is idle.
- Never exceed the configured pool size.
- Keep repeated failures from degrading the service's ability to serve later requests.
- Do not modify the tests or `verify.sh`.

## Edge cases to handle

- A lookup that raises after the connection is borrowed but before any query
- A query that fails mid-transaction, leaving it open
- Nested borrows within one request
- An exception thrown while returning or rolling back
- Many consecutive failures, where a single leak per request is fatal

## What interviewers look for

Whether cleanup is structural — scoped acquisition that cannot be skipped — rather than an extra release call bolted onto each error branch. A full-marks answer also treats `idle in transaction` as its own defect, rolling back or committing before release, and can explain why a slightly bigger pool just delays the outage.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/database-connection-pool-exhaustion
