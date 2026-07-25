# Online Exam Attempt and Grading Engine

Difficulty: Medium. Core topic: attempt state machine, idempotency.

The exam-lifecycle round. One active attempt per user, a bounded attempt count, a time limit that expires in-progress attempts, and grading that must run exactly once — the classic places a backend quietly lets a student game the rules.

## Scenario

You inherit a partially implemented online exam attempt and grading backend. It starts, autosaves, and submits an attempt through its lifecycle; enforces one active attempt per user and a bounded number of attempts per exam; expires in-progress attempts once the time limit elapses; and grades each submission exactly once against the correct answers. Several visible tests fail because some repository and service logic is incomplete or incorrect.

The bugs sit on the enforcement boundaries: a second attempt slipping in while one is still active, an autosave that isn't idempotent, an attempt that stays "in progress" past its deadline, or a submission that gets graded more than once and drifts the score.

## Requirements

- Drive an attempt through start → autosave → submit.
- Enforce exactly one active attempt per user at a time.
- Enforce a bounded number of attempts per user per exam.
- Expire in-progress attempts once the time limit elapses.
- Make answer autosave idempotent — a replayed save doesn't corrupt state.
- Grade each submission exactly once against the correct answers.

## Edge cases to handle

- Starting a new attempt while one is already active.
- A save arriving after the time limit has elapsed.
- The same autosave request replayed by the client.
- Submitting an already-expired attempt.
- A second grade request on an already-graded submission.

## What interviewers look for

Whether the attempt is modeled as an explicit state machine with idempotent transitions, not a bag of flags. A full-marks answer pins expiry to the injected clock, makes autosave and grading safe to retry, and enforces the one-active and max-attempts limits as invariants the service guarantees — because every one of these gaps is a way for a student to submit twice, retry after time, or inflate a score.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/online-exam-attempt-and-grading-engine
