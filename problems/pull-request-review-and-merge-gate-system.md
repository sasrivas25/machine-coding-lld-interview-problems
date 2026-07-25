# Pull Request Review and Merge Gate System

Difficulty: Hard. Core topic: review state machine, merge gates.

The state-machine round. Required approvals, changes-requested blocks, approvals that must go stale when new commits land, and authors who must not approve their own work. Getting the merge gate to reflect the latest per-reviewer state is the exam.

## Scenario

You are working on a partially implemented pull-request review and merge-gate backend. It is built to exercise realistic interview behavior around state machines, approval gates, aggregate counting, and correctness of lifecycle transitions. The intended rules are familiar from any code-review tool: a PR is mergeable only when it clears required approvals, has no outstanding changes-requested review, passes required status checks, and its approvals still reflect the current commits.

Several visible tests fail because parts of the repository and service logic are incomplete or incorrect. The gate miscounts approvals when a reviewer reviews more than once, ignores a changes-requested block, keeps stale approvals alive after new commits are pushed, or lets an author approve their own pull request. Your task is to read the models, repositories, services, and tests, understand the intended behavior, and fix the implementation without rewriting it or touching the tests.

## Requirements

- Count approvals by each reviewer's latest review state, not by raw review volume.
- Block merge while any reviewer's latest state is changes-requested.
- Treat required status checks as merge preconditions.
- Dismiss stale approvals when new commits are pushed.
- Prevent an author from approving their own pull request.
- Preserve existing public method contracts across the fix.

## Edge cases to handle

- A reviewer who approves, then later requests changes (and vice versa)
- New commits pushed after approvals were already recorded
- An author submitting a review on their own PR
- Required checks pending, failed, or passed at merge time
- Exactly meeting versus falling one short of required approvals

## What interviewers look for

Whether the gate is computed from a clean projection of per-reviewer latest state rather than an ever-growing tally, so re-reviews and reversals resolve correctly. A full-marks answer treats a new commit as an event that invalidates prior approvals, enforces self-approval and changes-requested as hard blocks, and keeps status checks as independent preconditions. The tell is a PR that flips out of mergeable the instant a reviewer reverses or a commit lands, then flips back only when the latest states truly satisfy the gate.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/pull-request-review-and-merge-gate-system
