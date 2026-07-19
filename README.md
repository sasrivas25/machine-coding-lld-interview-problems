# Machine Coding Problems

A collection of real-world backend machine coding problems for SDE-1 / SDE-2 interview practice. Each one is written the way the round actually plays out: you inherit a service that mostly works, the requirements say what correct looks like, and the edge cases are where the grading happens. Statements only — bring your own language and implementation.

Every problem here can also be practiced against a real repository with a failing test suite, free, at [gronex.org](https://gronex.org).

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [Food Delivery Order Status Tracker](problems/order-status-tracker.md) | Easy | State machines |
| [Seat Reservation System](problems/seat-reservation.md) | Medium | State machines, validation |
| [Hotel Room Booking System](problems/hotel-room-booking.md) | Medium | Date-range logic |
| [API Rate Limiter](problems/rate-limiter.md) | Medium | Time accounting, concurrency |
| [Concurrent Bank Transfer](problems/bank-transfer.md) | Medium | Deadlock prevention |
| [Loyalty Points and Tiers](problems/loyalty-points.md) | Medium | FIFO accounting, idempotency |
| [E-Commerce Coupon Engine](problems/coupon-engine.md) | Medium | Pricing rules |
| [URL Shortener with Aliases and Quotas](problems/url-shortener.md) | Medium | Uniqueness, quota accounting |
| [Auction Bidding and Closing](problems/auction-bidding.md) | Medium | Concurrency, critical sections |
| [Ride Booking / Driver Matching](problems/ride-matching.md) | Medium | Matching, dispatch lifecycle |
| [Notification Preferences and Delivery](problems/notifications.md) | Medium | Policy evaluation, idempotency |
| [Inventory Stock Reservation](problems/stock-reservation.md) | Medium | Inventory accounting, rollback |
| [Flash Sale Inventory](problems/flash-sale-inventory.md) | Hard | Atomic check-and-decrement |
| [Distributed Job Scheduler](problems/job-scheduler.md) | Hard | Leases, exactly-once execution |
| [File Upload Dedup and Multipart Finalization](problems/file-upload-dedup.md) | Hard | Content-addressed storage |
| [Wallet Transactions and Refunds](problems/wallet-refunds.md) | Hard | Ledger discipline, idempotency |
| [Message Queue Consumer](problems/queue-consumer.md) | Hard | At-least-once delivery, DLQ |
| [Marketplace Escrow](problems/escrow-payments.md) | Hard | Money under concurrency |
| [Feature Flags and Rollout Targeting](problems/feature-flags.md) | Hard | Deterministic rule evaluation |
| [Subscription Billing and Proration](problems/subscription-billing.md) | Hard | Proration math, dunning |
| [Shared Calendar Slot Booking](problems/calendar-booking.md) | Hard | Multi-resource locking, deadlock |

These are original problems in the style of common interview rounds (booking systems, wallets, schedulers, limiters) — not questions leaked from any company's process.
