# Inventory Stock Reservation System

Difficulty: Medium. Core topic: inventory accounting, rollback.

The reservation model is what separates real inventory systems from toy ones: stock is not one number but three — on hand, reserved, and available — and an order does not decrement stock, it reserves it, to be committed on dispatch or released on cancel. The backbone of e-commerce and quick-commerce machine coding rounds.

## Scenario

You maintain the inventory service of a commerce backend: orders reserve stock from warehouses, reservations are confirmed on fulfilment or released on cancellation, and availability queries drive what customers can buy. The operations exist and the accounting is broken: available stock ignores reservations so the same units sell twice, an order with two lines for the same SKU reserves double, a multi-line order that fails partway keeps the lines it already grabbed, cancelling returns wrong quantities, and confirming ships stock that was never reserved.

## Requirements

- The three-quantity invariant holds at every moment: on hand = reserved + available, per SKU per warehouse.
- Reserve moves available → reserved; confirm moves reserved out of on hand; release moves reserved → available. No operation may drive a bucket negative.
- Duplicate lines for one SKU in an order merge into one reservation before validation.
- Multi-line reservation is all-or-nothing: a failure on the third line means the first two were never taken, or are precisely undone.
- Availability queries subtract live reservations, not just sales.
- Cancel and confirm are idempotent: repeating a transition has no extra effect.

## Edge cases to handle

- A cart containing the same SKU on two lines
- Partial failure mid-way through a multi-line reservation
- Double cancel (must release once)
- Confirming a reservation that was already released
- Two orders competing for the last available unit

## What interviewers look for

Whether you make the invariant executable first and define every operation as a movement between buckets. The naive decrement-at-order-time design fails on the first follow-up: what happens when payment fails? When the order cancels? When two orders want the last unit? The reservation model answers all three — and normalising order lines per SKU before validating is the small step everyone forgets.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/inventory-stock-reservation-system-coding-problem
