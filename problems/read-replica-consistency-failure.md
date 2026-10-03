# Read Replica Serves Stale Data After a Write

Difficulty: Hard. Core topic: read-after-write consistency, replica routing.

The database-engineering round. Writes go to the primary, reads go to an asynchronous replica, and the router does not know the difference — so a customer who just placed an order is told it does not exist.

## Scenario

The order service writes to a primary database and serves reads from a replica. A replication link applies committed changes asynchronously and the replica records how far it has caught up. A router decides, per read, whether to use the primary or the replica.

Customers place an order, land on the confirmation page, and are told the order does not exist. A refresh a few seconds later shows it correctly. Order listings behave the same way: a newly placed order is missing from the customer's list until replication catches up.

Routing everything to the primary would fix correctness and destroy the reason the replica exists. The replication link is shared infrastructure and is not yours to change.

## Requirements

- A read issued after a write by the same session must observe that write.
- Listings must include the caller's own recent writes.
- Reads not following a write must still be served by the replica.
- Once the replica has caught up, reads return to the replica.
- Do not modify the tests or `verify.sh`; sending every read to the primary is not an acceptable fix.

## Edge cases to handle

- A session that writes once and then reads many times
- A different session reading data it did not write
- Replica lag that is already zero at read time
- The replica's catch-up position going backwards or stalling
- Concurrent sessions with different write positions

## What interviewers look for

Whether you reach for the replica's recorded position rather than a sleep or a blanket primary read. A full-marks answer tracks the session's own write position, compares it against replica progress per read, and can state exactly which consistency guarantee it provides — and which it deliberately does not.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/read-replica-consistency-failure
