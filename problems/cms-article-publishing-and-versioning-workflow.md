# CMS Article Publishing and Versioning Workflow

Difficulty: Medium. Core topic: versioning, optimistic locking.

The state-machine-with-history round. An article moves from draft to review to scheduled to published to archived, every edit leaves an immutable snapshot behind, two writers race on the same document, and a rollback must restore content without erasing the trail.

## Scenario

You inherit a partially built content management backend for authoring, reviewing, and publishing articles. The models, repositories, and services are all present, and a set of visible tests describes the intended behavior around a guarded lifecycle, immutable version snapshots, optimistic concurrency, time-gated scheduling, and audit history.

The plumbing looks complete but the invariants leak. Some snapshots are mutated in place instead of being captured immutably on each edit, stale-state edits slip through where an optimistic version check should reject them, scheduled publishing does not respect its release time, and rollback overwrites history rather than preserving it. Several visible tests fail until the repository and service logic hold these guarantees.

## Requirements

- Enforce the lifecycle transitions: draft, review, scheduled, published, archived — no illegal jumps.
- Capture an immutable version snapshot on every edit; past versions never change.
- Reject edits made against a stale version via an optimistic version check.
- Publish scheduled articles only once their release time has arrived.
- Support rollback to a prior version while preserving the full history.
- Keep the existing public method contracts; do not modify the tests.

## Edge cases to handle

- Two edits racing on the same expected version
- A rollback followed by further edits and another rollback
- A scheduled publish requested before its time is due
- Archiving from a state that should not permit it
- A snapshot accidentally shared by reference between versions

## What interviewers look for

Whether the lifecycle is a real state machine with a single guarded transition point, whether version snapshots are genuinely immutable rather than aliased references, and whether the optimistic check compares against the version the caller believed it was editing. The strongest answers make rollback a history-preserving append, not a destructive overwrite, so the audit trail always tells the truth.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/cms-article-publishing-and-versioning-workflow