# Multi-Tenant API Key and Scope Management System

Difficulty: Hard. Core topic: tenant isolation, scope inheritance.

The access-control-backend round. Tenants that must never see each other's keys, effective scopes that combine role and per-key grants, and a rotation grace window that has to expire cleanly. The scaffolding exists; isolation and lifecycle are leaking.

## Scenario

You are working on a partially implemented multi-tenant API key and scope management backend. It models per-tenant roles and keys, effective scopes derived from a role's scopes plus a key's extra scopes, scope-based authorization for active keys, and key rotation with a grace window followed by revocation. Several visible tests fail because some repository and service logic is incomplete or incorrect — tenant boundaries leak, scope inheritance is computed wrong, rotated keys stay valid past their grace window, or revoked keys still authorize.

Read the models, repositories, services, and visible tests, infer the intended behaviour, and fix the implementation. Do not rewrite from scratch, do not change public method contracts, and do not modify the tests.

## Requirements

- Strict per-tenant isolation of roles and API keys — no cross-tenant reads.
- Effective scopes = role scopes plus the key's extra scopes.
- Authorization succeeds only for active keys carrying the required scope.
- Rotation issues a new key while the old one stays valid through a grace window.
- After the grace window, and on explicit revocation, a key no longer authorizes.
- Existing public contracts and tests remain untouched.

## Edge cases to handle

- A key or role lookup that must not resolve across tenant boundaries
- A key whose extra scopes broaden its role's base scopes
- The old key during, and just after, its rotation grace window
- A revoked key that must fail authorization immediately
- A required scope granted by the role but not the key, or vice versa

## What interviewers look for

Whether tenant isolation is enforced at every lookup rather than assumed, and whether effective scope is computed in one place as the union of role and key grants. Full marks treat key lifecycle as a state machine — active, rotating within grace, revoked — so authorization always reflects the key's current status, and no cross-tenant data ever surfaces.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/multi-tenant-api-key-and-scope-management-system
