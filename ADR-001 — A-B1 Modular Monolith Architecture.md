---
type: synthesis
created: 2026-09-08
updated: 2026-09-08
sources: ["wiki/projects/001-calendar-os/Calquartz — Architecture Decision Brief.md", "wiki/projects/001-calendar-os/architecture/Architecture Recommendation.md", "wiki/projects/001-calendar-os/architecture/Architecture Decision Matrix.md", "wiki/projects/001-calendar-os/architecture/Phase 6 — Independent Adversarial Architecture Review.md", "wiki/projects/001-calendar-os/questions/Architecture Open Questions.md"]
tags: [architecture, adr, decision, calendar-os, calquartz]
confidence: high
provenance: owner-directed — explicit approval given in conversation, 2026-09-08, recorded per the Durable Decision Capture Policy
classification: DECISION
scope: calquartz-specific
status: APPROVED — architecture decided; implementation not started; spikes 1-4 executed 2026-09-08 (Phase 7), none triggered a reopen; all implementation gates (security floor, verification) remain outstanding
---

# ADR-001 — Calquartz v1 Architecture: Modular Monolith (A/B1)

Per this project's own operating contract ([[../CLAUDE.md]] §30, "Decision Records"),
recorded in the required shape: ID, title, status, date, context, problem, options,
selected option, reasoning, rejected alternatives, security implications, scalability
implications, maintenance implications, source evidence, consequences.

## ID

ADR-001

## Title

Calquartz v1 architecture: modular monolith (Alternative A/B1)

## Status

**Approved** — 2026-09-08, by explicit owner decision in conversation. Not yet
implemented. Not a claim that any implementation, security, or correctness gate has been
satisfied (see [[../architecture/Calquartz — Architecture Decision Brief|Architecture
Decision Brief]] §8's 2026-09-08 clarification and §"Implementation gates" below).

## Date

2026-09-08

## Context

Calquartz is a commercial, multi-tenant scheduling SaaS, designed by evaluating and
selectively combining verified characteristics of two reference systems (SnagTime,
Cal.diy) rather than merging either wholesale — see [[../CLAUDE.md]] and
[[../README|the project README]] for the full history (Phases 1–7, 5A, 5B, 6). Four
genuinely distinct architecture alternatives were designed against the project's real
constraints (one 2 CPU/8 GB/100 GB VPS, no further infrastructure purchase, one solo
AI-assisted operator) and evaluated in [[../architecture/Architecture Decision Matrix|the
Decision Matrix]]. A/B1 has been the standing recommendation since Phase 3, re-audited in
Phase 5, and independently, adversarially re-examined in
[[../architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6]] against
three newly-decided product-scope inputs before this approval:

- **Hosted-only** (owner decision, 2026-09-08) — self-hosting is not a v1 requirement.
- **Flat tenancy** (owner decision, 2026-09-08) — no org-of-orgs hierarchy in v1.
- **Recurring bookings required in v1** (owner decision, 2026-09-08) — reverses the prior
  "not in v1" working assumption; the single most consequential changed input, examined
  hardest by the Phase 6 review.
- Payments/subscriptions/billing deferred from v1 (owner decision, 2026-09-06, unchanged).

The current VPS state was independently re-verified read-only on 2026-09-08
([[../engineering/Phase 5B — Factual and Technical Prerequisites|Phase 5B]]): idle, 2 CPU,
7.6 GiB RAM (7.0 GiB available), 99 GB disk (21% used), no drift since the Phase 7
decommission.

## Problem

Which architecture, if any, should Calquartz approve for v1, given the confirmed product
scope and infrastructure constraints?

## Options considered

Full descriptions: [[../architecture/Architecture Alternatives]]. Scored (qualitatively,
not summed): [[../architecture/Architecture Decision Matrix]].

- **A/B1 — Modular Monolith**: one deployable (web + worker processes), one PostgreSQL
  database as sole authoritative datastore, durable Postgres-backed job table, disciplined
  enforced module boundaries with designed extraction seams, a database-level
  tenant-isolation backstop as a named candidate requirement. No Redis. No second
  deployable.
- **B2 — A/B1 + Redis**: identical to A/B1 plus Redis used strictly non-authoritatively
  (cache/queue acceleration).
- **C — Isolated Booking Engine**: a separate OS-level process (and schema) for the
  booking-correctness subsystem specifically, on the same single VPS.
- **D — Event-Sourced Core**: booking state modeled as an append-only event log with
  materialized projections, rather than plain CRUD rows.

## Selected option

**A/B1 — Modular Monolith.**

## Reasoning

All four alternatives clear the Security Floor **architecturally** (structural capability
— not an implementation pass; see "Security implications" below) and place internal
correctness invariants (double-booking, idempotency, valid state transitions) at the same
PostgreSQL layer, so they are not distinguished on correctness. They are distinguished on
cost against the actual constraints: one VPS, one solo operator, no further infrastructure
purchase.

- **A/B1 vs. B2**: no concrete, currently-measured requirement justifies Redis
  ([[../architecture/Redis Justification Analysis|Redis Justification Analysis]]). B2
  scores at or below A/B1 on every matrix row, with no offsetting advantage today.
- **A/B1 vs. C**: C's one real advantage (fault isolation / trust-boundary separation) is
  paired with a cost (roughly doubling what one operator manages) that is unconditional
  and current, for a benefit (independent scaling) unavailable on one VPS. The hosted-only
  decision additionally forecloses C's clearest future trigger — a confirmed self-hosting
  requirement — directly, per [[../architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6]] finding 4.
- **A/B1 vs. D**: D's real advantage (structural auditability, a second independent
  recovery path) is achievable more cheaply inside A/B1 via a dedicated append-only audit
  table, without event sourcing's new runtime failure modes (projection lag,
  event-schema evolution, replay failure). **Recurring bookings — examined specifically
  because it is the newest and most consequential input — does not favor D.** A plain
  series/occurrence CRUD model satisfies the actual product requirement (partial-series
  success as an explicit outcome) at a fraction of D's cost; neither reference repository
  offers an event-sourced recurring-booking precedent, meaning D likely costs *more* for
  this specific feature than the existing 8–14 engineer-day CRUD-shaped estimate, not
  less. This corrects, rather than confirms, the prior recommendation's own "modest pull
  toward D" framing — see Phase 6 §5.

