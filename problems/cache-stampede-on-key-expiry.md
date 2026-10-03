# A Slow Origin Received Thirty Times the Traffic It Should Have

Difficulty: Hard. Core topic: cache stampede, request coalescing.

The distributed-systems round. A 200 ms TTL in front of a hot key means that when the origin slows down, every miss during the load becomes another load — the edge amplifies the slowdown it is supposed to absorb.

## Scenario

An edge service serves product reads from an in-process cache in front of an origin. Entries live for 200 ms: a fresh key is served from cache, and a missing or expired key loads from the origin and stores the result. Half the traffic is for a single key.

The origin had a slow period — about 300 ms per load instead of 20 ms — and was expected to recover on its own. It did not. Origin load rose sharply during the slowdown, and it rose because of the edge, which sent many times more requests than while the origin was healthy. Origin CPU stayed pinned until traffic was shed manually, and the slower the origin got, the more work the edge sent it.

The edge served every read. Nothing failed. The origin simply received far more load while struggling than while healthy.

The evidence in `artifacts/` lists every load sent to the origin, with the key and the number of concurrent loads of that same key at that moment.

Four tests pass and four fail. The passing four describe behaviour that must stay correct: every read is served, values are never older than the TTL allows, a brief origin slowdown does not delay cold keys, and a healthy origin is not reloaded more than expiry requires.

## Requirements

- Collapse concurrent loads for the same key into one origin request.
- Keep serving every read.
- Never serve a value older than the TTL allows.
- Keep cold keys from waiting behind loads for other keys.
- Do not reload a healthy origin more than expiry requires.

## Edge cases to handle

- Many readers arriving in the window while one load is in flight
- A load that fails — followers must not be left waiting forever
- Different keys loading concurrently without blocking each other
- A key expiring again while its previous load is still running
- The first request for a cold key

## What interviewers look for

Whether you coalesce per key rather than guarding the cache with one lock, which would serialise unrelated keys. A full-marks answer makes the number of origin loads independent of concurrency, propagates failure to all waiters, and respects the TTL instead of extending it to dodge the problem.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/cache-stampede-on-key-expiry
