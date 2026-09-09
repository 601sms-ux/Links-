---
type: synthesis
created: 2026-09-08
updated: 2026-09-08
sources: ["wiki/projects/001-calendar-os/architecture/Security Floor.md", "wiki/projects/001-calendar-os/architecture/Threat Model.md", "wiki/projects/001-calendar-os/reuse/Reuse Security and Licensing Findings.md"]
tags: [engineering, security, phase-5a, calendar-os, calquartz]
confidence: medium
provenance: hand-authored, operationalizes the existing 16-item Security Floor + Threat Model
classification: RECOMMENDATION
scope: calquartz-specific
status: DRAFT — not yet exercised against real code (none exists)
---

# Calquartz Security Floor — Operational Review Process

Deliverable 6 of [[Phase 5A — Engineering Capability Foundation]]. Converts the existing
16-item [[../architecture/Security Floor|Security Floor]] (13 original items + 3 added in
Phase 5) from a **stated requirement** into an **executable review procedure** the
`security-reviewer` agent (or a human) actually runs, producing a
[[Verification Evidence Format|Verification Evidence Record]] as its output.

> **This document does not add, remove, or reinterpret any Floor item.** It adds *how to
> check* each one. Where a check genuinely cannot be performed yet (no code exists), that
> is stated as UNKNOWN, not skipped silently.

---

## How to use this document

For a given change, walk every row whose "Applies when" condition is true. Record a
PASS/FAIL/NOT-APPLICABLE/UNKNOWN for each applicable row in a Verification Evidence
Record. **A single FAIL on an applicable row means the change is not ready, regardless of
how everything else scores** — this is the Floor's own stated framing (a gate, not a
weighted score), preserved here rather than softened into a checklist average.

---

## The sixteen items, operationalized