Eight adversarial attack lines were run against A/B1 specifically (Phase 6 §3) before this
approval; none forced a change to any of A/B1's defining properties.

## Rejected alternatives

- **B2 (Redis)**: not rejected in principle — **deferred** to named, concrete, measured
  triggers ([[../architecture/Redis Justification Analysis|Redis Justification
  Analysis]]): sustained Postgres CPU/IO pressure specifically from availability-read or
  job-polling query classes; a measured availability-latency threshold breach; confirmed
  Tier 1 volume; or a specific feature requiring sub-second cross-request coordination a
  database-backed approach cannot provide.
- **C (Isolated Booking Engine)**: not rejected in principle — **deferred** to: a second
  machine actually provisioned with measured need; a public API confirmed with untrusted
  third-party callers; or the hosted-only decision being explicitly reopened and reversed.
- **D (Event-Sourced Core)**: not rejected in principle — **deferred** to: audit/compliance
  requirements becoming materially more central to commercial positioning than currently
  documented; or recurring-series dispute-resolution being confirmed to specifically
  require full historical replay as a product requirement (not merely a nice-to-have).

## Security implications

**Architectural capability, not implementation verification — these are different
claims.** No alternative is structurally incapable of satisfying any of the sixteen
[[../architecture/Security Floor|Security Floor]] items; A/B1 specifically offers no
structural disadvantage relative to the others on any item, and no structural advantage
over C (item 1, schema-level isolation) or D (item 13, auditability) — both of those
advantages are real but were judged not to outweigh their alternatives' costs (above).
**No Calquartz implementation has passed, or could yet have passed, the Security Floor —
no Calquartz code exists.** All sixteen items remain `UNKNOWN` until real code exists and
[[../engineering/Calquartz Security Floor — Operational Review Process|the operational
review process]] actually runs against it. The pre-existing OAuth coverage gap (identified
during the independent-review pass, 2026-09-08) remains an open finding, not resolved or
added as a Floor item by this decision.

## Scalability implications

