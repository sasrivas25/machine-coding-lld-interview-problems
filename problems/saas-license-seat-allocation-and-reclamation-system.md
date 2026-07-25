# SaaS License Seat Allocation and Reclamation System

Difficulty: Medium. Core topic: seat capacity, concurrency.

The concurrency round. A per-org seat cap that must hold under simultaneous requests, one active seat per user, reclaim accounting that keeps used-counts honest, and tenants that must never leak into each other. Staying correct when many requests race is the exam.

## Scenario

You are working on a partially implemented SaaS license seat-allocation backend, designed to surface capacity, multi-tenant isolation, lifecycle, and concurrency bugs that show up in real system-design interviews. Organizations have a seat capacity; users take active seats; idle seats past a threshold get reclaimed automatically. The intended invariants are strict — never exceed a per-org cap, at most one active seat per user in an org, used-seat counts consistent with reality, and complete isolation between organizations.

Several visible tests fail because parts of the repository and service logic are incomplete or incorrect. Under concurrent seat requests the cap is breached, a user ends up with more than one active seat, reclaim leaves the used-count drifting, or an idle reclaim frees the wrong capacity. Your task is to read the models, repositories, services, and tests, understand the intended behavior, and fix the implementation without rewriting it or touching the tests.

## Requirements

- Allocate seats only within each organization's seat capacity.
- Guarantee at most one active seat per user within an organization.
- Keep used-seat counts consistent through allocation and reclamation.
- Automatically reclaim seats idle past the threshold.
- Keep organizations isolated so one org's seats never affect another.
- Stay correct when many seat requests arrive concurrently.

## Edge cases to handle

- Concurrent requests racing for the last remaining seat
- A user requesting a second seat while already holding one
- Reclaiming a seat and re-allocating that freed capacity
- Idle reclamation at exactly the threshold boundary
- Two organizations at capacity, allocations interleaving across both

## What interviewers look for

Whether capacity and the one-seat-per-user rule are enforced atomically, so two racing requests cannot both observe a free seat and both succeed. A full-marks answer keeps the used-count as a consistent consequence of allocate and reclaim rather than a separately drifting number, scopes every check to the organization, and makes idle reclamation deterministic at the boundary. The tell is a burst of concurrent requests against a nearly-full org settling on exactly the cap — never one over.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/saas-license-seat-allocation-and-reclamation-system