| # | Floor item | Applies when | How to check | Evidence a PASS requires |
|---|---|---|---|---|
| 1 | No cross-tenant data access | Any code path touching tenant-scoped data — including background jobs, admin tooling, migrations | Run the adversarial test category from `tdd-guide`'s checklist: attempt access with an attacker-controlled tenant identifier in the URL, body, session, and any background-job payload the change touches | A specific test exists, names the attack vector, and fails before the fix / passes after it |
| 2 | Fail-closed authorization | Any authorization check | Identify every branch of the check; confirm every branch that cannot positively confirm authorization returns **deny**, including error/exception paths, not just the "normal" false case | Code walkthrough or test covering the ambiguous/error path specifically, not just the happy-deny path |
| 3 | Authenticated resource ownership | Any state-changing action | Confirm the check verifies the caller owns/is-authorized-against *this specific resource ID*, not merely that the caller is logged in | A test that authenticates as user A and attempts to modify a resource owned by user B, expecting denial |
| 4 | Secure public booking flows | The account-less invitee surface (public booking pages, manage links) | Confirm no response leaks another tenant's existence via slug collision, error message, or timing; confirm resource enumeration doesn't succeed against sequential/guessable identifiers | A test attempting to enumerate or infer another tenant's booking/event-type existence via the public surface |
| 5 | Webhook authenticity | Any inbound webhook Calquartz v1 actually has (calendar-provider push, if used — **not** payment webhooks, which are future-version scope) | Confirm signature verification happens against the **raw** request body, before any parsing | A test with a tampered payload and a valid-looking-but-wrong signature, expecting rejection; a code citation showing the raw-body read happens before `JSON.parse` or equivalent |
| 6 | Replay protection | Same webhook scope as item 5 | Confirm a durable, unique record of "this provider event ID has been processed" is written and checked atomically with (or before) the effect | A test delivering the same event ID twice, expecting the second delivery to be recognized as a duplicate, not re-executed |
| 7 | Secret protection | Any code touching a secret/key/credential | Grep the diff for hardcoded credential-shaped strings; confirm all secrets are read from environment/config, never logged, never in debug output | A clean grep result plus a citation of where the secret is read from |
| 8 | Safe logging by construction | Any logging call | Confirm the logging mechanism is allowlist-based (only named fields can be emitted) rather than denylist/redaction-based, per the SnagTime pattern the Reuse Audit found materially stronger than Cal.diy's under-applied redactor | A citation of the logging function's allowlist, or a specific finding if the project instead uses redaction (acceptable, but then confirm the redaction call site actually wraps this specific log statement) |
| 9 | Rate limiting, fails closed | Auth, public booking, and API endpoints | Confirm the limiter's own failure mode (missing config, backing-store timeout, unconfigured) is **deny/throttle**, not allow — the exact defect the Reuse Audit found in Cal.diy's rate limiter (fails open on three independent paths) | A test or code citation showing the limiter's error path returns "limited," not "allowed" |
| 10 | Privileged-operation controls | Any admin/support action that bypasses ordinary tenant scoping | Confirm the bypass is explicit (a named function/role), narrow (scoped to what's needed), and logged | A citation of the specific bypass mechanism and its audit-log call |
| 11 | Worker authorization | Any background worker/job | Confirm the worker's credential is narrower than the web application's — ideally column-level or table-level grants matching only the job classes it actually performs, per the SnagTime pattern the Reuse Audit found genuinely valuable | A citation of the worker's actual granted scope vs. what it writes |
| 12 | Database isolation/backstop | Any tenant-scoped table | Confirm some database-level mechanism exists such that an omitted or defective application check does not by itself produce a cross-tenant breach — mechanism TBD pending [[Calquartz — Pre-Approval Technical Spikes|spike 1]] | UNKNOWN until spike 1 resolves the mechanism; do not assume RLS or any specific mechanism is in place before then |
| 13 | Auditability of security-sensitive actions | Role/permission changes, admin/support actions, authentication events | Confirm the action is recorded in a way that survives the actor's own account being later compromised or deleted (e.g., an append-only audit table, not a mutable "last modified by" field) | A citation of the audit-write call and confirmation it's on a separate, append-only record |
| 14 | Browser-side security headers | Any HTTP response | Confirm CSP, `frame-ancestors`/`X-Frame-Options`, HSTS, and `X-Content-Type-Options` are all set — neither reference repository satisfied all four (SnagTime had none of these beyond `Referrer-Policy`; Cal.diy had a good CSP and none of the other three) | A response-header inspection (curl, browser devtools, or a header-assertion test) showing all four present |
| 15 | Production-configuration validation at boot | The application's startup path | Confirm the process refuses to start in production on missing/placeholder/insufficiently-random secrets, a database URL without verified TLS, or demo/local provider fallbacks active — enforced at every entry point (web start, worker start, readiness probe), not just one | A test that starts the process with a deliberately bad config and confirms it refuses to boot, for each entry point |
| 16 | Dependency/licensing gate | Any dependency addition; any release | Confirm a lockfile is committed and enforced in CI, vulnerability scanning blocks (not just reports), and no verbatim reuse of third-party code whose licence position is unresolved (per [[../reuse/Reuse Security and Licensing Findings|Reuse Security and Licensing Findings]] — nothing from Cal.diy's `apps/api/v2` specifically) | CI configuration citation showing block-on-finding behavior, not report-only |

---

## Explicit non-goals of this document

- It does not select the tenant-isolation mechanism (item 12) — that's spike 1's job.
- It does not add new Floor items beyond the sixteen already established — if a gap is
  found during actual use, that's a finding to bring back to
  [[../architecture/Security Floor|Security Floor]] itself for a Phase-5-style extension,
  not something to quietly patch into this operational document alone.
- It does not cover payment-related items (the Floor's own v1 scope note already excludes
  these) — do not add payment-webhook checks here; payments remain out of scope for v1.

## Current status against real code

**Every row above is UNKNOWN** in the sense that no Calquartz code exists to check yet.
This document is the *procedure*; running it produces the actual evidence, once there is
something to run it against. This is stated explicitly rather than left implicit, per the
instruction not to claim verification that hasn't happened.

---

## Related

- [[Phase 5A — Engineering Capability Foundation]]
- [[../architecture/Security Floor]] · [[../architecture/Threat Model]] — the source this operationalizes
- [[Verification Evidence Format]] — the record format this process's output uses
- [[Authority, Risk, and Routing Model]] — routes HIGH/CRITICAL security-domain tasks here
- [[drafts/agents/security-reviewer|security-reviewer agent]] — the agent that runs this process
