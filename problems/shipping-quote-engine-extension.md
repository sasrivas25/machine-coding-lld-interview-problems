# Shipping Quote Engine Extension

Difficulty: Hard. Core topic: architecture extension, backward compatibility.

The architecture-extension round. A working, tested shipping quote engine prices parcels by weight only; you extend it to zone pricing, dimensional weight, stacked surcharges and carrier rounding as decided in two ADRs, without changing a single byte of existing quotes. Originally set as a Gronex Engineering Cup #1 problem.

## Scenario

Partners diff the engine's output, so v1 quotes must stay byte-identical. The pricing guild's decisions are recorded in `docs/ADR-011-zone-and-dimensional-pricing.md` and `docs/ADR-012-surcharges.md`, and every number you need is in `data/rate_card.json`. The request and quote models already carry the new fields; nothing fills them yet. The ADRs and the rate card are the specification.

## Requirements

- Price routed (v2) quote requests exactly as the ADRs specify: zone and route-class rates, carrier-specific dimensional weight, four surcharges with their precedence and caps, and GST.
- Apply each carrier's settlement rounding through the existing money utility.
- Keep requests without a route (v1) producing byte-identical quotes.
- Expect to touch the rate card repository, the quote service and the quote model.
- Do not edit the tests, the ADRs or the rate card, and run `./verify.sh`.

## Edge cases to handle

- v1 requests (no route) whose rendered quote must stay byte-for-byte identical
- Pincode zones resolved by the first two digits, origin before destination
- Carriers with no rate for a route class
- Dimensional weight beating actual weight, then rounding up to the carrier's weight step
- Four surcharges applied in a fixed precedence under one overall cap, then GST
- Carrier-specific settlement rounding through the existing money utility

## What interviewers look for

Whether you can extend a live system safely. A strong answer reads the ADRs as the contract, isolates the v2 path so v1 is provably untouched, keeps money in exact arithmetic, and adds characterization tests before changing pricing code.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/shipping-quote-engine-extension
