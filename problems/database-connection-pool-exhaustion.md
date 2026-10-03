# Connection Pool Exhausts After Failed Requests

Difficulty: Hard. Core topic: connection lifecycle, transaction cleanup.

The resource-lifecycle database round. A three-connection pool is adequate until several expected lookup failures permanently consume every connection and leave PostgreSQL sessions idle in transaction.

## Scenario

An admin console borrows pooled connections to build customer reports. Happy-path calls release the connection, but an unknown-customer exception jumps over cleanup. Other paths return a connection after `BEGIN` without committing or rolling back. After three failures, every request times out waiting for the exhausted pool; before that, `pg_stat_activity` already shows sessions sitting `idle in transaction`.

The pool implementation is correct. The defect is in the report service's ownership of borrowed connections and transaction boundaries.

## Requirements

- Return every acquired connection on success and failure.
- Commit successful transactions and roll back failed ones before release.
- Never return a connection in an open or aborted transaction.
- Preserve report results and existing error behavior.
- Fix lifecycle handling without enlarging the pool or timeout.

## Edge cases to handle

- An exception before the first query completes
- A missing customer after a transaction has begun
- A query failure that leaves PostgreSQL's transaction aborted
- Repeated failures followed by a valid request
- Cleanup itself needing deterministic ordering

## What interviewers look for

Whether connection ownership is expressed with scope-bound cleanup—`finally`, context management, try-with-resources, or RAII—and whether transaction cleanup happens before pool release. Increasing capacity only delays the outage and misses the pinned-snapshot damage.

---

Practice this in a real repo with a failing test suite → https://gronex.org/database-connection-pool-exhaustion-coding-problem
