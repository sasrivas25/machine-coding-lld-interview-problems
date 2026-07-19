# Wallet Transaction and Refund System

Difficulty: Hard. Core topic: ledger discipline, idempotency.

The signature fintech round — the payment-wallet style LLD question where the domain is money and the bar is exact. Credits, debits, and refunds sound like arithmetic until the invariants arrive: a balance can never go negative, refunds can never exceed what was captured, and every balance must be explainable from a ledger.

## Scenario

You maintain a wallet service: users hold balances, transactions credit and debit them, and refunds reverse debits partially or fully. Every operation lands in a ledger. The happy path works, and everything else does not: balances go negative under valid-looking sequences, replayed requests double-credit and double-debit, refunds exceed the original transaction when issued in parts, failed operations leave orphan ledger entries, and the ledger no longer sums to the balances it supposedly explains.

## Requirements

- Ledger entries are append-only; balances derive from them and always reconcile.
- No overdrafts: funds validated before money moves.
- Credits, debits, and refunds carry idempotency keys; a replayed request returns the original outcome with no double effect.
- Cumulative partial refunds never exceed the original transaction amount.
- A rejected operation leaves balance and ledger completely untouched.
- Money in integer minor units, exact accounting throughout.

## Edge cases to handle

- Three 40% partial refunds against one transaction (must be capped cumulatively)
- The same debit request delivered twice
- A refund against a transaction that was itself never captured
- Interleaved partial refunds racing each other
- Ledger-vs-balance reconciliation after any failure

## What interviewers look for

Anchoring everything on the ledger: operations append entries, balances derive from them, and any cached state reconciles back. Every mutation is validate-then-commit under an idempotency key. The classic bug is validating each partial refund in isolation instead of against the cumulative refunded amount — that is the case that always gets probed.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/wallet-transaction-and-refund-system-coding-problem
