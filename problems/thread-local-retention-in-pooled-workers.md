# Thread-Local Retention in Pooled Workers

Difficulty: Hard. Core topic: thread-local lifecycle, worker pools.

The runtime-diagnostics round. Per-request state lives in thread-local storage on a pool that never recycles its threads, so request contexts accumulate for the life of the process — including overnight, with nothing in flight.

## Scenario

A payments platform settles through a five-stage pipeline on a fixed pool of worker threads. Each request gets a context holding the tenant, the principal, and a trace id, and every stage drops a breadcrumb so a failed payment can be reconstructed from its trail. Context and breadcrumbs live in thread-local storage so the stages need not thread them through every call.

The pool never grows and never shrinks: four workers, restarted only on redeploy.

Resident memory climbs from the moment traffic starts and never comes back down, including overnight when the pipeline is idle for hours. The pool is not growing, no queue is backing up, and the audit ring is exactly its configured size. Requests drain normally — nothing is stuck in flight and the settled counter matches upstream.

A heap capture taken while the pipeline is idle, with no requests in flight, still shows request contexts and breadcrumb objects in numbers that track total requests ever handled.

## Requirements

- Make per-request thread-local state unreachable once its request finishes.
- Keep reachable per-request state flat after the workload drains.
- Keep it flat at twice the workload size.
- Keep the breadcrumb trail available for the duration of a request.
- Do not modify the tests or the workload.

## Edge cases to handle

- A stage that raises, skipping the cleanup path
- A worker reused immediately by the next request, inheriting stale state
- Breadcrumbs appended but never cleared between requests
- The audit ring holding references beyond its logical capacity
- Nested or re-entrant pipeline invocations on one worker

## What interviewers look for

Whether you understand that thread-local lifetime equals thread lifetime, and that a pooled thread outlives every request. A full-marks answer clears thread-locals in a finally-equivalent scope, proves reachable state is flat after drain at two workload sizes, and does not confuse "requests complete" with "request state released".

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/thread-local-retention-in-pooled-workers
