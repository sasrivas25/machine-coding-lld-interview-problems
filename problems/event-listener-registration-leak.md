# Event Listener Registration Leak in a Collaboration Server

Difficulty: Hard. Core topic: listener lifecycle, heap-dump analysis.

The runtime-diagnostics round. Editing sessions subscribe to a shared change bus and nothing ever unsubscribes, so the bus grows with every session the process has ever seen while the active-session dashboard stays flat.

## Scenario

A document collaboration server keeps a shared change bus. Every editing session subscribes when it opens so that edits from one collaborator reach everyone viewing the same document.

The service is restarted every few days and operators have learned to live with it. Between restarts resident memory climbs steadily and change delivery gets slower in a way that tracks process uptime rather than how many people are currently editing. The team's dashboard reports a healthy active-session count, so whatever is growing is invisible from there.

A capture taken from a long-running instance is in `artifacts/`. Your job is to find what the server retains and fix it, without changing the tests or the workload they drive.

## Requirements

- Release every subscription when its session closes.
- Keep retained listener count proportional to active sessions, not sessions ever opened.
- Keep delivery latency flat as the process serves more sessions.
- Keep edit delivery semantics intact — every active viewer still receives edits.
- Do not modify the tests or the driven workload.

## Edge cases to handle

- A session that ends abnormally, without a clean close
- The same session subscribing more than once
- Unsubscribing during delivery, while the bus is iterating listeners
- A closure capturing the session and keeping it reachable
- Delivery to a session that has already closed

## What interviewers look for

Whether you read the heap evidence to find the retaining path instead of guessing at the usual suspects. A full-marks answer gives subscription a symmetric lifecycle tied to session close, makes it safe against abnormal termination, and explains why slowing delivery and growing memory are the same defect seen twice.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/event-listener-registration-leak
