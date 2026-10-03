# Thread-Pool Starvation from Nested Tasks

Difficulty: Hard. Core topic: pool starvation, task decomposition.

The executor-liveness round. Small cycles finish, but when batch count reaches pool size every worker waits for shard tasks queued to the same saturated pool.

## Scenario

A telemetry service submits one outer task per batch to a fixed executor. Each outer task splits its batch into shards, submits those inner tasks back to the same executor, and waits for them. With eight batches and eight workers, all workers are occupied by parents waiting for children that cannot start. CPU and reference lookups remain at zero.

Increasing the pool merely moves the threshold. The result and throttle semantics are already correct and must not change.

## Requirements

- Eliminate nested blocking on work submitted to the same pool.
- Complete both ordinary and much larger batch cycles.
- Produce one output per input record in input order.
- Preserve the configured pool size and lookup throttle.
- Propagate unknown-region errors correctly.

## Edge cases to handle

- Batch count equal to or greater than worker count
- Partial final shards
- Failure in one shard while others are active
- Maintaining deterministic output order
- Shutdown after a failed cycle

## What interviewers look for

Whether you identify thread-pool starvation from the dump and flatten task ownership so workers execute leaf work rather than wait inside the pool. Wider pools and longer deadlines mask the structural dependency; separate executors or caller-side orchestration can also work when lifecycle is handled carefully.

---

Practice this in a real repo with a failing test suite → https://gronex.org/thread-pool-starvation-nested-tasks-coding-problem
