# Unbounded Cache Heap Exhaustion

Difficulty: Hard. Core topic: retention, cache eviction, heap analysis.

The runtime-diagnostics round. A freight quoting service dies a few hours after every restart while reporting a near-zero hit rate on highly repetitive traffic — a cache that bounds nothing and a key that never matches.

## Scenario

You inherit the quoting service behind a cross-border parcel platform. It has one meaningful operation: given a shipment request, return a price. A request carries the shipping lane, a service level, a weight, a declared value, caller-supplied attributes, and the raw payload the edge received. Pricing is table-driven — a tariff table loaded at startup maps lane, service level, and weight band to a base rate, fuel percentage, and handling fee.

Because the tariff lookup is the expensive part, the service was given a quote cache with a configured capacity and a small metrics buffer recording the outcome of every request.

The pod is restarted nightly. Within hours resident memory climbs, collection stops reclaiming anything meaningful, and the pod is killed. Traffic during that window is ordinary. Two details stand out: the reported hit rate is near zero on highly repetitive traffic, and the reported capacity is well below the number of entries the process appears to be holding.

A retention snapshot captured after a fixed 2000-request workload is in `artifacts/`, as plain text — no profiler needed.

## Requirements

- Keep live cache entries at or below the configured capacity, never growing with requests served.
- Let no request context outlive the request that created it.
- Serve repeated shipments with identical pricing inputs from the cache.
- Keep a cached price equal to what a fresh computation would produce.
- Keep every steady-state number unchanged when the workload doubles.
- Do not touch the tests, `verify.sh`, its pinned runtime flags, or `artifacts/`.

## Edge cases to handle

- A cache key that includes per-request data, so no two requests ever collide
- Entries inserted without eviction once capacity is reached
- The metrics buffer retaining the full request rather than its outcome
- Emptying the cache at the end — measured mid-workload, so it is not a fix
- Keeping the design cached rather than removing the cache entirely

## What interviewers look for

Whether you read the retention evidence before editing code, and whether you identify both defects: the key that defeats hits and the structure that defeats eviction. A full-marks answer separates "what is retained" from "why it never gets reused", and knows that raising the heap ceiling is the non-answer.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/unbounded-cache-heap-exhaustion
