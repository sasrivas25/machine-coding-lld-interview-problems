# Tenants Are Allowed Four Times Their Contracted Rate

Difficulty: Hard. Core topic: distributed rate limiting, shared quota state.

The distributed-systems round. A per-second limiter that was correct on one machine now runs on four, so every tenant gets four times their contract — and adding gateways makes it worse.

## Scenario

Every tenant of the public API has a contracted ceiling of sixty requests per second. The gateway fleet enforces it, and the code is short enough to read in one sitting: count what this tenant has used in the current second, refuse anything past sixty. It was written when the service ran on one machine, and it was correct then.

The fleet runs on four machines now. Capacity planning is built on contracted ceilings and the numbers have stopped adding up: the busiest tenant sustains several times what it pays for, the backend behind the gateway is sized for the contracted total and is not coping, and a tenant told they were being throttled produced traffic logs showing they were not. Adding gateways to cope made it worse, which nobody could explain.

A limit is a property of the tenant, not of the machine that happens to answer. Four gateways enforcing sixty each is not a limit of sixty.

The captured evidence shows, for each tenant and each one-second window, how many requests were offered, how many were let through, and how those admissions split across the fleet.

## Requirements

- Hold each tenant to the contracted ceiling across the whole fleet.
- Make the ceiling independent of how many gateways are running.
- Still allow a tenant with demand to spare the full sixty, not a fraction.
- Keep enforcement correct when admissions are unevenly distributed across gateways.
- Do not modify the tests.

## Edge cases to handle

- Traffic arriving at one gateway only
- Traffic split unevenly across all four
- Window boundaries, where a tenant could double up across the edge
- Concurrent admission decisions for the same tenant on different gateways
- A tenant well under their ceiling never being refused

## What interviewers look for

Whether the counter becomes shared state with an atomic check-and-increment rather than per-process arithmetic. A full-marks answer keeps the fleet-wide ceiling exact under concurrency, avoids per-node static splits that waste quota, and can explain why fixed per-gateway allocations fail the "full sixty on demand" requirement.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/distributed-rate-limit-drifts-per-node
