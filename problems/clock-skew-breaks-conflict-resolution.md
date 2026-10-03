# One Node's Writes Started Winning Every Conflict

Difficulty: Hard. Core topic: logical clocks, conflict resolution, replication.

The distributed-systems round. Three leaderless nodes resolve conflicts by wall-clock timestamp. One node's clock runs ahead, so its writes win everything — and read-modify-write updates silently revert to the value they replaced.

## Scenario

A replicated key-value store runs three nodes with no primary. Any node accepts a write for any key and replicates it to the other two. The links have different delays, so the peers do not necessarily see a write at the same moment. When two writes to the same key meet, a resolver decides which survives, and every node must reach the same decision or the nodes would drift apart permanently.

The store stayed available and every node kept accepting writes. Updates nonetheless began disappearing: a value would be written, confirmed, and then quietly revert to something a user had already replaced.

Reviewing the run, one node's writes won essentially every contested key, and the discarded writes were the ones made after reading someone else's value. A user who read the current value and updated it would find their update gone, replaced by the very value they had just read.

The evidence records every write with the clock value it carried and the final winner of each key, each write with the clock its node read and whether it was kept, the replication of every write to every peer, and per-node logs.

## Requirements

- Never discard a write made after reading a value in favour of the value it replaced.
- Keep the nodes in agreement: every node ends up holding the same value for every key.
- Keep all three nodes accepting writes — availability must not regress.
- Keep resolution deterministic, so order of arrival does not change the outcome.
- Do not modify the tests.

## Edge cases to handle

- Two genuinely concurrent writes with no causal relationship
- Links delivering the same pair of writes in different orders to different peers
- A node whose clock is ahead of the others
- Ties that need a deterministic, node-agnostic tiebreaker
- A write arriving after the key already converged

## What interviewers look for

Whether you replace physical time with causality — a logical clock that captures what each write saw — instead of trying to fix the clocks. A full-marks answer preserves causal ordering, breaks genuine concurrency deterministically on every node, and can explain why wall-clock last-write-wins makes the fastest clock the winner.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/clock-skew-breaks-conflict-resolution