No alternative scales out today — there is one VPS and none is being purchased. A/B1's
module boundaries are deliberately sized so a specific module could later be extracted
into its own deployable if and when a second machine exists; this is a designed-for future
path, not a current capability. Vertical headroom on the current VPS is large and
independently re-verified (idle, ~92% RAM free, 21% disk used,
[[../engineering/Phase 5B — Factual and Technical Prerequisites|Phase 5B]]) — this says
nothing about behavior under real production load, which cannot be measured before
deployment.

## Maintenance implications

One deployable, one datastore, one migration history, one backup/restore target — the
simplest operational story of the four alternatives, matched deliberately to the
project's one-operator, AI-assisted-development constraint. The accepted trade-off is the
weakest failure isolation of the four (a fault anywhere runs in the same process as
everything else) — accepted deliberately for operational simplicity, not overlooked.
Recurring bookings adds real code to this same process (per
[[../architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6]] finding
1) and a previously-unanticipated scheduled batch-job shape (occurrence materialization,
finding 2) that widens, not changes, the job architecture's required scope.

## Source evidence

[[../Calquartz — Architecture Decision Brief]] · [[../architecture/Architecture Alternatives]] ·
[[../architecture/Architecture Decision Matrix]] · [[../architecture/Architecture Recommendation]] ·
[[../architecture/Redis Justification Analysis]] · [[../architecture/Multi-Tenancy Trust Model]] ·
[[../architecture/Database-Level Tenant Isolation — RLS and Alternatives]] ·
[[../architecture/Failure-Mode Analysis]] · [[../architecture/Threat Model]] ·
[[../architecture/Security Floor]] · [[../architecture/Infrastructure Constraint Analysis]] ·
[[../questions/Architecture Open Questions]] ·
[[../engineering/Phase 5B — Factual and Technical Prerequisites]] ·
[[../architecture/Phase 6 — Independent Adversarial Architecture Review]] ·
[[../reuse/Reuse Audit Summary]] (reuse findings judged not to discriminate between
alternatives — [[../Calquartz — Architecture Decision Brief|Decision Brief]] §7).

## Consequences

### Explicit non-decisions (this ADR does not decide)

Carried forward from [[../Calquartz — Architecture Decision Brief|the Decision Brief]]
§17, unchanged by this approval:

- The tenant-isolation mechanism. "A database-level backstop" is a named requirement; RLS
  is the leading candidate; neither is selected — pending
  [[../Calquartz — Pre-Approval Technical Spikes|spike 1]].
- The double-booking invariant's exact shape (per-minute occupancy rows, a `tstzrange`
  exclusion constraint, an exact-tuple key) — pending spike 2, scope widened for recurring
  bookings.
- The recurring-booking series/occurrence data model's exact design — confirmed as
  required scope within this architecture; not designed by this ADR.
- The API paradigm (REST/tRPC/GraphQL); the date/time library; the ORM; the web framework;
  the deployment topology beyond process count; the authentication scheme; the calendar
  provider; the notification provider.
- Seats/group bookings and round-robin/team assignment in v1 — still open, non-blocking
  owner decisions, unaffected by this ADR.
- The canonical product name (Calquartz / Calendar OS / Hybrid Scheduling SaaS) — still
  unresolved, non-blocking.

### Constraints this architecture is approved subject to

One VPS (2 CPU / 8 GB RAM / 100 GB NVMe), no further infrastructure purchase; one solo,
AI-assisted operator; PostgreSQL must never be publicly exposed; booking correctness and
double-booking prevention are release-blocking; independent branding, no implied
Cal.com/SnagTime affiliation — all per
[[../Calquartz — Architecture Decision Brief|Architecture Decision Brief]] §3, unchanged.

### Reopen triggers — when this decision must be revisited, not routed around

Per the owner's own explicit condition on approval: **if technical-spike or
implementation evidence demonstrates that a defining property of this architecture cannot
be satisfied within the stated constraints, the architecture must be reopened, not
silently worked around.** Specifically, per
[[../architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6]] §8:

1. **No database-level tenant-isolation backstop mechanism** (not merely RLS specifically)
   survives Calquartz's actual connection-pooling configuration.
2. **No candidate booking/occupancy constraint** correctly enforces the double-booking
   invariant under concurrent recurring-series writes at realistic volume.
3. **No DST construction** can be demonstrated correct across a recurring series' full
   transition history.
