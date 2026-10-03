# Fulfilment Could Not See the Order That Triggered It

Difficulty: Hard. Core topic: causal consistency, replica read routing.

The distributed-systems round. The orders service writes to the primary and immediately calls fulfilment, which reads from a lagging replica — so fulfilment acts on a world that predates the event it is reacting to.

## Scenario

Two services share an account store. The orders service writes an account update to the primary and then immediately asks the fulfilment service to act on that account. Fulfilment reads the account from one of two read replicas, which the primary replicates to asynchronously. A shared read-path helper builds the cross-service request and chooses which source serves the read.

One replica fell behind. Fulfilment began acting on accounts it could not see: reading a version older than the write that caused the request, and in some cases finding no account at all for orders that had definitely been written before the request was sent.

Nothing failed and no error was raised. Half of all reads were served by the lagging replica, and every one of them was stale.

The evidence contains each source's replication lag alongside the number of reads it served, a per-request record of the version required versus the version actually seen, the raw trace of writes and reads, and each service's log.

## Requirements

- Never let a request read state older than the write that caused it.
- Keep replicas carrying read load — on a healthy cluster every replica is current.
- Do not route every read to the primary; that is not a solution.
- Keep the cross-service request contract working.
- Do not modify the tests.

## Edge cases to handle

- One replica current and one lagging at the moment of the read
- A request with no causal predecessor
- Replica lag that clears between the write and the read
- An account that does not exist yet on the chosen replica
- Both replicas lagging past the required version

## What interviewers look for

Whether the required version travels with the request and gates source selection. A full-marks answer carries the causal token across the service boundary, picks a replica that has reached it (falling back only when none has), and keeps read load distributed on a healthy cluster.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/causal-consistency-lost-across-services
