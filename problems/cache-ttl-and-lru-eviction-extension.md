# In-Memory Cache: TTL Expiry + LRU Eviction

Difficulty: Medium. Core topic: caching, LRU eviction.

The extend-a-working-primitive round. You start with a correct unbounded key-value cache and add the two features every production cache eventually grows: per-entry expiry and a bounded size with least-recently-used eviction — where the subtle part is how the two interact.

## Scenario

You inherit a small, correct, unbounded in-memory key-value cache exposing `put`, `get`, and `size`. Time is injected via a `now` argument, so there is no wall clock and no sleeps — every scenario is fully deterministic. The base behaviour works today; your job is to grow it without regressing it. The same behaviour and the same deterministic scenarios are shared across Python, Java, and C++, and `./verify.sh` in a language directory checks your work.

The interesting failures live at the seam between the two new features. An expired entry that still occupies a slot, or a `get` that reads a value but forgets to refresh its recency, quietly changes which key gets evicted next.

## Requirements

- TTL expiry: an entry stored with a time-to-live is a miss when read at or after `now >= put_now + ttl`, and must be purged on that read.
- Entries stored without a TTL never expire.
- LRU eviction: with a configured `max_size`, inserting a new distinct key into a full cache evicts the least-recently-used entry.
- Both `get` and `put` on an existing key refresh that key's recency.
- Expired entries do not count toward capacity — purge them before evicting a live entry, so an expired slot frees capacity first.
- Preserve the existing `put` / `get` / `size` contract exactly.

## Edge cases to handle

- Reading an entry exactly at its expiry instant (`now == put_now + ttl`).
- A full cache where the "least recently used" key was just refreshed by a `get`.
- Inserting into a full cache that contains an expired entry — the expired slot is reclaimed before any live eviction.
- Re-`put`ting an existing key: it updates the value and recency, not the count.
- Mixing TTL and no-TTL entries under capacity pressure.

## What interviewers look for

Whether recency and expiry are tracked as first-class invariants rather than bolted on. A full-marks answer keeps eviction O(1) with the right data structures, treats expiry as a read-time purge that happens before eviction decisions, and can state precisely which key leaves the cache in every ordering of reads, writes, and time advances.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/cache-ttl-and-lru-eviction-extension