4. **The durable job claim/lease/fence pattern cannot be extended** to the
   scheduled, cross-tenant occurrence-materialization job shape without double-execution
   or starvation.

Additionally, per [[../architecture/Architecture Recommendation|Architecture
Recommendation]]'s own overturn table (unchanged by this ADR): a second machine actually
provisioned with measured need, a confirmed untrusted-caller public API, or the
hosted-only decision being explicitly reopened → revisit C. A measured Postgres CPU/IO or
latency threshold breach, or confirmed Tier 1 volume → revisit B2. Audit/compliance
becoming commercially central, or recurring-series history genuinely needing full replay
as a confirmed product requirement → revisit D.

**Update (2026-09-08, Phase 7)**: all four spikes named in triggers 1–4 above have since
executed against a disposable local PostgreSQL instance —
[[../Calquartz — Pre-Approval Technical Spikes|Pre-Approval Technical Spikes]] carries
each spike's Result; full evidence in `4. Calquartz Spikes/evidence/`. **None of the four
triggers fired** — every candidate mechanism tested satisfied its required property under
real concurrent load. This architecture remains approved, unchanged. Triggers 1–4 remain
stated above exactly as written, since they still govern any *future* spike or
implementation evidence, not only the ones already run.

### Implementation gates (conditions of implementation and release, not yet satisfied)

Security (16-item [[../architecture/Security Floor|Security Floor]], currently all
`UNKNOWN`), correctness (booking correctness per
[[../engineering/Calquartz Booking Correctness — Operational Review Process|the
operational review process]]), tenant isolation (spike 1), DST/timezone correctness
(spike 3), durable-job correctness (spike 4, recurring-bookings-scope-widened),
licensing/provenance (per [[../Calquartz — Architecture Decision Brief|Decision Brief]]
§9 — blocks verbatim copying, not design reuse), and verification (every change producing
a [[../engineering/Verification Evidence Format|Verification Evidence Record]] per
[[../engineering/Authority, Risk, and Routing Model|Authority, Risk, and Routing Model]]).
**None of these gates is satisfied by this ADR.** This ADR selects the architecture the
gates apply to; it does not assert any gate has been passed, and it is not an assertion
that any eventual implementation is safe or complete.

### What happens next

Per [[../Calquartz — Architecture Decision Brief|Decision Brief]] §18 (checklist item 12):
~~run [[../Calquartz — Pre-Approval Technical Spikes|Pre-Approval spikes 1–4]]~~ (spike 5 —
the VPS inventory — was already done; **spikes 1–4 executed 2026-09-08, Phase 7** — see
that document's per-spike Results). Then implementation planning (item 13) — **not yet
started.** Running the spikes did not authorize, and was not, implementation of Calquartz
itself: no Calquartz repository exists; no application code has been written; ECC remains
uninstalled; continuous learning remains disabled; production infrastructure remains
untouched; the disposable PostgreSQL instance used for spikes 1/2/4 was deleted after
evidence capture.

## Owner's approval statement (recorded verbatim in substance)

Given in conversation, 2026-09-08 (voice input; reconstructed here against the exact
acceptance statement drafted in
[[../architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6]] §10,
which the owner was reading from — noted for provenance, not to substitute Claude's
wording for the owner's decision):

> "I approve A/B1 as Calquartz's v1 architecture... subject to the documented security,
> correctness, [booking correctness,] tenant isolation, DST/timezones, durable jobs,
> licensing/provenance, and, of course, verification gates as well. I understand that this
> approval selects the A/B1 architecture, not an implementation stack, [n]or an assertion
> that the implementation is safe [or] complete. I also understand [that if] implementation
> or technical spike evidence demonstrates that an A/B1 defining property cannot be
> satisfied within the stated [A/B1] constraints, the architecture must be reopened rather
> than silently working around the failure."

## Related

- [[../Calquartz — Architecture Decision Brief]] — decision status updated to reflect
  this ADR
- [[../architecture/Architecture Recommendation]] — the recommendation this ADR approves
- [[../architecture/Phase 6 — Independent Adversarial Architecture Review]] — the
  independent review this ADR's approval follows
- [[../engineering/Durable Decision Capture Policy]] — governs how this decision was
  recorded
- [[../questions/Architecture Open Questions]] — the three product-scope decisions this
  architecture approval was gated behind
