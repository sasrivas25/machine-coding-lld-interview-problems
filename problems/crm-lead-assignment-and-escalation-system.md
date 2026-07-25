# CRM Lead Assignment and Escalation System

Difficulty: Medium. Core topic: assignment, load balancing.

The routing round. Least-loaded agent selection, load counts that must move exactly once, terminal leads that must never move at all, and time-based escalation that only fires on the right leads. Invariants over counts and state are the exam.

## Scenario

You maintain the service layer of a CRM that assigns leads to agents and escalates the ones that go stale. Only active agents on the lead's team are eligible, and auto-assignment is meant to pick the agent with the least current load, breaking ties by agent id. The shape is right and the bookkeeping drifts: assignments sometimes adjust load more than once, reassignment forgets to release the previous owner, terminal leads get reassigned or escalated when they should be frozen, and escalation fires on leads that are not actually overdue — or misses ones that are.

Every real assignment change is supposed to write exactly one audit event. Your task is to repair the service so the counts, the state transitions, and the audit trail all stay honest.

## Requirements

- Only active agents in the lead's team are eligible for assignment.
- Auto-assignment picks least current load, tie-broken by agent id.
- Each assignment adjusts an agent's load exactly once; reassignment releases the old owner.
- Terminal leads are never reassigned and never escalated.
- Escalation applies only to open leads whose last contact — or creation, if none — is older than the threshold.
- Each real assignment change writes exactly one audit event.

## Edge cases to handle

- Reassigning a lead already owned by another agent
- A team with no eligible active agents
- A lead marked terminal before an escalation sweep runs
- Ties in current load resolved deterministically by id
- An open lead exactly at the escalation threshold boundary

## What interviewers look for

Whether load accounting is a single source of truth that changes by exactly one unit per real transition, and whether "did anything actually change?" gates both the count adjustment and the audit write. A full-marks answer treats terminal state as a hard stop, computes overdue-ness from last-contact-or-creation against a clear threshold, and never double-counts on a no-op reassignment. The tell is a run where nothing eligible changes producing zero load movement and zero audit noise.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/crm-lead-assignment-and-escalation-system
