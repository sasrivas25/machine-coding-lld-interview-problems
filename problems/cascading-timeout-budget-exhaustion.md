# Requests Ran Far Past the Deadline the Caller Was Waiting On

Difficulty: Hard. Core topic: deadline propagation, timeout budgets.

The distributed-systems round. Four hops each apply a generous 200 ms timeout against a client that waits 250 ms, so no hop ever times out, every service reports itself healthy, and the chain keeps working on answers nobody will receive.

## Scenario

A checkout edge accepts a request and calls four services in sequence: cart, pricing, inventory, then ledger. Each hop normally takes 20 ms, so a healthy request finishes in about 80. The client waits 250 ms before giving up, and each hop applies its own timeout of 200 ms — generous next to 20 ms of normal work.

All four services then slowed down at once. None failed, and no hop ever hit its own timeout, so every service reported itself healthy throughout. Requests nevertheless ran far longer than the client waits. Work continued deep in the chain long after the client had stopped waiting, and the services stayed busy producing results nobody would receive, which made the slowdown worse.

The evidence records every hop call with its start time, how much of the request's time had already elapsed, and the timeout it was granted, alongside each request's outcome and end-to-end latency, plus the edge's own log. Comparing elapsed time against granted timeout, and both against the client deadline, shows the problem.

## Requirements

- Never let a request run longer than the deadline the caller is waiting on.
- Start no work for a request whose deadline has already passed.
- Let requests that genuinely fit still succeed — a single slow hop inside the overall deadline completes normally.
- Make every request settle one way or the other.
- Do not modify the tests.

## Edge cases to handle

- The remaining budget shrinking to near zero before the last hop
- A hop whose own timeout exceeds the remaining budget
- One slow hop that still leaves the chain inside the deadline
- A deadline already expired on arrival at the edge
- Cancelling in-flight work rather than letting it finish unread

## What interviewers look for

Whether a deadline becomes a value carried through the chain rather than a per-hop constant. A full-marks answer computes the remaining budget at each hop, refuses to start expired work, and can explain why per-hop timeouts that sum beyond the client's patience guarantee this failure.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/cascading-timeout-budget-exhaustion
