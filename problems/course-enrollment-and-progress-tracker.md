# Course Enrollment and Progress Tracker

Difficulty: Medium. Core topic: progress tracking, one-time side effects.

The learning-platform round. Enrollment eligibility, progress measured against the lessons that actually count, a certificate and notification that must fire exactly once on completion, and a clean per-user view of what someone is enrolled in.

## Scenario

You maintain the enrollment and progress module of a learning platform. Students enroll in courses, complete lessons, and check their progress; when a course is finished the system issues a certificate and sends a notification. The repositories and services exist and mostly behave, but the details are off. Progress is computed against all lessons instead of only the required ones, a student can complete a course and yet be judged incomplete, certificates or notifications fire more than once when completion is re-evaluated, and the per-user enrollment listing returns the wrong set. Several visible tests fail on this behavior.

## Requirements

- Enforce enrollment eligibility before a student is enrolled in a course.
- Compute progress from the required lessons, distinguished from the total lesson count.
- Mark a course complete only when its completion condition is genuinely met.
- Issue the certificate and send the completion notification exactly once.
- Return the correct set of enrollments for a given user.
- Preserve the public method contracts; do not modify the tests.

## Edge cases to handle

- A course with optional lessons that must not count toward required progress
- Re-marking an already-completed lesson
- Completion re-evaluated after a certificate has already been issued
- A student enrolling twice in the same course
- A user with no enrollments versus one enrolled in several

## What interviewers look for

Whether completion is a derived, idempotent fact rather than an event fired blindly each time progress changes. A full-marks answer separates eligibility, progress computation, and completion side effects, guards the certificate and notification behind a one-time transition, and keeps required-versus-total lesson accounting explicit. The giveaway of a weak solution is a second certificate appearing when completion is checked twice.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/course-enrollment-and-progress-tracker
