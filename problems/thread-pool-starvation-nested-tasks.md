# Thread Pool Starvation from Nested Task Submission

Difficulty: Hard. Core topic: pool starvation, task decomposition, blocking waits.

The runtime-diagnostics round. A batch enrichment service submits subtasks to the pool it is running on and then blocks waiting for them, so at production size the pool fills with waiters and nothing finishes.

## Scenario

A telemetry platform enriches device records in batches. A batch arrives, each record is decorated with reference data for its region, and the enriched records come back in submission order.

Enrichment runs on a fixed-size worker pool. The service splits each batch into shards so a large batch does not monopolise one worker, and a throttle bounds how many reference lookups may be in flight so the reference directory is not overwhelmed.

On a light workload it is correct — a handful of batches enrich cleanly and output matches input record for record. Under a production-sized workload it stops rather than slows: every worker is occupied, the work queue grows, and the completed batch count stays where it started until the caller's deadline expires. Adding more batches makes it worse, not slower.

A thread dump and a pool state snapshot captured during a stalled workload are in `artifacts/`.

## Requirements

- Make the workload complete within its deadline.
- Keep enriched output identical and in submission order.
- Respect the in-flight lookup throttle.
- Keep pool size, shard size, and deadline as the caller sets them — a fix that only works with a bigger pool is not a fix.
- Do not modify the tests.

## Edge cases to handle

- A task that blocks on the completion of tasks queued behind it
- Throttle permits held while waiting for a nested task
- Batches smaller than one shard
- Ordering of results after decomposition changes
- Exceptions inside a shard not stranding the batch

## What interviewers look for

Whether you recognise the hold-and-wait pattern inside one pool as a starvation deadlock, and restructure the work so no worker blocks on work that needs a worker. A full-marks answer keeps shard decomposition and the throttle, shows completion inside the fixed deadline, and never argues its way to a larger pool.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/thread-pool-starvation-nested-tasks
