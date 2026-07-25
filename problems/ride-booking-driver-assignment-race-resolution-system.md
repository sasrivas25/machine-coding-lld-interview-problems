# Ride Booking Driver Assignment Race Resolution System

Difficulty: Hard. Core topic: assignment races, ordered locking.

The contention round. Many rides chasing the same driver, a lease a driver can hold only once, and atomic transitions of both ride and driver under stable per-resource locks so exactly one ride wins and the driver is never double-booked.

## Scenario

You are given a backend for ride driver assignment under contention. A driver can hold at most one active lease — OFFERED or ASSIGNED — at a time. Rides offer, drivers accept, offers expire, and either side cancels; each of those must move both the ride and the driver together, atomically, under locks taken in a stable order so competing rides cannot deadlock or interleave into a bad state. Only the driver who was offered may accept, and only before the offer expires. When several rides compete for one driver, exactly one may win. Expired or cancelled offers must release the driver once and exactly once, and the event history must stay chronological and free of duplicates. The service layer gets some of this right and races on the rest.

## Requirements

- Allow a driver at most one active OFFERED or ASSIGNED lease.
- Transition ride and driver atomically under stable per-resource locks.
- Let only the offered driver accept, and only before expiry.
- Ensure exactly one competing ride wins a contested driver.
- Release the driver exactly once on expiration or cancellation.
- Keep event history chronological and non-duplicative.

## Edge cases to handle

- Two rides offering the same driver at the same instant
- An acceptance arriving just after the offer expires
- A cancellation and an expiration racing on the same offer
- A driver freed by one release being offered again immediately
- Ordering locks consistently to avoid deadlock across resources

## What interviewers look for

Whether both resources move as one transaction under a deterministic lock order, so no interleaving leaves a driver assigned to two rides or released twice. A full-marks answer picks a stable ordering for the ride and driver locks, makes the winning acceptance a single atomic transition, and treats release as idempotent so an expiry and a cancellation cannot both free the same driver. The invariant is unambiguous: one driver, one active lease, one winner.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/ride-booking-driver-assignment-race-resolution-system
