# Unbounded Cache Heap Exhaustion

Difficulty: Hard. Core topic: cache eviction, retained heap.

The heap-retention round. A freight quote service is constructed with a 128-entry cache and a 256-sample metrics buffer, yet memory grows with every unique request until the process must be recycled.

## Scenario

`PricingService` caches expensive quote calculations over a hot workload with a long tail. Heap histograms and referrer chains show request-derived keys and metadata retained through cache and metrics structures. Prices remain correct, but the number of live entries scales with total traffic rather than configured capacity.

Tests repeat the workload at twice the size, preventing a repair that merely raises a threshold or delays exhaustion.

## Requirements

- Enforce the configured cache capacity with a deterministic eviction policy.
- Keep recent metrics bounded at their configured capacity.
- Preserve quote values and cache hits for hot keys.
- Release request metadata that is not part of a canonical key or result.
- Make retained objects independent of total request count.

## Edge cases to handle

- Updating an existing key without growing the cache
- Capacity zero or one
- Correct recency after a cache hit
- Eviction under a hot-set/long-tail workload
- Mutable request metadata passed into key construction

## What interviewers look for

Whether you use the heap evidence to identify the retaining containers and enforce actual bounds, usually with LRU semantics and minimal immutable keys. Weak references or manual collection do not fix a strongly reachable, logically unbounded cache.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/unbounded-cache-heap-exhaustion
