# One Slow Consumer Is Holding Up Every Other Consumer

Difficulty: Hard. Core topic: fan-out isolation, per-subscriber queues.

The distributed-systems round. An event broker delivers to five consumers in lockstep, so an analytics deployment that slowed one of them puts billing minutes behind — and the delay grows for as long as the incident lasts.

## Scenario

An internal event broker fans every domain event out to five consumers: billing, the search index, the audit log, outbound webhooks, and analytics. Each handles an event in about five milliseconds and events arrive every forty, so there is ample room.

After an analytics deployment, billing started running minutes behind. Nothing about billing changed and billing is not slow, but its events now arrive late and the delay grows for as long as the incident lasts. Every consumer shows the same pattern, and the one consumer that genuinely is slow is only slightly worse off than the rest.

The capture in `artifacts/` samples the delay between publication and handling for each consumer over the incident.

## Requirements

- Make a consumer that falls behind fall behind on its own.
- Keep the slow consumer receiving events — disconnecting it is not the fix.
- Keep healthy consumers' delay flat during the incident.
- Keep every consumer receiving every event.
- Do not modify the tests.

## Edge cases to handle

- One consumer slower than the publish interval
- Several consumers slow at once
- Per-consumer backlog growth and its bound
- Ordering within a single consumer's stream
- Recovery, where the slow consumer drains its own backlog

## What interviewers look for

Whether delivery becomes independent per subscriber rather than a synchronous loop over handlers. A full-marks answer gives each consumer its own queue and progress, keeps healthy consumers unaffected, and can explain why lockstep fan-out makes the slowest consumer the broker's clock.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/slow-subscriber-stalls-broadcast
