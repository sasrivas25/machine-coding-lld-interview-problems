# One Dependency Never Got Cut Off, Another Never Got Restored

Difficulty: Hard. Core topic: circuit breakers, state transitions, recovery.

The distributed-systems round. Two incidents filed a week apart with opposite symptoms — a breaker that never tripped during a full outage, and one that never closed after a 600 ms blip — turn out to share a single cause.

## Scenario

A search API calls a suggestion service on every request through a circuit breaker whose job is to stop sending traffic to a failing dependency and to resume once it is healthy.

In the first incident, the suggestion service returned errors for a full second and the search API sent it every single request for the entire outage — two hundred calls at full rate into a dependency answering none of them. The breaker never changed state. In the second, the suggestion service timed out for 600 ms and then recovered completely, and the search API refused to call it for the rest of the run: seventy requests still refused long after the dependency was healthy, cleared only by restarting the caller.

One team concluded the breaker was too lax. The other concluded it was too aggressive. Both incidents have the same cause, and a change that addresses one without the other is not a fix.

The evidence in `artifacts/` holds both incidents side by side, including every state transition the breaker made. Four tests pass and four fail; the passing ones describe behaviour that must stay correct — a healthy dependency is never cut off, and a brief blip of three failures does not trip anything.

## Requirements

- Trip the breaker when the dependency is genuinely failing, not on the first failure.
- Resume traffic once the dependency is healthy again, without a restart.
- Keep a healthy dependency from ever being cut off.
- Keep a three-failure blip from tripping anything.
- Do not modify the tests.

## Edge cases to handle

- Failures interleaved with successes inside the same window
- The trial request after the open period, and what its outcome means
- Concurrent callers observing the breaker mid-transition
- Timeouts counted the same as errors
- A dependency that recovers while the breaker is open

## What interviewers look for

Whether you find the shared defect in how outcomes are counted and state is advanced, rather than tuning one threshold in each direction. A full-marks answer implements the full closed/open/half-open cycle with a bounded window, proves both incidents fixed by the same change, and avoids tripping on a single failure.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/circuit-breaker-never-opens
