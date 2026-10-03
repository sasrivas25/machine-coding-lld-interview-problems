# Machine Coding, LLD, FDE & Production-Incident Interview Problems

A collection of 130 real-world backend engineering interview problems — machine coding & low-level design (LLD), repository-based tasks, distributed-systems failure modes, runtime diagnostics (GC, heap and thread dumps), database engineering, forward-deployed-engineer (FDE) / integration rounds, AI-researcher & data-pipeline bugs, and production-incident debugging. Each is written the way the round actually plays out: you inherit a service that mostly works, the requirements say what correct looks like, and the edge cases are where the grading happens. Statements only — bring your own language and implementation.

Every problem here can also be practiced against a **real repository with a failing test suite**, free, at **[gronex.org](https://gronex.org)** — solve it in the browser workspace or download the starter repo and run the bundled verify script in Java, Python, or C++.

## Repository implementation — LLD & machine coding

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [Food Delivery Order Status Tracker](problems/order-status-tracker.md) | Easy | state machines |
| [CMS Article Publishing and Versioning Workflow](problems/cms-article-publishing-and-versioning-workflow.md) | Medium | versioning, optimistic locking |
| [CRM Lead Assignment and Escalation System](problems/crm-lead-assignment-and-escalation-system.md) | Medium | assignment, load balancing |
| [Course Enrollment and Progress Tracker](problems/course-enrollment-and-progress-tracker.md) | Medium | progress tracking, one-time side effects |
| [Customer Support Ticket SLA System](problems/customer-support-ticket-sla-system.md) | Medium | SLA tracking, injected clock |
| [E-Commerce Coupon Application Engine](problems/coupon-engine.md) | Medium | pricing rules, idempotent redemption |
| [Hotel Room Booking System](problems/hotel-room-booking.md) | Medium | date-range logic |
| [Inventory Stock Reservation System](problems/stock-reservation.md) | Medium | inventory accounting, rollback |
| [Kanban Board with WIP Limits and Card Flow](problems/kanban-board-with-wip-limits-and-card-flow.md) | Medium | ordering, WIP limits |
| [Loyalty Points and Tier Management System](problems/loyalty-points.md) | Medium | FIFO accounting and idempotency |
| [Notification Preferences and Delivery System](problems/notifications.md) | Medium | policy evaluation, idempotent delivery |
| [Online Exam Attempt and Grading Engine](problems/online-exam-attempt-and-grading-engine.md) | Medium | attempt state machine, idempotency |
| [Poll and Voting System with Quorum](problems/poll-and-voting-system-with-quorum.md) | Medium | deduplication, quorum tally |
| [Ride Booking and Driver Matching System](problems/ride-matching.md) | Medium | matching, dispatch lifecycle |
| [Seat Reservation System](problems/seat-reservation.md) | Medium | state machines, cross-entity validation |
| [Tournament Bracket and Match Progression System](problems/tournament-bracket-and-match-progression-system.md) | Medium | bracket tree, match progression |
| [URL Shortener with Custom Aliases and Quotas](problems/url-shortener.md) | Medium | uniqueness, quota accounting |
| [Warranty RMA Return Authorization System](problems/warranty-rma-return-authorization-system.md) | Medium | state machine, transactional consistency |
| [Expense Report Approval and Reimbursement System](problems/expense-report-approval-and-reimbursement-system.md) | Hard | approval routing, budget state |
| [Feature Flag and Rollout Targeting System](problems/feature-flags.md) | Hard | deterministic rule evaluation |
| [Insurance Claim Adjudication System](problems/insurance-claim-adjudication-system.md) | Hard | coverage math, approval authority |
| [IoT Device Provisioning and Firmware Rollout System](problems/iot-device-provisioning-and-firmware-rollout-system.md) | Hard | device lifecycle, staged rollout |
| [Multi-Tenant API Key and Scope Management System](problems/multi-tenant-api-key-and-scope-management-system.md) | Hard | tenant isolation, scope inheritance |
| [Project Task Dependency Manager](problems/project-task-dependency-manager.md) | Hard | dependency graphs, cycle detection |
| [Pull Request Review and Merge Gate System](problems/pull-request-review-and-merge-gate-system.md) | Hard | review state machine, merge gates |
| [Purchase Order Receiving and Three-Way Match System](problems/purchase-order-receiving-and-three-way-match-system.md) | Hard | cumulative receipts, three-way match |
| [Referral Program and Reward Attribution System](problems/referral-program-and-reward-attribution-system.md) | Hard | attribution, idempotency |
| [Subscription Billing and Proration Engine](problems/subscription-billing.md) | Hard | proration math, lifecycle state machines |
| [Wallet Transaction and Refund System](problems/wallet-refunds.md) | Hard | ledger discipline, idempotency |

## Concurrency & race conditions

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [API Rate Limiter](problems/rate-limiter.md) | Medium | time-based accounting, concurrency |
| [Auction Bid Placement and Closing System](problems/auction-bidding.md) | Medium | concurrency, critical sections |
| [Concurrent Bank Transfer](problems/bank-transfer.md) | Medium | deadlock prevention, atomicity |
| [SaaS License Seat Allocation and Reclamation System](problems/saas-license-seat-allocation-and-reclamation-system.md) | Medium | seat capacity, concurrency |
| [API Rate Limiter and Quota Enforcement System](problems/api-rate-limiter-and-quota-enforcement-system.md) | Hard | token-bucket concurrency |
| [Ad Campaign Budget Pacing and Spend Ledger](problems/ad-campaign-budget-pacing-and-spend-ledger.md) | Hard | concurrency-safe budget accounting |
| [Bank Account Transfer and Deadlock Prevention System](problems/bank-account-transfer-and-deadlock-prevention-system.md) | Hard | concurrency, deadlock prevention |
| [Distributed Job Scheduler with Singleton Execution](problems/job-scheduler.md) | Hard | distributed coordination, leases |
| [File Upload Deduplication and Multipart Finalization](problems/file-upload-dedup.md) | Hard | content-addressed storage, reference counting |
| [Flash Sale Inventory Purchase System](problems/flash-sale-inventory.md) | Hard | concurrency, atomic check-and-decrement |
| [Hotel Room Reservation with Expiring Holds System](problems/hotel-room-reservation-with-expiring-holds-system.md) | Hard | concurrency, expiring holds |
| [Marketplace Escrow: Payment Hold and Release](problems/escrow-payments.md) | Hard | money under concurrency, ledger reconciliation |
| [Message Queue Consumer: Idempotency, Retries, DLQ](problems/queue-consumer.md) | Hard | at-least-once delivery, exactly-once effects |
| [Ride Booking Driver Assignment Race Resolution System](problems/ride-booking-driver-assignment-race-resolution-system.md) | Hard | assignment races, ordered locking |
| [Shared Calendar Slot Booking System](problems/calendar-booking.md) | Hard | multi-resource locking, deadlock prevention |

## Distributed systems — delivery, coordination & flow control

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [A Paged Export Is Silently Returning a Third of the Orders](problems/cross-shard-paging-loses-boundary-rows.md) | Hard | cross-shard pagination, merge boundaries |
| [A Read Returned Data the Cluster Had Already Replaced](problems/quorum-read-without-repair.md) | Hard | quorum reads, read repair |
| [A Slow Origin Received Thirty Times the Traffic It Should Have](problems/cache-stampede-on-key-expiry.md) | Hard | cache stampede, request coalescing |
| [A Slow Replica Turns Into a Fleet-Wide Overload](problems/hedged-requests-amplify-load.md) | Hard | hedged requests, load amplification |
| [A Stalled Writer Overwrote the New One](problems/expired-lease-write-after-pause.md) | Hard | leases, fencing, pause tolerance |
| [A Struggling Dependency Received Three Times Its Normal Load](problems/retry-storm-amplifies-outage.md) | Hard | retry budgets, backoff, failure classification |
| [Account Updates Land Out of Order Under Load](problems/ordering-broken-by-parallel-consumers.md) | Hard | per-key ordering, parallel dispatch |
| [Cache Hit Rate Collapses Every Time a Node Is Added](problems/key-remapping-on-cluster-resize.md) | Hard | consistent hashing, membership change cost |
| [Duplicate Charges After a Consumer Restart](problems/at-least-once-duplicate-side-effects.md) | Hard | at-least-once delivery, idempotent side effects |
| [Fulfilment Could Not See the Order That Triggered It](problems/causal-consistency-lost-across-services.md) | Hard | causal consistency, replica read routing |
| [Healthy Nodes Keep Getting Thrown Out of the Fleet](problems/heartbeat-flapping-evicts-healthy-nodes.md) | Hard | failure detection, membership stability |
| [Indexing Latency Grew Without Bound During a Worker Slowdown](problems/missing-backpressure-unbounded-queue.md) | Hard | backpressure, admission control |
| [Jobs Vanish Whenever a Worker Joins or Leaves](problems/rebalance-without-handoff-drops-in-flight-work.md) | Hard | rebalancing, in-flight work handoff |
| [One Bad Record Freezes a Partition](problems/poison-message-blocks-partition.md) | Hard | poison messages, retry budgets, dead-lettering |
| [One Dependency Never Got Cut Off, Another Never Got Restored](problems/circuit-breaker-never-opens.md) | Hard | circuit breakers, state transitions, recovery |
| [One Node's Writes Started Winning Every Conflict](problems/clock-skew-breaks-conflict-resolution.md) | Hard | logical clocks, conflict resolution, replication |
| [One Slow Consumer Is Holding Up Every Other Consumer](problems/slow-subscriber-stalls-broadcast.md) | Hard | fan-out isolation, per-subscriber queues |
| [One Slow Dependency Made Every Dependency Slow](problems/shared-pool-head-of-line-blocking.md) | Hard | bulkheads, resource isolation |
| [Replayed Updates Roll Accounts Back to Old Values](problems/dlq-replay-reorders-live-stream.md) | Hard | replay ordering, state convergence |
| [Report Exports Are Pushing Checkouts Off the Gateway](problems/admission-ignores-request-class.md) | Hard | admission control, flow control by request class |
| [Requests Ran Far Past the Deadline the Caller Was Waiting On](problems/cascading-timeout-budget-exhaustion.md) | Hard | deadline propagation, timeout budgets |
| [Row Locks Are Never Released After a Coordinator Restart](problems/two-phase-commit-blocks-on-coordinator-loss.md) | Hard | two-phase commit, cooperative termination |
| [Search Reported Complete Results While Two Shards Were Refusing Requests](problems/scatter-gather-partial-failure-masked.md) | Hard | partial failure, fan-out completeness |
| [Settlements Vanish After a Consumer Restart](problems/offset-committed-before-processing.md) | Hard | commit ordering, exactly-once effects |
| [Some Ledger Records Never Reach the Ledger](problems/batch-ack-hides-record-failures.md) | Hard | per-record acknowledgement, failure accounting |
| [Stale Inventory After a Delivery Reordering](problems/out-of-order-event-application.md) | Hard | version-aware projections, monotonic state |
| [Tenants Are Allowed Four Times Their Contracted Rate](problems/distributed-rate-limit-drifts-per-node.md) | Hard | distributed rate limiting, shared quota state |
| [Two Edge Nodes Served a Feature Flag That Had Been Changed Ten Minutes Earlier](problems/stale-cache-after-cross-node-invalidation.md) | Hard | cache invalidation, generation checks |
| [Two Nodes Ran the Same Schedule](problems/split-brain-under-network-partition.md) | Hard | leader election, quorum, fencing by term |
| [Two Workers Render the Same Document at Once](problems/visibility-timeout-shorter-than-handler.md) | Hard | visibility timeouts, lease renewal |

## Runtime diagnostics — GC, heap & thread dumps

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [Allocation Rate Explosion in a Telemetry Hot Path](problems/allocation-rate-explosion-in-hot-path.md) | Hard | allocation rate, defensive copying, hot paths |
| [Coarse Lock Convoy in a Shared Registry](problems/coarse-lock-convoy-in-shared-registry.md) | Hard | lock granularity, read concurrency, convoys |
| [Event Listener Registration Leak in a Collaboration Server](problems/event-listener-registration-leak.md) | Hard | listener lifecycle, heap-dump analysis |
| [Finalizer and Cleaner Backlog in a Scratch File Service](problems/finalizer-cleaner-backlog.md) | Hard | deterministic resource release, native handles |
| [Full Result Materialization Heap Spike](problems/full-result-materialization-heap-spike.md) | Hard | streaming, bounded peak memory |
| [Humongous Allocation Region Pressure](problems/humongous-allocation-region-pressure.md) | Hard | allocation shape, large-object thresholds, chunking |
| [Lock Leaked on the Exception Path](problems/lock-leaked-on-exception-path.md) | Hard | critical sections, exception safety, thread dumps |
| [Lock Ordering Deadlock in the Transfer Path](problems/lock-ordering-deadlock-in-transfer-path.md) | Hard | deadlock, lock ordering, concurrency |
| [Missed Signal in a Handoff Queue](problems/missed-signal-lost-wakeup-queue.md) | Hard | condition variables, lost wakeups, shutdown |
| [Off-Heap Buffer Churn and the Full GC Storm](problems/offheap-buffer-churn-full-gc-storm.md) | Hard | off-heap memory, buffer pooling, explicit collection |
| [Premature Promotion and Survivor Overflow in a Batch Rollup](problems/premature-promotion-survivor-overflow.md) | Hard | generational GC, object lifetime, working-set shape |
| [Rule Bundle Reloads Never Release the Previous Generation](problems/plugin-reload-classloader-leak.md) | Hard | hot-reload lifecycle, generation retention |
| [Thread Pool Starvation from Nested Task Submission](problems/thread-pool-starvation-nested-tasks.md) | Hard | pool starvation, task decomposition, blocking waits |
| [Thread-Local Retention in Pooled Workers](problems/thread-local-retention-in-pooled-workers.md) | Hard | thread-local lifecycle, worker pools |
| [Unbounded Cache Heap Exhaustion](problems/unbounded-cache-heap-exhaustion.md) | Hard | retention, cache eviction, heap analysis |

## Database engineering

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [CDC Search Index Synchronization](problems/cdc-search-index-synchronization.md) | Hard | change data capture, transaction visibility, idempotent projections |
| [Catalog Pagination Skips and Repeats Products](problems/large-table-pagination-failure.md) | Hard | keyset pagination, stable cursors |
| [Connection Pool Exhausts After Failed Requests](problems/database-connection-pool-exhaustion.md) | Hard | resource lifecycle, error paths, transaction hygiene |
| [Database Overload Under a Traffic Spike](problems/database-overload-traffic-spike.md) | Hard | indexing, bounded query work, N+1 |
| [Inventory Overselling Under Concurrency](problems/inventory-overselling-under-concurrency.md) | Hard | atomic reservation, idempotency, hold expiry |
| [One Tenant's Volume Slows Every Tenant](problems/hot-partition-multi-tenant-database.md) | Hard | partition pruning, tenant isolation |
| [Outbox Publishes Phantom and Duplicate Events](problems/transactional-outbox-implementation.md) | Hard | transactional outbox, exactly-once delivery |
| [Payment Ledger Consistency Under Refunds](problems/payment-ledger-consistency.md) | Hard | transactional atomicity, double-entry ledgers |
| [Read Replica Serves Stale Data After a Write](problems/read-replica-consistency-failure.md) | Hard | read-after-write consistency, replica routing |
| [Zero-Downtime Column Split Loses Rows](problems/zero-downtime-database-migration.md) | Hard | expand-contract migration, resumable backfill |


## Architecture extension

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [In-Memory Cache: TTL Expiry + LRU Eviction](problems/cache-ttl-and-lru-eviction-extension.md) | Medium | caching, LRU eviction |
| [Job Queue: Delayed Jobs & Priority Ordering](problems/job-queue-delayed-and-priority-extension.md) | Medium | scheduling, priority queue |
| [Rate Limiter: Sliding-Window Algorithm & Per-Tier Limits](problems/rate-limiter-sliding-window-and-tiers-extension.md) | Medium | rate limiting, sliding window |
| [Resumable Multipart Upload](problems/resumable-multipart-upload-extension.md) | Hard | multipart upload, dedup |

## API contract debugging — FDE / integrations

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [API Contract Debugging: Authorization Leak (Broken Access Control)](problems/api-authorization-leak-debugging.md) | Medium | access control, authorization |
| [API Contract Debugging: Cursor Pagination](problems/api-pagination-contract-debugging.md) | Medium | cursor pagination, contract testing |
| [API Contract Debugging: Idempotent POST /orders](problems/api-idempotency-contract-debugging.md) | Medium | idempotency, HTTP contracts |
| [API Contract Debugging: Input Validation & Error Envelope](problems/api-validation-and-error-shape-debugging.md) | Medium | API contract, validation |
| [Model Serving Prediction API Contract](problems/model-serving-prediction-api-contract.md) | Hard | model serving, batch inference |
| [Webhook Event Idempotency and Ordering](problems/webhook-event-idempotency-and-ordering.md) | Hard | idempotency, event ordering |

## Production incident debugging

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [Incident Debugging: Crash Loop from a Bad Config](problems/incident-crash-loop-config-debugging.md) | Easy | config validation, crash loop |
| [Incident Debugging: Checkout Hangs When Recommendations Stall](problems/incident-missing-timeout-hang-debugging.md) | Medium | timeouts, graceful degradation |
| [Incident Debugging: Connection Pool Leak (Service Outage Under Load)](problems/incident-connection-pool-leak-debugging.md) | Medium | resource leak, error handling |
| [Incident Debugging: OOM Crash-Loop on Startup](problems/incident-oom-on-startup-debugging.md) | Medium | memory budget, lazy loading |
| [Incident Debugging: Poison Message Kills the Queue Worker](problems/incident-poison-message-worker-debugging.md) | Medium | poison messages, dead-lettering |
| [Incident Debugging: Retry Storm Without Backoff](problems/incident-retry-storm-debugging.md) | Medium | retry backoff, resilience |
| [API Client Pagination and Retry](problems/api-client-pagination-and-retry.md) | Hard | paginated sync, retry idempotency |
| [Airflow Revenue DAG: Early Publish and Retry Inflation](problems/airflow-dag-dependency-and-idempotent-load.md) | Hard | DAG dependencies, idempotent load |
| [Incident Debugging: Late CDC Corrupts a Customer Snapshot](problems/incremental-cdc-merge-late-arriving-data.md) | Hard | CDC merge, late-arriving data |
| [Incident Debugging: Ledger Transfers Freeze in Production](problems/incident-lock-ordering-deadlock-debugging.md) | Hard | lock ordering, deadlock |

## Debugging & bug fixes — data / ML / backend

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [Connector Record Reconciliation Sync](problems/connector-record-reconciliation-sync.md) | Hard | reconciliation, sync idempotency |
| [Customer Data Schema Mapping Ingestion](problems/customer-data-schema-mapping-ingestion.md) | Hard | schema mapping, deduplication |
| [Debug Batched Classification Metrics](problems/ml-metric-computation-pooling-bug.md) | Hard | classification metrics, batch pooling |
| [Debug Data Leakage in a Retention Feature Pipeline](problems/ml-data-leakage-feature-pipeline.md) | Hard | data leakage, feature engineering |
| [Debug a Logistic Training Loop](problems/ml-training-loop-gradient-bug.md) | Hard | gradients, training loop |
| [Pandas Order Reconciliation: Join Fan-out and Deduplication](problems/pandas-join-fanout-deduplication.md) | Hard | joins, change-data-capture |
| [SQL Window Isolation in Fulfillment Event History](problems/sql-window-function-partition-bug.md) | Hard | window functions, partitioning |

## Linux fundamentals

| Problem | Difficulty | Core topic |
| --- | --- | --- |
| [Access Log Triage & Request Analysis](problems/linux-log-triage-and-request-analysis.md) | Easy | shell log analysis |
| [File Permission & Secret Hardening](problems/linux-file-permission-and-ownership-hardening.md) | Easy | file permissions, hardening |
| [CSV Group-By Aggregation in the Shell](problems/linux-csv-aggregation-pipeline.md) | Medium | shell text processing |
| [Hardening a Fragile Deploy Script](problems/linux-deploy-script-hardening.md) | Medium | shell reliability, error handling |

---

These are original problems in the style of common interview rounds — not questions leaked from any company's process. Solutions, starter repos, and a graded test suite for each live at [gronex.org](https://gronex.org).
