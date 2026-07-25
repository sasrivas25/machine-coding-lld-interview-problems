# Project Task Dependency Manager

Difficulty: Hard. Core topic: dependency graphs, cycle detection.

The task-graph round. Dependencies confined to a project, guarded against self-reference, duplicates, and cycles; state transitions gated on dependency completion; and a ready-task query that respects a defined ordering.

## Scenario

You are given a backend for a project task dependency manager. Tasks depend on other tasks, and those edges form a graph the service must keep valid: a dependency must stay within the same project, a task cannot depend on itself, no dependency may create a cycle, and no edge may be duplicated. A task can only start or complete once every dependency it has is COMPLETED, and COMPLETED and CANCELLED are terminal states. On top of that sits a ready-task query — the OPEN tasks whose dependencies are all done — that must come back in a defined order. The service layer enforces some of this and mishandles the rest.

## Requirements

- Reject dependencies that cross projects or point a task at itself.
- Reject a dependency that would form a cycle, and reject duplicate edges.
- Allow a task to start or complete only when all its dependencies are COMPLETED.
- Treat COMPLETED and CANCELLED as terminal states.
- Compute ready tasks as OPEN tasks whose dependencies are all completed.
- Sort ready tasks by due date, then by task id.

## Edge cases to handle

- A dependency chain that would close into a cycle several edges away
- A task depending on a cancelled versus a completed prerequisite
- An attempted transition out of a terminal state
- Duplicate dependency edges submitted twice
- Ready-task ordering when several tasks share a due date

## What interviewers look for

Whether cycle detection is a real graph traversal rather than a shallow immediate-neighbor check, and whether the transition guards and the ready-task query read the same notion of "all dependencies completed." A strong answer keeps the graph invariants enforced at the edge that mutates them, treats terminal states as truly terminal, and produces a stable, fully specified ordering. The follow-up is always "add another constraint," and a well-factored guard absorbs it cleanly.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/project-task-dependency-manager
