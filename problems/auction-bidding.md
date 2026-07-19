# Auction Bid Placement and Closing System

Difficulty: Medium. Core topic: concurrency, critical sections.

The concurrency problem with a scoreboard — the eBay-style round. Many bidders race to outbid each other, the highest valid bid must win, and at some moment the auction closes, even as more bids are still in flight. It looks like a comparison and an if-statement; that is the trap.

## Scenario

You maintain the bidding service of an auction backend: bidders place bids against open auctions, the current high bid is tracked, and a close operation picks the winner. Single-threaded, everything works. Under concurrent bidders it collapses: two racing bids both get accepted as the new highest, a bid sneaks in after closing and silently becomes the winner, closing twice crowns two different winners, and the recorded bid history disagrees with the winner it produced.

## Requirements

- Validating a bid against the current high and installing it as the new high is one atomic step per auction.
- Auctions are protected individually — no global lock across all auctions.
- Closing is a terminal transition that wins every race: no bid accepted after the close, no accepted bid ignored by it.
- Close is idempotent: a second close returns the same winner rather than recomputing one.
- Bids on closed, unknown, or expired auctions are rejected cleanly.
- The bid history always reconciles with the declared winner.

## Edge cases to handle

- Two bidders both reading a high bid of 100 and both submitting 110
- A bid and the close interleaving
- Close called twice
- A bid equal to the current high
- Bids arriving for an auction that never existed

## What interviewers look for

Two races on one object. The first is bid-versus-bid: "if bid > highest" is not atomic, and two threads can both pass it against the same stale value. The second is bid-versus-close — solving the first and missing the second is the most common way strong candidates lose this round. Closing belongs inside the same critical section as bidding, not in a separate code path.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/auction-bid-placement-and-closing-system-coding-problem
