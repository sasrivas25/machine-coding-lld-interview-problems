# Coarse Lock Convoy in a Shared Registry

Difficulty: Hard. Core topic: lock granularity, read concurrency, convoys.

The runtime-diagnostics round. A feature-flag registry guards both its cheap read path and its expensive view rebuild with one exclusive lock, so readers serialise behind the rebuild and extra threads add no throughput while CPU sits idle.

## Scenario

A platform gates every request through a feature-flag registry. The registry holds a set of targeting rules and serves a derived view of them: for each flag it resolves the rollout bucket and the segment a request belongs to. Callers ask for one flag at a time and expect an answer immediately.

The derived view is expensive to build relative to a lookup, so it is cached and rebuilt only when the rule set is invalidated. Building it walks every rule and folds a rollout hash for each one.

Single-threaded, the registry is correct: lookups return the right decision, an unknown flag returns nothing, and invalidation produces a new view. Under concurrent load it stops scaling — request latency tracks the cost of the rebuild rather than the cost of a lookup, adding reader threads adds no throughput, and server CPU stays low while requests queue.

A thread dump and a convoy summary captured while concurrent readers were inside the registry are in `artifacts/`.

## Requirements

- Let readers occupy the read path at the same time.
- Make an invalidation cause exactly one rebuild, no matter how many readers observe it.
- Never let a reader see a half-built view — each reader's view checksum must match its own entries.
- Keep derived decisions identical: the same rule set yields the same buckets and segments.
- Keep `lookup`, `view`, `invalidate`, and the metrics accessor unchanged in name and shape.

## Edge cases to handle

- Many readers arriving at an invalidated view simultaneously
- A second invalidation landing while a rebuild is in progress
- Publishing the rebuilt view atomically rather than mutating it in place
- An unknown flag during a rebuild
- Rebuild count asserted by the metrics accessor

## What interviewers look for

Whether you distinguish mutual exclusion from the actual requirement — one rebuild, many concurrent readers. A full-marks answer separates read access from rebuild coordination, publishes the new view as an immutable snapshot, and can explain why low CPU with queued requests is the signature of a convoy rather than overload.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/coarse-lock-convoy-in-shared-registry
