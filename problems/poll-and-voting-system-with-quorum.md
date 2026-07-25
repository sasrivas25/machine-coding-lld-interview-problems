# Poll and Voting System with Quorum

Difficulty: Medium. Core topic: deduplication, quorum tally.

The one-vote-per-user round. Voters may change their mind while a poll is open but must never count twice, closed and expired polls reject new votes, tallies stay per-option, and validity hinges on a quorum measured by unique participants — not raw ballots.

## Scenario

You are working on a partially implemented poll and voting backend. A poll accepts one vote per voter, allows changing that vote while open, closes on a deadline or by the creator, and reports whether the result is valid given a quorum of participating voters. Models and repositories exist, but several visible tests fail because some service and repository logic is incomplete or wrong.

The symptoms are the usual voting hazards: a revote adds a second ballot instead of replacing the first, votes slip in after the deadline, per-option counts drift, quorum is measured against total ballots rather than distinct voters, and non-creators can close a poll early. Fix the implementation in place without touching the tests.

## Requirements

- One vote per voter per poll; a new vote while open replaces the previous one.
- Votes are rejected once the poll is closed or its deadline has passed.
- Vote counts are tallied per option.
- Result validity is decided by a quorum measured as unique participating voters.
- Only the poll creator may close a poll early.
- Existing public method contracts are preserved.

## Edge cases to handle

- A voter changing their vote several times before close.
- A vote arriving exactly at or just after the deadline.
- Quorum computed from unique voters, not total ballots cast.
- A non-creator attempting an early close.
- A poll that closes with zero votes.

## What interviewers look for

Whether deduplication is a property of the data model rather than a filter applied at read time. A full-marks answer keys ballots by voter so a revote overwrites cleanly, gates writes on both explicit closure and the deadline, and computes quorum over the distinct-voter set — keeping the tally and the validity decision consistent no matter how many times people changed their minds.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/poll-and-voting-system-with-quorum