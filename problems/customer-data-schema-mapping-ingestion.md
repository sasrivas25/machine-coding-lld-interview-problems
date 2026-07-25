# Customer Data Schema Mapping Ingestion

Difficulty: Hard. Core topic: schema mapping, deduplication.

The forward-deployed-engineer round. Real enterprise data arrives from three systems that disagree on field names, and your mapper both silently drops valid values and manufactures duplicate customers when two sources describe the same person. The job is faithful mapping plus honest reconciliation.

## Scenario

You own a customer ingestion adapter used during enterprise rollouts. It takes rows from CRM exports, support systems, and hand-authored onboarding spreadsheets and maps them into one canonical customer shape for downstream provisioning. Customers report two failures: valid email and consent values sometimes vanish from rows that were otherwise accepted, and imports create duplicate customer records for the same person when their data arrives from more than one source system.

The mapper is subtly wrong in two directions at once — it discards fields it should have carried under the configured schema, and it fails to recognise that two source rows are the same customer. Everything runs in memory with deterministic fixtures and no external services.

## Requirements

- Map source rows into the canonical shape honouring the configured schema.
- Preserve valid email and consent values on every accepted row.
- Reject only rows that are genuinely unidentifiable.
- Reconcile repeated customers across source systems into a single record.
- Keep every source identifier when merging; never drop provenance.
- On merge, retain the newer profile data rather than clobbering it with stale fields.

## Edge cases to handle

- The same person arriving from two different source systems
- A row with a valid field the mapper currently drops
- A row that truly has no usable identifier
- Conflicting profile fields where one source is newer than the other
- Multiple source identifiers that must all survive reconciliation

## What interviewers look for

Whether mapping and deduplication are treated as separate, testable steps, and whether "same customer" is decided by an explicit identity rule rather than an accident of field order. A strong answer never loses a source identifier during a merge, resolves conflicts by recency deliberately, and rejects only what it genuinely cannot identify instead of quietly dropping data.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/customer-data-schema-mapping-ingestion-coding-problem
