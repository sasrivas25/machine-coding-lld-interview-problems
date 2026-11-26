# Parking Garage Billing

Difficulty: Easy. Core topic: business rules, time calculations.

The classic parking-lot LLD round with a twist: the specification is a real tariff board, not the prompt. You build ticketed entry and exit for a multi-storey garage and charge exactly what the published tariff says. Originally set as a Gronex Engineering Cup #1 problem.

## Scenario

Gronex Central Garage is replacing its exit-barrier software. The models, the clock helpers and the tariff loader already exist; the two pieces that actually run the barrier do not. The rates and limits live in `config/tariffs.json`, and `docs/TARIFF_BOARD.md` is the board printed at the entry barrier explaining how they combine. Time is passed in as whole minutes since the start of day 0, so every test is deterministic.

## Requirements

- Implement `Garage`: issue a ticket on entry, close it on exit (with the ticket or as a lost ticket), and reject anything the barrier should refuse using the error codes already defined next to the models.
- Implement `calculate_charge`: work out what the driver pays on the way out.
- Charge exactly according to `config/tariffs.json` and the tariff board: grace period, billing units, hour bands, daily caps, overnight flat rate, discounts and lost tickets.
- Do not edit the tests, the tariff file or the board, and run `./verify.sh`.

## Edge cases to handle

- A stay exactly at the grace limit versus one minute over (billed from entry, grace not deducted)
- Billing units counted from entry time, with every started unit charged in full
- Hour bands priced by unit position, with a vehicle class paying its percentage of the band rate and any rupee fraction dropped per unit
- Daily caps applied per calendar day, by the day each unit starts on
- Overnight stays paying the lower of the flat rate and the computed amount
- Lost tickets: no grace period, and the higher of the fee and the full computed charge

## What interviewers look for

Whether you read the specification you were given instead of the one you assume. A strong answer models tickets and their lifecycle explicitly, keeps tariff rules data-driven rather than hard-coded, and tests the boundaries (period edges, caps, lost tickets) before the happy path.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/parking-garage-billing
