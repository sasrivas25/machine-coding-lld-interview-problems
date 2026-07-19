# E-Commerce Coupon Application Engine

Difficulty: Medium. Core topic: pricing rules, idempotent redemption.

The pricing-rules round. Percentage versus flat discounts, minimum cart values, category restrictions, caps, stacking rules, expiry — and a redemption that must burn exactly once even when the request is retried.

## Scenario

You maintain the coupon service of a checkout backend. Coupons have eligibility conditions, discount logic, and usage limits; carts come in, discounted totals go out. The shape is right and the numbers are wrong: ineligible coupons apply anyway, expired and exhausted coupons still discount, stacking rules are ignored so combinations produce absurd totals, caps and rounding drift by a rupee here and there, and a retried redemption burns a single-use coupon twice.

## Requirements

- Eligibility: minimum cart value, category restrictions, user constraints, expiry — all enforced before any discount applies.
- Discount computation: percentage vs flat, caps, and explicit rounding, correct to the exact minor unit.
- Stacking: only combinable coupons combine, applied in a defined order.
- Redemption is idempotent: a retried apply returns the original result and never double-burns.
- Usage limits enforced exactly, per-coupon and per-user.
- Money in integer minor units; totals always reconcile.

## Edge cases to handle

- Expired or usage-exhausted coupons
- Stacked coupons where application order changes the total
- A percentage discount hitting its cap
- Rounding at intermediate steps vs at the end
- The same redemption request replayed by a client or gateway

## What interviewers look for

Whether the three phases stay separate and honest: eligibility (may this coupon apply at all?), computation (what is the discount, capped and rounded, in a defined order?), and redemption (record the burn, exactly once). Rules encoded as data behind one evaluation point survive the inevitable "now add one more coupon type" follow-up; nested ifs do not.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/e-commerce-coupon-application-engine-coding-problem
