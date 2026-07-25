# Incident Debugging: Checkout Hangs When Recommendations Stall

Difficulty: Medium. Core topic: timeouts, graceful degradation.

The on-call incident round. One downstream dependency starts stalling, and because a single call never gives up, the stall walks straight up the stack and takes the whole checkout service down. Your job is to read the log, find the missing deadline, and make the service degrade instead of hang.

## Scenario

You are on call for a checkout service. At 14:04 UTC its `recommendations-service` started stalling after a deploy, and within five minutes checkout was down: p99 latency jumped from ~130 ms to over 240 s, in-flight requests climbed until the worker pool was exhausted, the gateway shed traffic with `503 overloaded`, and `/healthz` failed until the instance was pulled from the load balancer. The bundled `logs/incident.log` in each language project is where you start — one line gives it away: requests eventually completed with `latency_ms=300218 status=200 source=live`. The service never stopped waiting on the stalled dependency.

Everything is modeled in process and deterministically — no HTTP server, no threads, no real sleeps. An injected fake `Clock` provides `now()`, and an injected `RecommendationsClient` takes `(product_id, timeout_seconds)` and reports each call's duration on the fake clock; a call that would exceed a positive timeout aborts with a `DownstreamTimeout` instead of waiting out the full latency. Each handled request holds an in-flight capacity slot for as long as it was blocked, requests beyond capacity are shed with `503 overloaded`, and `health()` reports `saturated` once capacity is full.

## Requirements

- A healthy dependency still returns live recommendations, unchanged.
- A slow-but-acceptable dependency (≤ 2 s) is still served live, not cut off.
- A stalled dependency (300 s) costs at most the 2-second deadline on the fake clock and returns a degraded fallback (`200`, source `fallback`).
- Under a burst of stalled requests, every request is answered, in-flight capacity never overflows, and health stays `ok`.
- Load shedding stays intact: when capacity genuinely is full, requests are still shed with `503`.
- Fix the handler; do not modify the tests.

## Edge cases to handle

- A downstream call landing exactly at the 2-second deadline versus just over it.
- A slow-acceptable call that must not be prematurely aborted.
- A burst arriving while earlier stalled calls are still resolving.
- Distinguishing a genuine capacity-full shed from a stall-induced pile-up.
- Health transitioning to `saturated` only for real saturation, not for a recoverable stall.

## What interviewers look for

Whether you diagnose from the evidence before touching code — the log line with the 300-second live response points straight at a call with no enforced deadline. A full-marks answer bounds the downstream call at the deadline, converts a timeout into a fallback rather than an error, and preserves the two tests that already pass (healthy path, load shedding) while fixing the two that fail. The distinction that separates strong candidates: a timeout is not a failure to surface, it is a budget to spend and then fall back.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/incident-missing-timeout-hang-debugging
