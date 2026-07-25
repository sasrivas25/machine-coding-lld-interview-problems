# Incident Debugging: Retry Storm Without Backoff

Difficulty: Medium. Core topic: retry backoff, resilience.

The production-incident round. A dependency blips for a minute, a client retries in a tight unbounded loop, and outbound traffic multiplies a hundredfold so the dependency can never recover — a self-inflicted outage born from a missing backoff.

## Scenario

You are on call for `checkout-api`. Its dependency `payments-auth` did a routine rolling deploy and briefly returned 503s — a blip that should have cost nothing. Instead the checkout path went down and stayed down: outbound request rate to `payments-auth` multiplied 100x within seconds, the dependency stayed unhealthy under the hammering, and alerts cascaded for ten minutes. The on-call notes in `logs/incident.log` hold the smoking gun — a single order with 800+ authorize attempts inside one millisecond, every retry logged with `delay_ms=0`, and attempt counters with no ceiling. The outbound call is modelled deterministically with an injected dependency and an injected clock, so the tests assert the exact timing of every attempt.

## Requirements

- Read `logs/incident.log` and locate the tight, unbounded retry loop in the client.
- Retry with capped exponential backoff: base delay 100ms, factor 2.
- Cap attempts at 5, taking delays on the injected clock.
- Raise a clean `RetriesExhausted` once the cap is hit, never loop forever.
- A failure that clears after two failures must succeed on exactly the third attempt.
- Do not edit the tests; make all three language projects pass and run `./verify.sh`.

## Edge cases to handle

- A transient failure that clears mid-sequence versus a persistent one
- The exact fake-clock schedule (0/100/300ms, then 700/1500ms) attempts must follow
- Stopping at the fifth attempt rather than a sixth
- Delays taken on the injected clock, never on real time or sleeps
- Surfacing `RetriesExhausted` instead of the raw dependency error

## What interviewers look for

Whether the retry policy is a bounded, backing-off good citizen rather than a busy loop that amplifies a blip into an outage. A full-marks answer reproduces the exact fake-clock schedule, treats the attempt cap and the exhaustion signal as first-class, and understands why immediate retries turn a recoverable dependency into a permanently overloaded one. Backoff is not decoration here; it is the difference between a blip and a ten-minute incident.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/incident-retry-storm-debugging
