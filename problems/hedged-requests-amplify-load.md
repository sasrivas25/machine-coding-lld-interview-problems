# A Slow Replica Turns Into a Fleet-Wide Overload

Difficulty: Hard. Core topic: hedged requests, load amplification.

The distributed-systems round. Unconditional second attempts after 50 ms fixed the tail and created a new failure mode: when one replica slows, the gateway doubles its own traffic into the replicas that were still healthy.

## Scenario

A search gateway spreads queries over three replicas. To keep the tail in check it uses a second attempt: if a query has not come back after 50 ms, the gateway sends the same query to another replica and takes whichever answer arrives first. This was added to fix a tail latency problem, and it worked.

The failure mode has since changed shape. When one replica gets slow, the whole fleet gets slow, and the gateway is measurably making it worse. Traffic into the gateway is flat, but attempts leaving it climb sharply, and the replicas that were healthy start queueing behind work no user sent them. Recovery takes far longer than the original slowdown.

The capture in `artifacts/` shows attempts per 100 ms window alongside the flat arrival rate, and per-replica totals with peak concurrency.

## Requirements

- Stop the gateway amplifying load during a slowdown.
- Keep the second attempt doing its job: an occasional slow query still returns inside the latency budget.
- Keep total attempts bounded relative to a flat arrival rate.
- Keep healthy replicas from queueing work no user sent.
- Do not modify the tests.

## Edge cases to handle

- A broad slowdown where most queries would qualify for a hedge
- An isolated slow query on an otherwise healthy fleet
- Cancelling the loser once one attempt answers
- Choosing a target that is not already the slow replica
- Recovery, where hedging should return to normal

## What interviewers look for

Whether hedging becomes budgeted — a bounded fraction of traffic, adaptive to observed latency — rather than a fixed rule applied to every slow request. A full-marks answer keeps tail protection for the isolated case, caps amplification during a broad slowdown, and can explain why an unconditional hedge is positive feedback on an overloaded fleet.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/hedged-requests-amplify-load
