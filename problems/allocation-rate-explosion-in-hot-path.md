# Allocation Rate Explosion in a Telemetry Hot Path

Difficulty: Hard. Core topic: allocation profiling, defensive copying.

The GC-throughput round where the live heap stays tiny but the collector runs constantly. Correct telemetry enrichment allocates transient keys, maps, tag sets, and copies on every event even though its configuration is immutable.

## Scenario

An enrichment stage derives routing data and attaches tenant, region, and tags to each inbound event. Production throughput falls as event rate rises; GC logs show frequent collections that reclaim almost everything, while heap captures show no growing retained set.

An allocation ledger identifies work proportional to configuration size inside the hot path. The digest over enriched output is pinned, so behavior cannot be simplified away.

## Requirements

- Keep enriched output byte-for-byte equivalent.
- Reduce allocated bytes per event to a small bounded budget.
- Make per-event cost independent of routing-table size.
- Reuse immutable configuration-derived data safely.
- Preserve fallback routing and public API behavior.

## Edge cases to handle

- Unknown routes using the fallback region
- Mutable caller input versus immutable returned data
- Configuration with eight versus dozens of routes
- Repeated enrichment of the same event
- Avoiding a cache that becomes a retention leak

## What interviewers look for

Whether you distinguish allocation churn from a leak and use allocation-site evidence. Strong repairs precompute immutable lookup state, avoid defensive copies that provide no isolation, and construct only the genuinely per-event result objects.

---

Practice this in a real repo with a failing test suite → https://gronex.org/allocation-rate-explosion-in-hot-path-coding-problem
