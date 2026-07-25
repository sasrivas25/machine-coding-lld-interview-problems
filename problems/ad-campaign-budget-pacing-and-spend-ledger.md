# Ad Campaign Budget Pacing and Spend Ledger

Difficulty: Hard. Core topic: concurrency-safe budget accounting.

The spend-ledger round. Daily and lifetime budgets, automatic pausing before an overspend, a daily counter that resets when the campaign day rolls over, and spend events that must debit exactly once no matter how often they replay — all while many requests land at the same instant.

## Scenario

You inherit a partly built ad campaign budget backend. Campaigns carry a daily budget and a total budget; spend events arrive, get recorded against a running ledger, and the campaign pauses itself once a charge would push it over a limit. The models and repositories are in place and the shape reads correctly, but the numbers do not hold up. A campaign occasionally slips past its daily budget under load, the daily counter carries yesterday's spend into a new day, a replayed spend event debits twice, and concurrent requests interleave so two charges both read the same remaining budget and both commit.

## Requirements

- Enforce both the daily budget and the total budget so no accepted spend ever crosses either.
- Pause a campaign automatically the moment a spend would exceed a budget.
- Reset accumulated daily spend when the campaign's day advances.
- Make spend events idempotent: a replayed event returns the prior outcome and never debits again.
- Record spend safely under many simultaneous requests against the same campaign.
- Preserve the existing public method contracts; do not modify the tests.

## Edge cases to handle

- A spend that exactly hits a budget boundary versus one that crosses it
- Two concurrent spends that individually fit but together overflow the budget
- A day rollover arriving between reading and writing the daily counter
- The same spend event id delivered more than once
- A campaign already paused receiving further spend attempts

## What interviewers look for

Whether the budget check and the debit form one atomic decision rather than a read followed by a hopeful write. A full-marks answer isolates per-campaign state under a stable lock, treats the idempotency key as the source of truth for replays, and derives the daily reset from the campaign day rather than trusting a stale counter. The invariant is simple and unforgiving: accepted spend never exceeds budget, and every event lands exactly once.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/ad-campaign-budget-pacing-and-spend-ledger
