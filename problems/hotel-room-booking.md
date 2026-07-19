# Hotel Room Booking System

Difficulty: Medium. Core topic: date-range logic.

The Booking.com / Airbnb-style round. It reads like CRUD, but the grading lives in date arithmetic: two stays conflict only when their date ranges truly overlap, and a guest checking out on the 10th must not block a guest checking in on the 10th.

## Scenario

You maintain the backend of a hotel booking service. Creating bookings, cancelling them, and querying room availability all exist — but the date logic is wrong. Overlapping stays for the same room get accepted, back-to-back stays that should be legal get rejected, invalid ranges slip through validation, and cancelling a booking does not reliably free its dates for the next guest.

## Requirements

- Create a booking for a room and a check-in/check-out date range; reject it if it conflicts with an existing active booking for that room.
- Same-day turnover is legal: a check-out on the 10th and a check-in on the 10th can coexist.
- Validate input: check-out must be strictly after check-in, and the room must exist.
- Cancelling a booking releases every night it held — and only those.
- Availability queries must stay consistent as bookings are created and cancelled.
- Listings come back in a stable, deterministic order.

## Edge cases to handle

- Identical date ranges for the same room
- One stay fully contained inside another
- Back-to-back stays sharing a boundary date
- Inverted or zero-length ranges
- Bookings against unknown rooms
- Cancel followed immediately by a rebooking of the same dates

## What interviewers look for

The overlap predicate, done once and put in exactly one place: two half-open intervals [a, b) and [c, d) overlap when a < d and c < b. Treating check-out day as exclusive is what makes same-day turnover work — most failed attempts either double-book a boundary night or reject a valid back-to-back stay. Beyond that: validate-first structure, so a rejected request leaves state completely untouched.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/hotel-room-booking-system-coding-problem
