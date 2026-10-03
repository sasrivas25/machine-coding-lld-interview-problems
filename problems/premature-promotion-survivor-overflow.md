# Premature Promotion and Survivor Overflow

Difficulty: Hard. Core topic: object lifetime, generational GC.

The generational-GC round. A nightly rollup holds every intermediate record until the entire input has been grouped, forcing short-lived data to survive young collections and promote into the old generation.

## Scenario

A batch job folds 262,144 metric records across 64 series into counts, totals, minima, and maxima. The starter implementation first materializes per-series buckets, keeping all boxed values reachable. GC evidence shows a rising post-young-collection floor, shrinking tenuring thresholds, survivor overflow, and eventual full collections that reclaim almost all promoted data.

The aggregate is correct and pinned. The defect is lifetime shape, not arithmetic or a permanent leak.

## Requirements

- Preserve every aggregate and the canonical digest.
- Fold records incrementally into bounded per-series state.
- Bound temporary chunks independently of total input size.
- Keep long-lived/tenured growth within budget.
- Leave runtime flags and collector configuration unchanged.

## Edge cases to handle

- Empty series and sparse series IDs
- A final partial processing chunk
- Minimum/maximum initialization
- Repeated job runs in one process
- Maintaining deterministic output ordering

## What interviewers look for

Whether you interpret rising survivor occupancy as an object-lifetime problem and stream the reduction rather than tuning generation sizes. The strongest repair keeps only O(series count + fixed chunk size) state, allowing intermediates to die young.

---

Practice this in a real repo with a failing test suite → https://gronex.org/premature-promotion-survivor-overflow-coding-problem
