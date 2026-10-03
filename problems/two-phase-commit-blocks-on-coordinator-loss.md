# Row Locks Are Never Released After a Coordinator Restart

Difficulty: Hard. Core topic: two-phase commit, cooperative termination.

The distributed-systems round. A coordinator restart during a deploy leaves prepared stores holding locks forever, and the periodic sweep that could resolve them gathers the evidence and then does nothing with it.

## Scenario

Three stores take part in transactions driven by a coordinator. The coordinator asks each store to prepare, each store votes and holds locks on the rows it will change, and the coordinator then tells every store whether to commit or abort.

When the coordinator is restarted during a deploy, some stores are left holding locks that are never released. They sit waiting for an instruction that will never arrive, and every query touching those rows blocks behind them until an operator clears the state by hand. The stores stay up throughout, they can still reach each other, and they know which transactions they are waiting on.

The stores also have a periodic sweep that already notices when it has been waiting too long and already gathers what the other stores think about the same transaction. It just never does anything with either.

A capture of one restart is in `artifacts/`, including the final state of every transaction at every store.

## Requirements

- Release the stranded locks.
- Never reach an outcome that contradicts one already published by the coordinator or reached by another store — guessing is not acceptable.
- Keep normal commit and abort paths working.
- Keep the sweep bounded and safe to run repeatedly.
- Do not modify the tests.

## Edge cases to handle

- One store that already committed while others are still prepared
- One store that already aborted
- All participants prepared and none decided
- A participant unreachable during the sweep
- The coordinator returning mid-resolution with a decision

## What interviewers look for

Whether you implement cooperative termination — ask the peers, adopt a decision if any exists, abort only when it is provably safe — instead of a timeout that unilaterally aborts. A full-marks answer keeps atomicity across participants, uses the evidence the sweep already collects, and can explain why 2PC blocks by design on coordinator loss.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/two-phase-commit-blocks-on-coordinator-loss
