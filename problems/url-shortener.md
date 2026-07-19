# URL Shortener with Custom Aliases and Quotas

Difficulty: Medium. Core topic: uniqueness, quota accounting.

Probably the most famous backend interview question in existence — "design TinyURL" opens a thousand system design books. This is the half that can actually be graded: a working shortener where aliases must be unique, users have quotas, links expire, and only owners can delete.

## Scenario

You maintain a URL shortener backend: users create short links with generated codes or custom aliases, resolution redirects to the target, quotas cap how many active links a user may hold, and links can expire or be deleted. Every feature exists and every feature fails: duplicate aliases get accepted, quota checks count expired links (or miss deleted ones), resolution happily serves expired short codes, and any user can delete anyone's link.

## Requirements

- Alias claiming is one atomic check-and-reserve; two users racing the same custom alias get exactly one winner.
- Generated codes and custom aliases share one namespace safely.
- Quota counts exactly the active links — expired and deleted links free capacity.
- Resolution respects expiry at read time: a link is dead the moment it expires, not when a cleanup job runs.
- Only the owner can delete a link.
- Identical requests resolve identically, every time.

## Edge cases to handle

- Two concurrent requests claiming the same alias
- A generated code colliding with someone's existing custom alias
- A link expiring between a quota check and a create
- Resolving a link at the exact moment of expiry
- Deletion attempts by non-owners

## What interviewers look for

Whether "active" is a single derived definition (not expired, not deleted) that quota checks, listings, and resolution all share — scatter it across call sites and one path will count a link another path refuses to serve. The alias namespace is the contended resource; each of these requirements is a classic check-then-act trap in a friendly domain.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/url-shortener-with-custom-aliases-and-quotas-coding-problem
