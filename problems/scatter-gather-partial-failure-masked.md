# Search Reported Complete Results While Two Shards Were Refusing Requests

Difficulty: Hard. Core topic: partial failure, fan-out completeness.

The distributed-systems round. The coordinator merges whatever comes back and reports success, so two dead shards out of six look exactly like a healthy service — result counts drop by a third and every dashboard stays green.

## Scenario

A search coordinator answers queries by fanning out to six shards, waiting up to 120 ms for each, and merging whatever comes back into one ranked list. Each shard owns a disjoint slice of the corpus, so every shard contributes rows no other shard has. The answer carries the merged rows, a total, and a completeness flag.

Two shards started refusing requests for several minutes. Nobody noticed at the time. It surfaced days later from a customer: a saved search that normally returned two dozen results returned sixteen, and reported no problem doing so. Reviewing the window, result counts for many queries had dropped by a third and then recovered on their own.

No query failed, none timed out, latency was normal, and the dashboards showed a healthy service for the whole window — from the coordinator's point of view every query was answered successfully.

The evidence in `artifacts/` records how many shards each answer was actually built from alongside what that answer reported.

Four tests pass and four fail. The passing ones describe behaviour that must stay correct: every query is answered, a shard replying inside the timeout is not a failure, rows from healthy shards are still returned when others fail, and aggregator instances stay independent.

## Requirements

- Report an answer built from fewer than all shards as incomplete.
- Keep a failing shard from failing the query.
- Keep a healthy fan-out from being reported as degraded.
- Keep the total consistent with what was actually gathered.
- Keep aggregator instances independent of one another.

## Edge cases to handle

- A shard that times out versus one that refuses
- All six shards healthy, where completeness must be true
- All shards failing
- A shard returning zero rows legitimately
- Concurrent queries sharing no state

## What interviewers look for

Whether completeness is derived from what actually answered rather than defaulted to true. A full-marks answer tracks responding shards per query, surfaces degradation to the caller and to metrics, and can explain why silent partial results are worse than an error.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/scatter-gather-partial-failure-masked
