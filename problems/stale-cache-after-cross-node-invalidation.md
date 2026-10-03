# Two Edge Nodes Served a Feature Flag That Had Been Changed Ten Minutes Earlier

Difficulty: Hard. Core topic: cache invalidation, generation checks.

The distributed-systems round. Invalidation is sent best-effort to the other nodes, and the nodes that miss it have no second line of defence — so they serve the old flag until someone restarts them.

## Scenario

Three edge nodes each keep a local cache of configuration values read from a config store. The store holds a value and a generation number per key, and the generation increases on every write. A write is applied at whichever node receives it; that node updates the store, updates its own cache, and sends an invalidation to the other two so they drop their copies and reload on next read.

A feature flag was switched off. It took effect immediately for some users and never took effect for others, and which group a user landed in depended on which edge node served them. One node served the new value straight away. The other two kept serving the old one until they were restarted.

The store held the correct value the whole time, every node could reach the store the whole time, nothing was reported as failed, and no read errored.

The evidence in `artifacts/` lists every invalidation sent, whether it arrived, and the reads served afterwards with the generation each returned against the generation current at that moment.

Four tests pass and four fail. The passing ones describe behaviour that must stay correct: every read is served, no node serves a superseded value when invalidations all arrive, node caches stay independent, and the cache absorbs reads rather than forwarding them to the store.

## Requirements

- Stop serving a superseded value when an invalidation is lost.
- Keep the cache absorbing reads — reading the full value on every request is not a fix.
- Keep node caches independent.
- Accept that invalidation delivery cannot be made reliable from `src/`.
- Keep every read served.

## Edge cases to handle

- An invalidation that never arrives at one node
- Generation unchanged, where the cached value is still valid
- Two writes in quick succession, so a node misses one of two
- A node with no cached entry for the key
- Concurrent reads during a refresh

## What interviewers look for

Whether you add a cheap authority check — compare generations, fetch the value only when it moved — rather than trusting a best-effort message or giving up on caching. A full-marks answer bounds staleness without collapsing the hit rate, and can explain why push invalidation always needs a pull fallback.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/stale-cache-after-cross-node-invalidation
