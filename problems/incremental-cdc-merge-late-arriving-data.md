# Incident Debugging: Late CDC Corrupts a Customer Snapshot

Difficulty: Hard. Core topic: CDC merge, late-arriving data.

The green-pipeline-wrong-data round. A customer snapshot fed by incremental CDC is serving contradictory state: an authoritative correction that never landed, and a deleted customer who came back to life after a delayed event arrived. The pipeline is green, so you diagnose from evidence and fix the merge logic.

## Scenario

A customer-profile snapshot built from incremental change-data-capture is contradicting itself. Finance reports an authoritative correction that never appeared in the snapshot; privacy monitoring reports a customer who was deleted reappearing after a delayed event was finally delivered. Nothing failed loudly — the pipeline stayed green — so the incident lives in the audit evidence at `PYTHON/logs/incident.log` and in the state-transition code.

The repository is a deterministic in-memory model of the production merger. Events can be duplicated, delayed, delivered out of order, and stamped by producers whose clocks disagree. The recovered system must land on the source's authoritative per-customer state, stay idempotent under replay, keep customers independent of one another, and refuse to let an older change undo a newer deletion across batch boundaries. You do not edit the tests or the log; `./verify.sh` from `PYTHON/` validates the recovery.

## Requirements

- Converge on the source's authoritative state for every customer.
- Stay idempotent: replaying the same events changes nothing.
- Keep independent customers isolated — one customer's late event never touches another.
- Never let an older change resurrect a customer after a later deletion.
- Reflect authoritative corrections even when they arrive out of order.
- Tolerate duplicated and delayed delivery without corrupting state.

## Edge cases to handle

- A correction stamped earlier by a producer clock but authoritative
- A deletion followed by a delayed pre-deletion update crossing batches
- Duplicate delivery of an already-applied event
- Out-of-order arrival within a single micro-batch
- Two customers whose events interleave across batches

## What interviewers look for

Whether you establish a well-defined ordering that does not trust wall-clock timestamps alone, and apply each per-customer change as a merge that only advances state — so a stale event is a no-op and a deletion is not undone by anything older. A full-marks answer makes replay provably idempotent and reasons about ordering across batch boundaries, not just within one batch. The tell is a deleted customer staying deleted no matter when the straggler arrives.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/incremental-cdc-merge-late-arriving-data-coding-problem
