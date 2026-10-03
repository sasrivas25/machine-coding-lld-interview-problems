# Healthy Nodes Keep Getting Thrown Out of the Fleet

Difficulty: Hard. Core topic: failure detection, membership stability.

The distributed-systems round. A coordinator evicts a node the moment a heartbeat is late, so a busier network segment turns ordinary jitter into a permanent reshuffle of the shard map.

## Scenario

A coordinator tracks which nodes are in a five-node fleet and owns the shard assignment. Nodes heartbeat every 100 ms. If the coordinator stops hearing from a node it removes it and spreads that node's shards across the survivors, restoring it when it hears from it again.

Since the fleet moved to a busier network segment, the shard map has not settled. The coordinator removes and restores nodes repeatedly, and every change moves a large fraction of the shards, forcing cache rebuilds and driving tail latency up. The nodes themselves are healthy: they are up the whole time, they send every heartbeat they are supposed to, and the coordinator's own counters show it received every one.

One node in the captured period really was powered off, and the coordinator handled that correctly.

A capture of one period is in `artifacts/`, including how many heartbeats each node sent and how many arrived.

## Requirements

- Stop evicting nodes that never stopped sending.
- Keep removing a genuinely dead node promptly and reassigning its shards.
- Keep the shard map stable under ordinary heartbeat jitter.
- Keep reassignment cost proportional to the membership change.
- Do not modify the tests.

## Edge cases to handle

- A heartbeat arriving late but within a tolerance window
- Several consecutive late heartbeats from one node
- The genuinely powered-off node, which must still be evicted
- A node restored immediately after eviction, causing churn
- Jitter affecting several nodes in the same window

## What interviewers look for

Whether failure detection becomes tolerant of jitter — a threshold of missed beats, with hysteresis — without blunting detection of real death. A full-marks answer keeps eviction prompt for the dead node, stops flapping for the live ones, and can explain why a single missed beat is not evidence of failure on a shared network.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/heartbeat-flapping-evicts-healthy-nodes
