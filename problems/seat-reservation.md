# Seat Reservation System

Difficulty: Medium. Core topic: state machines, cross-entity validation.

The BookMyShow / ticketing-style booking round — one of the most frequently asked machine coding problems for SDE-1 and SDE-2 backend roles. It looks simple on a whiteboard, but the grading happens in the details.

## Scenario

You maintain the backend of an event seat reservation service. Reserving seats, cancelling reservations, and listing availability all exist — but they are incomplete and partially broken. Requests sneak duplicate seat ids through, seats from the wrong event get reserved, non-available seats get double-booked, and users can cancel reservations they don't own without every seat returning to available.

## Requirements

- Seats move AVAILABLE → RESERVED with no illegal paths; a held seat cannot be reserved again.
- Requests are validated fully before any mutation: seat ids de-duplicated, every seat exists, every seat belongs to the event being booked, all seats AVAILABLE.
- Reserving is one consistent step: a partial failure cannot leave seats half-held.
- Ownership: a user can cancel only their own active reservation.
- Cancellation returns every held seat to AVAILABLE, atomically.
- Listings are deterministically ordered — sorted explicitly, not by map iteration order.

## Edge cases to handle

- Duplicate seat ids inside one request
- Seats belonging to a different event
- Reserving a seat that is already held
- Cancelling someone else's reservation
- Cancelling twice

## What interviewers look for

Correct state transitions under every edge case, and a clean validation-first structure in the service layer: validate the whole request, then transition every seat and create the reservation as one step. Cancellation as the exact mirror image. Deterministic output is part of the spec, not a nicety.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/seat-reservation-system-coding-problem
