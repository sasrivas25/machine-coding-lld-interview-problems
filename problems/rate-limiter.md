# API Rate Limiter

Difficulty: Medium. Core topic: time-based accounting, concurrency.

One of the most common backend rounds because almost every real API needs one, and it exposes exactly the skills that get graded: correct time-based accounting, safe concurrency, and clean edge cases. The tricky part is not the formula — it is correctness when many callers hit the same client bucket at the same instant.

## Scenario

You maintain a backend service that fronts a public API. Each client gets a fixed budget of requests over time, enforced by a token-bucket policy: tokens refill continuously at a configured rate up to a capacity, and a request is admitted only when at least one token is available. The service has the shape of a rate limiter but misbehaves under load: racing requests against the same bucket admit more than they should, one noisy client can starve others, and timestamps that arrive equal or slightly out of order corrupt the token count.

## Requirements

- Refill → check → consume is one atomic critical section per bucket; 20 racing requests admit exactly the capacity.
- Buckets are keyed by scope and client, so one client cannot drain another's budget.
- No shared global lock serialising unrelated clients.
- Refill adds tokens only when real time has advanced, clamps to capacity, and a backward clock never drains tokens.
- Rejected requests get a correct retry-after.
- Bad policy configuration fails loudly, not by silently mis-enforcing.

## Edge cases to handle

- Many concurrent requests racing the last token of one bucket
- Equal timestamps on consecutive requests
- Timestamps arriving slightly out of order
- A bucket left idle long enough to over-refill (must clamp)
- The very first request for a client (lazy bucket creation)

## What interviewers look for

Identifying the unit of contention: every admit decision for a given client must read tokens, refill by elapsed time, and consume — without another caller observing the same last token. Per-bucket locking, per-client isolation, and time handling that survives messy clocks. Token bucket and sliding window are graded on the same two things: time accounting and safe concurrent access.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/rate-limiter-coding-interview
