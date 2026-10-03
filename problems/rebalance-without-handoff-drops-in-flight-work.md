# Jobs Vanish Whenever a Worker Joins or Leaves

Difficulty: Hard. Core topic: rebalancing, in-flight work handoff.

The distributed-systems round. Every autoscaling event reassigns partitions immediately, and the jobs running at that instant are accepted, dispatched, and never heard from again.

## Scenario

A router spreads jobs over a pool of workers by partition. When a worker joins or leaves, the partitions are spread over whoever is left and the router updates its map. Each job takes a little over a hundred milliseconds and builds scratch state on the worker that started it, so a job cannot be picked up halfway through somewhere else.

Autoscaling now adds and removes workers during the day, and every time it does, a batch of jobs is never heard from again. They are accepted, they are dispatched, and they never complete. Nothing errors and nothing is retried. The jobs that vanish are always the ones running when the pool changed, and the effect scales with how busy the pool is: a change during a quiet period loses a few, a change at peak loses many.

A capture of one period with three pool changes is in `artifacts/`, listing every job, the worker it went to, and whether it finished.

## Requirements

- Lose nothing when the pool changes.
- Keep workers taking over partitions on a pool change.
- Keep a job finishing on the worker that started it.
- Keep accounting complete: every accepted job completes or is reported.
- Do not modify the tests.

## Edge cases to handle

- A job in flight on a worker whose partition is reassigned
- A worker leaving while it still holds in-flight jobs
- Several pool changes in quick succession
- A worker joining with no in-flight work to inherit
- New jobs arriving for a partition mid-handoff

## What interviewers look for

Whether reassignment waits for a drain rather than taking effect instantly. A full-marks answer lets the outgoing owner finish its in-flight jobs before the new owner accepts work for that partition, keeps new dispatch correct during the transition, and can explain why stateful jobs make migration impossible and handoff mandatory.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/rebalance-without-handoff-drops-in-flight-work
