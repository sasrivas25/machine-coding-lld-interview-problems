# Flash Sale Inventory Purchase System

Difficulty: Hard. Core topic: concurrency, atomic check-and-decrement.

The canonical "prevent overselling" problem: limited stock, a burst of simultaneous buyers, and one non-negotiable invariant — you sell exactly what you have, never unit 101 of 100. The bug is invisible on a single thread, which is exactly why it gets asked.

## Scenario

You maintain the purchase service of a flash-sale backend. Buyers place orders against limited per-item stock; the code checks availability, decrements stock, and records the order. Sequentially it works perfectly. Under concurrent load it oversells: parallel buyers pass the same availability check and stock goes negative, failed purchases leave stock and orders inconsistent, and per-buyer limits are enforced only per-thread.

## Requirements

- The availability check and the stock decrement are one atomic operation per item; two buyers can never both observe the last unit.
- Unrelated items do not contend: per-SKU protection, not one global lock.
- A failed purchase leaves stock and order records exactly as they were.
- Per-buyer purchase limits hold even when one buyer races themselves (double-click, retry).
- The books always balance: sold + remaining = initial stock, to the last unit.

## Edge cases to handle

- Many buyers racing the final unit of stock
- A buyer submitting two concurrent orders that together exceed their limit
- An order recorded without a successful reservation (phantom order)
- A reservation taken but the order write failing (lost stock)

## What interviewers look for

Whether you can name the time-of-check-to-time-of-use race and shrink the critical section to the contended resource — the per-item stock count. One big lock prevents overselling but serialises the whole sale, and interviewers will ask about it; per-SKU protection is the stronger answer. Per-buyer limits need the same atomic treatment as stock.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/flash-sale-inventory-purchase-system-coding-problem
