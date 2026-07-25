# API Contract Debugging: Authorization Leak (Broken Access Control)

Difficulty: Medium. Core topic: access control, authorization.

The broken-access-control round. Authentication is the easy half; the half that leaks data is authorization. This handler knows *who* you are and forgets to ask *what you may touch* — so one user reads another's account, and an admin route answers for everyone.

## Scenario

You inherit a small in-process account API modelled as a single `handle(request) -> response` function. There is no web framework: a fake auth layer resolves each caller into an authenticated principal (a `user_id` and a `role`) carried on the request, and each request names a method and path like `GET /accounts/{id}` or `GET /accounts/{id}/transactions`. An admin-only route `GET /admin/accounts` lists every account.

The handler authenticates correctly and then trusts everyone. As shipped it leaks: any authenticated user can fetch another user's account and transactions — a textbook Insecure Direct Object Reference — it returns `200` for principals it should reject, it confuses unauthenticated with authenticated-but-forbidden, and the admin route answers `200` for ordinary users. The bundled contract tests assert every one of these cases and fail on `main`.

## Requirements

- An owner reading their own account or transactions gets `200` with their own data.
- A different authenticated user gets `403`, and no field of the other user's resource appears in the body.
- Missing or invalid credentials get `401` — never `403`, never data.
- The admin route returns `403` for non-admins and `200` for admins.
- A missing resource, requested by an otherwise-allowed caller, returns `404`.
- No response ever leaks a resource the caller is not entitled to see.

## Edge cases to handle

- Unauthenticated request vs authenticated-but-forbidden — distinct status codes
- A valid user requesting an account id that isn't theirs
- A valid user requesting an account id that doesn't exist at all
- Admin-only route reached by a normal, fully authenticated user
- Ordering of the checks: authenticate, then authorize, then resolve existence

## What interviewers look for

Whether authentication and authorization stay separate and are applied in the right order, and whether the `401`/`403`/`404` distinction is deliberate rather than accidental. A full-marks answer proves ownership before touching the resource so no forbidden data is ever read, let alone serialized, and treats "does not exist" and "not allowed" as different questions with different answers.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/api-authorization-leak-debugging
