# Allocation Rate Explosion in a Telemetry Hot Path

Difficulty: Hard. Core topic: allocation rate, defensive copying, hot paths.

The runtime-diagnostics round. The service spends a large fraction of its time collecting garbage while a heap dump shows almost nothing alive. Nothing leaks — the hot path simply manufactures garbage on every event.

## Scenario

A telemetry platform enriches every inbound event before storage. The enrichment stage takes an event, builds a routing key from its source and id, resolves the region its edge reports to, and attaches the tenant and the active tag set from an enrichment configuration. That configuration is loaded once at startup, never changes, and is only read.

The stage is correct: every event gets the right key, bucket, region, tenant, and tags, and output is stable across runs. It is not cheap. At production rates collector pauses are frequent, throughput falls as volume rises, and yet the process retains no more memory than at idle. Four allocation sources hide in the per-event path — a defensive copy of the immutable configuration, a key built by repeated concatenation, a boxed counter, and an eagerly formatted debug message.

A collector log and an allocation ledger captured from a real run are in `artifacts/`.

## Requirements

- Allocate a small, bounded amount per event.
- Keep enriched output identical — a workload digest is pinned in the suite.
- Keep `enrich` taking an event and returning an enriched event with the same fields.
- Do not change the tests or the workload.

## Edge cases to handle

- Copying a configuration that is already immutable and shared
- Key construction that allocates once per concatenation
- Counters that box on every increment
- Log messages formatted before the level check
- Tag sets copied per event instead of shared

## What interviewers look for

Whether you read allocation rate as a distinct failure mode from retention — the heap is small, the churn is enormous. A full-marks answer removes each allocation source, justifies why sharing the config is safe given immutability, and leaves the computed output bit-for-bit unchanged.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/allocation-rate-explosion-in-hot-path
