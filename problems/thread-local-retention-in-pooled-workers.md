# Thread-Local Retention in Pooled Workers

Difficulty: Hard. Core topic: thread-local cleanup, worker pools.

The heap-dump round where request contexts survive long after work drains. The pool is fixed at four workers, but each immortal worker retains historical per-request state through thread-local storage.

## Scenario

A payment pipeline stores `RequestContext` and breadcrumb data in per-worker state so five stages can access it implicitly. Workers are reused for the process lifetime. The starter code initializes request-local collections but does not reliably clear them after successful or failed processing.

At idle, heap evidence still contains contexts and breadcrumbs proportional to all requests handled. The audit ring is bounded and the queue is empty, so neither is the retaining owner.

## Requirements

- Remove all request-scoped state when each request completes.
- Clean up on success, validation failure, and unexpected exception.
- Prevent one request from observing another's context.
- Keep worker reuse, pipeline behavior, and audit-ring bounds unchanged.
- Keep retention bounded during the workload, not only after it drains.

## Edge cases to handle

- Exceptions in an intermediate pipeline stage
- Reusing the same worker for many tenants
- Cleanup when state was only partially initialized
- Concurrent workers with independent local state
- Shutdown with requests still in flight

## What interviewers look for

Whether you recognize that thread-local lifetime follows the worker, not the request. The repair scopes initialization and cleanup with `finally`/RAII and removes all nested request-owned containers rather than clearing them only at batch completion.

---

Practice this in a real repo with a failing test suite → https://gronex.org/thread-local-retention-in-pooled-workers-coding-problem
