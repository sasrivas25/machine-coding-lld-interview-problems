# Ride Booking and Driver Matching System

Difficulty: Medium. Core topic: matching, dispatch lifecycle.

The Uber/Ola-style question of Indian machine coding rounds. Beneath the map pins it is a resource-allocation problem: riders request, a pool of drivers is available, and the system must pick the right driver, assign them exclusively, and give them back when things change. The same select–assign–release loop runs delivery dispatch, agent routing, and warehouse task allocation.

## Scenario

You maintain the matching service of a ride-hailing backend: riders request rides, drivers register availability, candidate drivers are ranked, one is assigned, and rides complete or cancel. The flow exists end to end and fails the way real dispatch systems fail: equidistant drivers are picked differently run to run, an assigned driver gets matched to a second ride, cancelled rides never return their driver to the pool, and completing a ride leaves the driver stuck in "on trip".

## Requirements

- Candidate selection filters availability and ranks by explicit criteria.
- Ties break deterministically: equal candidates always resolve the same way (e.g. by a stable secondary key).
- Assignment is exclusive: a matched driver is atomically removed from the available pool.
- Every ride exit path — completion, rider cancel, driver cancel — returns the driver to the pool.
- Driver availability is a state machine: available → assigned → on trip → available.
- Unknown riders/drivers and illegal requests are rejected with clear errors.

## Edge cases to handle

- Two equidistant drivers for one request
- A driver assigned to a ride while being considered for another
- A ride cancelled immediately after matching
- Re-requesting a driver right after their previous ride completes
- No available drivers at all

## What interviewers look for

Discipline in three places: selection must be deterministic (matching that depends on hash-map iteration order fails intermittently, and evaluators test with equal candidates on purpose), assignment must be exclusive everywhere instantly, and release must be complete on every exit path — leaked drivers are the bug that quietly empties a dispatch system. No geo math involved; the lifecycle is what gets graded.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/ride-booking-driver-matching-system-coding-problem
