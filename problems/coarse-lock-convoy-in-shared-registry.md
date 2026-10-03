# Coarse Lock Convoy in a Shared Registry

Difficulty: Hard. Core topic: lock convoy, read-path concurrency.

The thread-dump performance round. A feature-flag registry is correct on one thread, but concurrent readers queue behind a coarse lock while an expensive immutable view is rebuilt.

## Scenario

Callers look up individual flags from a cached derived view. Invalidation marks that view stale, and the next reader rebuilds it by walking every rule. The starter implementation holds one exclusive lock across both ordinary reads and the full rebuild. Thread evidence shows low CPU, many readers parked on the same monitor, and throughput that does not improve with additional threads.

The repaired registry must never expose a half-built view or trigger duplicate rebuilds.

## Requirements

- Allow multiple readers to execute the read path concurrently.
- Publish only fully constructed immutable views.
- Trigger exactly one rebuild per invalidated generation.
- Preserve lookup decisions, checksums, and public APIs.
- Keep invalidation safe while reads are in flight.

## Edge cases to handle

- Many readers observing invalidation simultaneously
- An invalidation arriving during a rebuild
- Unknown-flag lookups
- Rebuild failure without publishing partial state
- Metrics that accurately report concurrent readers and rebuilds

## What interviewers look for

Whether you use the dump to identify contention rather than deadlock and reduce the critical section around publication. Strong solutions use immutable snapshots and double-checked generation state or an appropriate read/write coordination primitive without sacrificing single-rebuilder semantics.

---

Practice this in a real repo with a failing test suite → https://gronex.org/coarse-lock-convoy-in-shared-registry-coding-problem
