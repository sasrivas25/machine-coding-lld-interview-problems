# Hotel Room Reservation with Expiring Holds System

Difficulty: Hard. Core topic: concurrency, expiring holds.

The race-safe booking round. A hold reserves a room's dates only until it expires; confirmation must atomically prove ownership, liveness, and the absence of any conflicting confirmed reservation — and no interleaving of holds and confirmations may ever double-book the same room interval.

## Scenario

You are given a backend repository for hotel room holds. A hold blocks overlapping dates temporarily; the guest then confirms it into a real reservation. The happy path works, but the service layer is loose exactly where it matters under contention. Confirmation checks ownership and state in ways that leave gaps, expired holds can still be confirmed, and expiry cleanup is blunt enough to endanger confirmed reservations.

Worse, concurrent overlapping requests can slip past each other: two holds on the same interval, or a hold plus a confirmation, can both succeed and produce two confirmed reservations for the same room and dates. Per-room locking exists in outline but the atomic check-and-commit is not actually atomic.

## Requirements

- A hold blocks overlapping dates only while ACTIVE and not expired.
- Confirmation atomically verifies ownership, ACTIVE state, non-expiry, and no conflicting confirmed reservation before committing.
- Expired holds cannot be confirmed.
- Expiry cleanup removes only expired holds and never touches confirmed reservations.
- Concurrent overlapping holds and confirmations never yield two confirmed reservations for the same room interval.
- Interval overlap is computed correctly (half-open dates, adjacent stays allowed).

## Edge cases to handle

- A hold that expires between the check and the confirm.
- Two confirmations racing on the same interval.
- A confirm arriving after cleanup has already reclaimed the hold.
- Adjacent date ranges that touch but do not overlap.
- Confirming a hold owned by a different guest.

## What interviewers look for

Whether the confirm path is a single atomic critical section per room, not a sequence of independent checks with a window between them. A full-marks answer scopes locking to the room interval, re-validates expiry inside the lock, and makes the no-double-book invariant hold under any interleaving — plus a cleanup routine that can never race a live confirmation into deleting real bookings.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/hotel-room-reservation-with-expiring-holds-system