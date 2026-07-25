# Tournament Bracket and Match Progression System

Difficulty: Medium. Core topic: bracket tree, match progression.

The bracket-progression round. Deterministic seeding, winners advancing into the right parent slot, a match that can't be played until both feeders finish, and a completed match that can't be reported twice — a tree state machine that has to stay correct under repeated and out-of-order actions.

## Scenario

You inherit a partially implemented single-elimination tournament bracket backend. It seeds teams deterministically into a bracket, advances winners into the correct parent-match slot, gates a match until both feeder matches are complete, and prevents a completed match from being reported again. The repo is built to exercise realistic interview behaviour around deterministic ordering, parent-child tree progression, and correctness under repeated or out-of-order actions. Several visible tests fail because some repository and service logic is incomplete or incorrect.

The bugs live in the tree mechanics: a seeding order that isn't reproducible, a winner landing in the wrong parent slot, a match that becomes playable before both feeders are done, or a finished match that accepts a second result and corrupts the bracket downstream.

## Requirements

- Seed teams deterministically into the single-elimination bracket.
- Advance each winner into the correct slot of its parent match.
- Allow a match to be played only once both feeder matches are complete.
- Prevent a completed match from being reported again.
- Preserve the existing public method contracts; do not modify the tests.
- Fix the implementation so all visible tests pass without rewriting the app.

## Edge cases to handle

- Reporting a match whose feeders are not both complete yet.
- A winner that must fill the left versus right parent slot.
- Re-reporting a match that is already decided.
- Results reported out of the natural bracket order.
- Reproducing the same seeding given the same team input.

## What interviewers look for

Whether you model the bracket as an explicit parent-child tree with guarded transitions rather than a flat list patched with conditionals. A full-marks answer makes seeding deterministic, computes the parent slot correctly for every match, and treats feeder-completion and single-report as invariants — so no sequence of repeated or out-of-order results can advance the wrong team or double-count a win.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/tournament-bracket-and-match-progression-system
