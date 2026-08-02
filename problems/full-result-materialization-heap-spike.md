# Full Result Materialization Heap Spike

Difficulty: Hard. Core topic: streaming, bounded memory.

The heap-profile round where an export configured for 128-row chunks still holds every result row until completion. Memory returns afterward, proving it is a workload-shaped spike rather than a leak.

## Scenario

A reporting service formats a database result set into delimited records and writes chunks to an output sink. The starter path eagerly converts the full iterator into a collection before chunking it. A 5,000-row capture already shows all source rows and formatted records live when the first chunk is written; production exports contain millions.

Output correctness is already pinned. The repair must change lifetime and traversal, not the exported bytes.

## Requirements

- Consume the result set incrementally.
- Hold no more source rows or formatted records than the configured chunk size.
- Write every row exactly once and in original order.
- Keep peak retention stable as result size grows.
- Avoid temporary-file or manual-GC workarounds.

## Edge cases to handle

- Empty and one-row result sets
- A final partial chunk
- Result sizes exactly divisible by chunk size
- Sink failure partway through export
- Iterators that cannot be rewound

## What interviewers look for

Whether you distinguish retention from leakage and remove eager materialization. A strong solution uses one bounded buffer, flushes and clears it deterministically, and demonstrates scale invariance between small and large exports.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/full-result-materialization-heap-spike
