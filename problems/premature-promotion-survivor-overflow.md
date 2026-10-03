# Premature Promotion and Survivor Overflow in a Batch Rollup

Difficulty: Hard. Core topic: generational GC, object lifetime, working-set shape.

The runtime-diagnostics round. A nightly rollup retains every intermediate value before reducing it, so per-batch state outlives the young generation, overflows the survivor space, and gets promoted — forcing full cycles that reclaim almost everything they walk.

## Scenario

A nightly rollup folds raw metric records into per-series summaries. Each run reads 262,144 records across 64 series and reduces them to a count, total, minimum, and maximum per series.

The numbers it produces are correct. It is also why the batch host now spends a visible fraction of every run inside garbage collection, and why its heap grows until the collector is forced into full cycles that walk everything and reclaim nearly all of it. The cause is lifetime, not volume: intermediate values are all held until the reduce step, so data that should die young survives long enough to be promoted.

Captured runtime evidence is in `artifacts/`. Read it before changing code.

## Requirements

- Reshape how intermediate state is held so per-batch data dies young.
- Change no number the job produces — the aggregate is pinned by digest.
- Keep peak live intermediate state within budget.
- Keep long-lived growth within budget, measured by the runtime's own accounting.
- Keep `AggregationJob`'s public surface working as the tests call it.
- Do not widen the heap, enlarge the young generation, change the survivor ratio, or switch collector — the flags in `verify.sh` are part of the problem.

## Edge cases to handle

- Reducing incrementally while keeping min/max and totals exact
- Floating-point or ordering differences that would change the digest
- Series that appear in only some batches
- The final partial batch
- Holding only one batch's worth of state at a time

## What interviewers look for

Whether you recognise the generational hypothesis being violated and fix the retention pattern rather than tuning the collector. A full-marks answer folds records into summaries as they stream, demonstrates that peak live state is independent of record count, and leaves the aggregate digest untouched.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/premature-promotion-survivor-overflow
