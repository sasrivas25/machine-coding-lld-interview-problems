# Shared Calendar Slot Booking System

Difficulty: Hard. Core topic: multi-resource locking, deadlock prevention.

The meeting scheduler is the multi-resource concurrency problem in calendar clothing: booking a slot for three attendees must check and claim time on three calendars at once — all or nothing — while other bookings race to claim overlapping slots on overlapping sets of people. One of the cleanest ways to test whether a candidate understands deadlock rather than merely recognising the word.

## Scenario

You maintain the booking service of a shared calendar: meetings are requested for a set of attendees and a time range, each attendee's calendar is checked for conflicts, and the meeting is confirmed on all of them or none. Sequentially it books fine. Under concurrent load it breaks both ways: overlapping meetings get confirmed onto the same attendee's calendar, and bookings with intersecting attendee sets deadlock the system solid. Cancellations also fail to free every attendee's slot, and partially failed bookings leave some calendars claimed.

## Requirements

- Conflict detection per calendar is atomic: two overlapping requests serialize, and the second sees the first's claim.
- A meeting lands on every attendee's calendar or on none; a failure partway releases exactly what was claimed.
- Per-attendee locks are acquired in a stable global order, so bookings on intersecting attendee sets cannot deadlock.
- Cancellation frees the slot on every attendee, immediately.
- Unrelated bookings (disjoint attendees) proceed concurrently — no single global lock.

## Edge cases to handle

- Booking (A, B) racing booking (B, A) — the textbook circular wait
- Larger lock cycles across three or more bookings
- A conflict discovered on the last attendee of a multi-attendee booking
- Two meetings that touch but do not overlap
- Concurrent cancel and book on the same attendee

## What interviewers look for

The escalation has two steps. Step one is the double-booking race on a single calendar: an atomic check-and-claim, not a check followed by a claim. Step two is the deadlock: acquiring locks in one globally stable order (sort the attendee set by a stable key first) makes the circular wait structurally impossible — the exact answer interviewers wait for in every multi-resource follow-up. The trade-off discussion versus one global lock is part of the grade.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/shared-calendar-slot-booking-system-coding-problem
