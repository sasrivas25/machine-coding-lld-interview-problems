# Payment Ledger Consistency

Difficulty: Hard. Core topic: double-entry accounting, atomic refunds.

The financial-consistency round. Concurrent refunds can exceed the captured payment, rejected operations leave half a ledger movement behind, and cached account balances drift from the journal.

## Scenario

A payment service records captures and refunds as balanced pairs of ledger entries. It also maintains account balance columns and tracks refund references for retry safety. The starter refund path performs checks and writes in separate steps: two transactions can both observe remaining refundable value, a failure can occur after only one side of the movement is written, and duplicate references race.

Finance requires invariants that hold after every commit, not eventual reconciliation.

## Requirements

- Make each refund fully atomic.
- Keep every movement balanced so the global ledger sum remains zero.
- Keep cached account balances equal to ledger-derived balances.
- Prevent cumulative refunds from exceeding the capture.
- Make duplicate references return the original refund.

## Edge cases to handle

- Concurrent partial refunds against the same payment
- Duplicate references arriving on separate connections
- Failure after one account has been updated
- Rejected refunds leaving no database trace
- Lock ordering across the accounts touched by a movement

## What interviewers look for

Whether you identify the accounting invariants, lock the authoritative payment state, use a unique database constraint for reference identity, and update refund, movement, ledger, and balance rows in one transaction. Catching an error after partial commit is too late.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/payment-ledger-consistency
