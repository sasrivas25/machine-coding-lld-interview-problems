# Outbox Publishes Phantom and Duplicate Events

Difficulty: Hard. Core topic: transactional outbox, concurrent publishers.

The reliable-events database round. The system publishes events for rolled-back orders, misses events for committed orders, and delivers the same outbox row twice after the publisher is scaled horizontally.

## Scenario

An order service writes an order and an `order.placed` event consumed by inventory, fulfillment, and email systems. The starter path does not commit business state and outbox state atomically. Its publisher also selects pending rows without claiming them, so multiple nodes can deliver the same row before either marks it complete.

`published_events` models the external broker, making phantom, missing, and duplicate delivery observable.

## Requirements

- Commit the order and its outbox record in one transaction.
- Leave no outbox event for a rejected or rolled-back order.
- Let multiple publisher instances process batches concurrently.
- Ensure each outbox row produces one delivered event.
- Mark delivery state consistently while honoring batch limits.

## Edge cases to handle

- Two publishers selecting at the same time
- A duplicate order reference rejected by the write path
- A publisher crash around claim, delivery, or marking
- Fewer pending rows than the batch limit
- Locked rows owned by another publisher

## What interviewers look for

Whether you use the database as the coordination boundary: atomic outbox insertion and a concurrency-safe claiming pattern such as row locks with `SKIP LOCKED`, plus durable uniqueness where appropriate. A global process lock prevents scale-out and does not coordinate other nodes.

---

Practice this in a real repo with a failing test suite → https://gronex.org/transactional-outbox-implementation-coding-problem
