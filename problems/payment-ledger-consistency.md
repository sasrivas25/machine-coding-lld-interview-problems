# Payment Ledger Consistency Under Refunds

Difficulty: Hard. Core topic: transactional atomicity, double-entry ledgers.

The database-engineering round. A payments service writes refunds across four tables without a single transaction boundary, so a rejection that arrives after a partial write leaves the ledger permanently wrong — duplicate refunds, drifting balances, and a double-entry sum that is no longer zero.

## Scenario

You inherit a payments service that captures card payments and refunds them against a double-entry ledger backed by PostgreSQL. `accounts` holds merchant, customer, and settlement accounts with a cached `balance_cents`. `payments` records each capture plus a running refunded total. `refunds` holds one row per successful refund, and `ledger_entries` holds the balanced pairs that every capture or refund must produce.

Three incidents were raised last month. Finance found one payment refunded twice under different reference codes, with the refunded total exceeding the capture — the service returned `REJECTED` the second time, but the ledger entries had already been written and never rolled back. A merchant's cached balance stopped agreeing with the sum of its ledger entries, because a refund failed the payment constraint after the ledger write. And after a batch of refund attempts, the sum of all ledger entries platform-wide was no longer zero.

Single-threaded runs against a fresh database look fine. Refunds that should succeed do, and refunds that should be rejected are rejected. The defect is entirely in what survives a rejection, and in the absence of refund deduplication.

## Requirements

- Make a refund atomic: it either commits fully or leaves no trace in the ledger or payment state.
- Make duplicate refund references impossible through a database constraint, not application checks.
- Keep every `balance_cents` equal to the sum of that account's ledger entries.
- Keep the global sum of all ledger entries at zero.
- Never let refunded amounts exceed the captured amount.
- Return the original refund for a repeated reference instead of creating a second one.
- You may add a new forward migration; do not rewrite the existing one, the seed data, or the tests.

## Edge cases to handle

- Rejection that fires after the ledger pair has already been inserted
- A duplicate reference arriving concurrently from two processes
- Partial refunds that cumulatively reach exactly the captured amount
- The refund result type the tests depend on staying unchanged
- Multiple application processes against one database — an in-process lock is not a fix

## What interviewers look for

Whether you push the invariant into the database instead of guarding it in application code. A full-marks answer wraps the whole refund in one transaction, enforces uniqueness with a constraint so concurrent duplicates fail at commit, and can explain why a mutex in one process is worthless once a second replica is deployed.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/payment-ledger-consistency
