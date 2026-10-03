# Read Replica Serves Stale Data After a Write

Difficulty: Hard. Core topic: read-after-write consistency, routing.

The replica-routing round. Moving reads off the primary improves throughput but breaks the confirmation flow because a user's next request can arrive before replication catches up.

## Scenario

An order router writes to the primary and returns the write's monotonic sequence number. Reads normally use a replica whose applied sequence is exposed in `replica_state`. Replication lag is deterministic in the exercise: callers decide exactly how far the replica has advanced.

Routing every read to the replica violates session read-after-write consistency. Routing every read to the primary restores correctness but defeats the architecture and fails the operational requirement.

## Requirements

- Ensure a session observes its own committed writes.
- Continue serving unrelated reads from the replica.
- Fall back to the primary only while the replica is behind the session watermark.
- Return to the replica once it has caught up.
- Preserve the public router API and observable source reporting.

## Edge cases to handle

- Several writes in one session before replication advances
- A read for data older than the session's latest write
- Replica progress exactly equal to the required sequence
- Sessions with no prior writes
- List and point reads sharing the same consistency rule

## What interviewers look for

Whether you track a per-session high-water mark and compare it with replica progress before routing. Strong answers distinguish global strong consistency from targeted read-your-writes guarantees and retain replica offload once the dependency is satisfied.

---

Practice this in a real repo with a failing test suite → https://gronex.org/read-replica-consistency-failure-coding-problem
