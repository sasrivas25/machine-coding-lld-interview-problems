# Message Queue Consumer: Idempotency, Retries, DLQ

Difficulty: Hard. Core topic: at-least-once delivery, exactly-once effects.

Every backend that uses Kafka, SQS, or RabbitMQ eventually asks the same question: your queue delivers at least once — what does your consumer do about it? A message can arrive twice, arrive out of order after a retry, or fail processing five times in a row, and the consumer must produce exactly-once effects anyway.

## Scenario

You maintain the consumer side of a messaging system: messages are pulled from a queue, processed into side effects, retried on failure, and parked in a dead-letter queue when they exhaust their attempts. The pipeline exists end to end, and it is broken in every dimension: redelivered messages apply their effects twice, a failing message is retried instantly and forever instead of backing off, messages that should be dead-lettered keep circulating, and a retry that finally succeeds still leaves duplicate side effects behind.

## Requirements

- Redelivery is normal, not an error: processing is idempotent, keyed by message identity, so replays are no-ops.
- Retries are bounded, with exponential backoff — no hot loops.
- A message that exhausts its retry budget moves to the DLQ with its error context; it neither vanishes nor recirculates.
- Ack/nack discipline: a message leaves the queue only when its effects are safely recorded.
- A poison message must not block the healthy stream around it.

## Edge cases to handle

- The same message delivered twice (broker cannot tell "processed but unacked" from "never processed")
- A crash between applying the side effect and recording that it was applied
- A retried message overtaking its successors
- A poison message inside an otherwise healthy stream
- A message succeeding on its final allowed attempt

## What interviewers look for

Starting from message identity, not payload: "I have handled message X" must be recorded atomically with X's side effects, or there is a crash window between them — walking that window is most of the problem. Then making failure boring: budgets, backoff, and the DLQ as a pressure valve. This round separates candidates who have used a queue from candidates who have operated one.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/message-queue-consumer-idempotency-and-retry-system-coding-problem
