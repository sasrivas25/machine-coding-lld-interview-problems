# Rate Limiter: Sliding-Window Algorithm & Per-Tier Limits

Difficulty: Medium. Core topic: rate limiting, sliding window.

The extend-the-limiter round. You start with a working fixed-window rate limiter and add the two things it's missing in production: a sliding window that closes the boundary-burst hole, and per-tier limits enforced independently per key.

## Scenario

You are given a working per-key fixed-window rate limiter. `allow(key, now)` returns `true` for up to `max_requests` calls per `window_seconds` window (per key) and `false` once the limit is hit, until the aligned window rolls over. Time is injected via `now`, so all behaviour is deterministic. It works — until you look at the window boundary, where the classic burst leaks through: with a limit of 5 per 10s, five requests at `t=9` and five more at `t=11` are all allowed, because a fresh calendar window opens at `t=11`.

Your job is to extend the limiter without breaking the existing fixed-window behaviour, adding a selectable sliding-window algorithm and configurable per-tier limits. Each language project ships a bundled test suite and a `verify.sh`; the base tests already pass and the new ones fail until you implement them.

## Requirements

- Keep the existing fixed-window `allow(key, now)` behaviour intact.
- Add a `sliding` algorithm selectable per limiter instance.
- Sliding-window rejects once more than `max_requests` fall within any trailing `window_seconds`, and allows again as old requests age out.
- Support configurable tiers such as `{"free": (5, 10), "pro": (100, 10)}` via `allow_tier(key, tier, now)`.
- Count each tier independently per key, with keys independent of one another.
- Treat an unknown tier as an error.

## Edge cases to handle

- The boundary burst: 5 requests at `t=9` then 5 at `t=11` must be rejected under sliding-window.
- Requests aging out of the trailing window so capacity returns.
- The same key evaluated under different tiers.
- Distinct keys that must never share a counter.
- An `allow_tier` call naming a tier that doesn't exist.

## What interviewers look for

Whether the sliding window is genuinely trailing rather than a renamed fixed window, and whether per-key, per-tier state stays cleanly separated. A full-marks answer keeps the two algorithms behind one selectable interface without duplicating logic, ages out old timestamps correctly, and reasons about memory as the request history grows — so the limiter that fixes the boundary burst doesn't quietly grow unbounded.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/rate-limiter-sliding-window-and-tiers-extension
