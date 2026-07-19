# Marketplace Escrow: Payment Hold and Release

Difficulty: Hard. Core topic: money under concurrency, ledger reconciliation.

Every marketplace has money in limbo: the buyer paid, the seller has not been paid, and the platform holds the difference until delivery confirms or a dispute resolves. One rule defines escrow: the same held rupee can leave exactly once.

## Scenario

You maintain an escrow service: payments create holds, delivery confirmation releases funds to sellers, disputes refund buyers, and holds past their deadline auto-release. All four paths exist, and money leaks between them: a refund racing an auto-release pays both the buyer and the seller, released holds can be released again, terminal states are re-entered by late operations, and the ledger drifts from the balances it is supposed to explain.

## Requirements

- Hold lifecycle: held → released or refunded, as one-way terminal transitions.
- Release, refund, and the deadline sweeper compete for the same hold; exactly one wins, the rest are clean rejections.
- Auto-release at the deadline cannot double-pay a hold that was just resolved.
- Late or repeated operations against a terminal hold are rejected.
- Ledger reconciliation holds at all times: held + released + refunded = captured, to the exact minor unit.
- A rejected resolution leaves the hold and the ledger untouched.

## Edge cases to handle

- A buyer's refund claim racing the deadline-based auto-release
- Release called twice on the same hold
- The sweeper reading "past deadline, still held" and acting on stale state
- A resolution failing partway (state flipped but no ledger entry, or vice versa)

## What interviewers look for

Resolution as a single compare-and-transition: an operation wins only if it finds the hold still in HELD and flips it to its terminal state in the same atomic step — and the deadline sweeper goes through that same gate as everyone else. The problem compounds a concurrency half and an accounting half; passing one without the other is the standard partial credit.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/marketplace-escrow-payment-hold-and-release-system-coding-problem
