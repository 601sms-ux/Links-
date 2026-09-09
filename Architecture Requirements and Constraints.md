---
type: synthesis
created: 2026-09-06
updated: 2026-09-08
sources: ["wiki/projects/001-calendar-os/CLAUDE.md", "wiki/projects/001-calendar-os/STORAGE.md", "wiki/projects/001-calendar-os/synthesis/SnagTime vs Cal.diy Synthesis.md"]
tags: [architecture, requirements, calendar-os, calquartz]
confidence: medium
---

# Calquartz — Architecture Requirements and Constraints

> **Phase boundary**: this document establishes what Calquartz needs, separate from what
> either evidence repository happened to build. It does not select an architecture.
> Every statement below is labelled **FACT**, **REQUIREMENT**, **CONSTRAINT**,
> **ASSUMPTION**, **INFERENCE**, **RECOMMENDATION**, or **OPEN QUESTION**. Where the
> project's own documentation doesn't establish something, it is marked **UNKNOWN**
> rather than filled in from general knowledge about scheduling SaaS products.

Source for everything labelled FACT/REQUIREMENT in this document: the project's own
operating contract ([[CLAUDE.md]], written by/for the project owner before any forensic
evidence existed — referred to below as "the charter") and today's phase instructions.
Nothing in [[SnagTime Forensic Analysis]] or [[Cal.diy Forensic Analysis]] is treated as a
Calquartz requirement — those are evidence sources, cited only where explicitly relevant
to an open question.

---

## 1. Product

| # | Statement | Label |
|---|---|---|
| 1 | Calquartz is a commercial, multi-tenant scheduling SaaS product, independently branded, not presented as affiliated with Cal.com or SnagTime. | FACT (charter §1, §8) |
| 2 | Calquartz operates as one hosted multi-tenant service the owner runs commercially — not a self-hostable open-source distribution like both evidence repositories. | **DECISION (owner, 2026-09-08, Phase 5B)**: Calquartz v1 is hosted-only. Customer/tenant self-hosting is explicitly not a v1 requirement, and self-hosting must not add v1 requirements, architecture complexity, support burden, security surface, or testing obligations. The owner's own framing, preserved verbatim: implementation should stay "reasonably portable" with "clean deployment boundaries" in case self-hosting becomes commercially necessary later — a portability preference, not a v1 self-hosting requirement. Supersedes the prior working ASSUMPTION below, which reached the same practical conclusion but was not yet an explicit decision. |
| 3 | Primary users: hosts/organizers who own event types and availability; invitees/bookers, who may hold no account; team/org admins and owners who manage membership and settings. | REQUIREMENT (charter §8-9) |
| 4 | Core workflow: register → verify email → create tenant → configure availability → create event type → connect calendar → publish booking page → book → verify calendar → verify notification → reschedule → cancel. | FACT — this exact flow is named as the required critical E2E path (charter §25) |
| 5 | A user may belong to multiple tenants, with a role (owner/admin/member/viewer) scoped per tenant. | REQUIREMENT (charter §9, §15) |
| 6 | Booking/scheduling model must support event types, availability, and bookings at minimum. | REQUIREMENT (charter §8) |
| 7 | Whether recurring bookings, seats/group bookings, and round-robin/collective team assignment are in scope for an initial release or deferred. | **Recurring bookings: DECISION (owner, 2026-09-08, Phase 5B) — YES, required in v1.** This resolves only the scope question; the technical implementation (atomicity model, series representation) is explicitly not designed by this decision — see [[Calquartz Booking Correctness — Operational Review Process]] §7. **Seats/group bookings and round-robin/collective team assignment remain OPEN QUESTION** — not part of this decision batch; both evidence repositories still differ sharply here (SnagTime: neither; Cal.diy: both, with correctness caveats — see [[SnagTime vs Cal.diy Synthesis]] §4), and both are already classified as resolvable after architecture approval, not blocking it (see [[Architecture Open Questions]]). |
| 8 | Host/team/organization model: whether a flat one-level tenant (SnagTime's shape) or a nested organization-of-organizations model (Cal.diy's shape) is needed. | **DECISION (owner, 2026-09-08, Phase 5B) — flat tenancy.** The charter's own conceptual model (§9) already read as flat, not nested; this decision confirms that reading as the actual v1 requirement rather than leaving it an unconfirmed possible future need. Nested/org-of-orgs is not a v1 requirement; per the evidenced Cal.diy migration-difficulty precedent (§4 below and [[Architecture Open Questions]] question 4), this is a deliberate choice made before data accumulates, not an oversight. |
| 9 | Expected integrations: calendar sync (provider(s) unspecified beyond "calendar integrations"), notifications. | REQUIREMENT (charter §8, §16-17) at the category level; provider/vendor selection is UNKNOWN (see below). **Billing/payment integration is removed from this v1 requirement — see item 11 and §5.** |
| 10 | Which calendar provider(s) are required for an initial release (Google only, per both evidence repos' most-proven path, vs. broader provider coverage per Cal.diy's breadth). | **OPEN QUESTION** — UNKNOWN, no statement in the charter beyond the generic category. |
| 11 | Payment processing, subscription/billing infrastructure, and payment webhooks. | **DECISION (owner, 2026-09-06): explicitly out of scope for Calquartz v1.** Not an open question — resolved by deferral, not by picking a processor. The charter's abstract billing model (tenant → subscription → plan → entitlements → usage, §18) remains valid as a **future-version capability**, not a v1 requirement. See §5 below and [[Architecture Requirements and Constraints#5. V1 scope, deferred scope, and remaining owner decisions|§5]] for the full scope statement, and [[SnagTime vs Cal.diy Synthesis]]/[[SnagTime Forensic Analysis]]/[[Cal.diy Forensic Analysis]] for the historical Phase 2 evidence on payment behavior in the two source repositories, which remains valid forensic record and is **not deleted or altered** by this scope decision. |
| 12 | Plan tiers (Free/Starter/Pro/Business/Team/Enterprise). | **ASSUMPTION**, now explicitly future-version scope given item 11 — charter §18 labels these "Possible plans," illustrative, not fixed, and not applicable to v1 at all. |
| 13 | Public vs. authenticated functionality: public booking pages requiring no account for the invitee; authenticated dashboard for hosts/admins. | REQUIREMENT (charter §8, both evidence repos also converge on this shape — see [[SnagTime vs Cal.diy Synthesis]] §2) |
| 14 | Whether Calquartz needs a public, third-party-facing developer API (comparable to Cal.diy's "Platform API") for an initial release. | **OPEN QUESTION** — charter §8 lists "APIs" and "API keys" in scope generally, without distinguishing an internal-only API from a public developer platform. Architecture-sensitive: this is one of the largest drivers of the API-paradigm decision area. |
| 15 | Whether embeddable booking widgets (comparable to Cal.diy's `embed-core`/`embed-react`) are required. | **OPEN QUESTION** — not mentioned in the charter at all. |
| 16 | Notification channels: email is required (the E2E flow explicitly requires "verify notification," charter §25). SMS is not mentioned. | REQUIREMENT (email) / **OPEN QUESTION** (SMS and any other channel) |
| 17 | Administrative functionality: audit logs, usage limits, tenant branding controls, admin/debug endpoint security review. | REQUIREMENT (charter §8, §24) |
| 18 | Booking correctness and double-booking prevention are explicitly first-class, release-blocking concerns, not ordinary features. | REQUIREMENT, stated directly and emphatically (charter §12-13) |
| 19 | One canonical authentication system; identity, tenant membership, and authorization kept as separate concerns from each other. | REQUIREMENT (charter §14-15) |
| 20 | Tenant-owned resources must have enforceable tenant ownership, audited across API routes, server actions, repositories, services, DB queries, background workers, webhooks, cache keys, files, calendar credentials, API keys, and logs. | REQUIREMENT, stated as a defense-in-depth expectation, not a specific mechanism (charter §10) — the charter explicitly does **not** mandate RLS; it says "PostgreSQL RLS where appropriate," which this phase treats as an open question, not a requirement (consistent with the Phase 2 correction — see [[SnagTime vs Cal.diy Synthesis]] §12). |
| 21 | Independent branding and identity; no implied Cal.com/SnagTime affiliation; white-label support (logo, favicon, colors, booking-page branding, email branding, custom domains) at the tenant level. | REQUIREMENT (charter §8, §19) |

---

## 2. Scale

The charter contains no tenant, user, or booking-volume figures, and none should be
invented. Per phase instructions, the following are **capacity assumption ranges for
evaluation**, not facts, and every architecture alternative is evaluated against the
range, not a single guessed number.

| Tier | Definition (working assumption) | Purpose |
|---|---|---|
| Tier 0 | Fewer than ~100 tenants, each with a handful of members; low tens to low hundreds of bookings/day system-wide | The realistic near-term operating range for a solo-run, newly-launched commercial product |
| Tier 1 | ~100–5,000 tenants; booking volume scaling roughly linearly, still well short of internet-scale | A plausible early-growth range if the product gains traction |
| Tier 2 | 5,000+ tenants | Included only to check that no alternative structurally forecloses this range later — not treated as a near-term design target |

**INFERENCE (domain-structural, not repository-derived)**: booking contention is
naturally sharded by `(host, time slot)` — even at high total tenant counts, the number of
requests racing for one specific host's one specific slot is small in realistic use
(a handful, not thousands). This matters directly for evaluating whether elaborate
distributed-locking or sharding schemes are justified versus premature. The same
per-host/per-tenant sharding logic applies to calendar-provider call volume and background
job volume — none of these are expected to be global bottlenecks at Tier 0 or Tier 1.

| Dimension | Status |
|---|---|
| Tenant/user/booking counts | UNKNOWN — no figures exist; see tiers above |
| Concurrency expectations | ASSUMPTION — naturally bounded per host+slot (see inference above); genuine high-concurrency risk is at the level of "a popular host's slot," not the system as a whole |
| Calendar-provider interaction volume | UNKNOWN exact multiplier; structurally bounded per connected host, not global fan-out |
| Background-job volume | UNKNOWN exact rate; scales with booking + notification + calendar-sync volume, bounded per tenant |
| Storage requirements | UNKNOWN total; individually small per booking row; tenant branding assets (logos) add modest object/blob storage; not expected to be a primary architecture driver at Tier 0–1 |
| Expected growth trajectory | UNKNOWN. REQUIREMENT (charter §21): the system "should be capable of horizontal scaling" and must avoid "premature microservices" — both stated directly, not inferred, and in tension with each other by design: capable of scaling later, without over-building now. |

---

## 3. Operations

| # | Statement | Label |
|---|---|---|
| 1 | Current production infrastructure: one YouStable vProfessional-class VPS — 2 CPU, 8 GB RAM, 100 GB NVMe storage, 8 TB bandwidth, full root access. | **FACT** (owner-supplied, matching the published vProfessional specification). Supersedes the earlier UNKNOWN in this row — see the note below. **This is one single machine. No second server, managed database, managed Redis, load balancer, or Kubernetes is available or being purchased.** |
| 1a | Actual current utilization of that VPS (CPU load, RAM/swap in use, disk usage, what's already running, existing containers/processes, reverse proxy, TLS, backup state, network exposure). | **UNKNOWN** — the published spec tells us the ceiling, not what's already consumed. See [[Infrastructure Constraint Analysis]] for the read-only inventory task this requires; not executed in this pass (no access to the actual machine from this session). |
| 2 | Calquartz must be designed around this VPS, with a credible path to scale later **without purchasing more infrastructure now**. | **CONSTRAINT**, stated directly by the owner: "I cannot invest more in infrastructure at this stage." Explicitly not "stay small forever" — see [[Infrastructure Constraint Analysis]] for how each alternative is evaluated against it. |
| 3 | Deployment must avoid exposing PostgreSQL or Redis (if used) publicly; container-based deployment is implied throughout the charter's infrastructure language and is compatible with a single-VPS deployment. | REQUIREMENT (charter §22, §29) at the "don't expose the database" level; containerization is an ASSUMPTION consistent with both evidence repositories and the charter's tone, not independently mandated |
| 4 | Acceptable operational complexity: the system must be operable by **one person** with AI assistance — not a team, and now known to be operating **one VPS**, not a fleet. | REQUIREMENT, stated directly and repeatedly (charter §1, §3: "minimum human involvement," "solo developer") — one of the heaviest-weighted constraints in the entire evaluation, since it directly penalizes any alternative whose day-to-day operation assumes specialized staff or infrastructure this VPS cannot provide. |
| 5 | Backup and recovery must exist; specific RPO/RTO targets are not stated. | REQUIREMENT (existence) / **ASSUMPTION** (a conservative target — e.g. RPO ≤24h, RTO of a few hours — is reasonable for an early-stage solo-run SaaS on one VPS, but this is an assumption, not a stated target; confirm with the owner before treating it as fixed) |
| 6 | Structured logs, request/job IDs, error tracking, health/readiness endpoints; never log secrets. | REQUIREMENT, stated directly (charter §26) |
| 7 | Expected operator capability: a solo developer, AI-assisted, not a dedicated SRE/ops team, not assumed to have deep specialized infrastructure expertise beyond what such an operator can reasonably run **on one root-access VPS**. | CONSTRAINT (derived directly from #4 above, sharpened by #1) |
| 8 | Budget/cost sensitivity. | **CONSTRAINT**, no longer merely an assumption: the owner has explicitly stated no further infrastructure investment is available at this stage. Exact ceiling for non-infrastructure costs (e.g. third-party SaaS fees) is still UNKNOWN. |

**Note on row 1**: this revises the prior UNKNOWN — the earlier pass searched this
workspace and correctly found no inventory *recorded here*; the owner has now supplied
the specification directly. What remains unknown is *utilization*, not the spec itself —
see [[Infrastructure Constraint Analysis]].

---

## 4. Summary of the open questions this document raises

These feed directly into [[Architecture Open Questions]] alongside the Phase 2 questions
carried forward. Listed here for traceability from the requirement they arose from:
seats/round-robin scope (§1.7 — recurring bookings resolved separately, see below),
calendar-provider scope (§1.10), public developer API (§1.14), embedding (§1.15),
SMS/other notification channels (§1.16), server/infrastructure inventory (§3.1 — resolved,
see [[Infrastructure Constraint Analysis]]'s 2026-09-08 update), backup RPO/RTO targets
(§3.4), and budget ceiling (§3.7). **No longer on this list, all resolved by owner
decision**: payment processor selection (resolved by the v1 payments deferral, §5);
hosted-only vs. self-hosting (§1.2, resolved 2026-09-08 — hosted-only); tenant nesting
depth (§1.8, resolved 2026-09-08 — flat); recurring bookings (§1.7, resolved 2026-09-08 —
yes, in v1).

**None of these being unresolved blocks producing architecture alternatives** — the
alternatives in [[Architecture Alternatives]] are designed to remain coherent across the
plausible answers to each, and each alternative states explicitly where an answer would
change its shape.

---

## 5. V1 scope, deferred scope, and remaining owner decisions

Per an explicit owner decision (2026-09-06): **payments are not part of Calquartz v1.**
This section consolidates that decision with the rest of the product-scope picture into
one authoritative statement, distinguishing three things that must not be conflated:
**historical repository evidence** (Phase 2 forensic findings about how SnagTime and
Cal.diy handle payments — unchanged, not deleted, remains valid provenance for whenever
payments are built), **future Calquartz capability** (payments, described here as a
later-version target), and **current Calquartz v1 scope** (what is actually being
architected now).

**Confirmed v1 (REQUIREMENT)**: authentication; tenancy/workspaces (flat, per the
2026-09-08 decision below); users/members; authorization; event types; availability;
timezone/DST correctness; calendar integration; public booking; booking; **recurring
bookings (owner decision, 2026-09-08 — see §1.7)**; cancellation; rescheduling;
notifications; booking management; security/audit requirements; backup/recovery;
observability.

**Deferred from v1 (DECISION, owner, dates as noted)**: payments; subscriptions/billing;
payment webhooks (all three: explicit owner decision, 2026-09-06, this section); Redis
(deferred per [[Redis Justification Analysis]] — an architectural finding, not an owner
decision, but reaching the same "not v1" status); a separate booking-engine service
(deferred per [[Architecture Recommendation]] — likewise an architectural finding);
**self-hosting (DECISION, owner, 2026-09-08 — hosted-only confirmed for v1; see §1.2)**.

**Resolved by owner decision, 2026-09-08, Phase 5B** (previously listed here as
"still requiring an owner decision before architecture approval" — moved out of this list,
not deleted; see the Phase 5B log entry and [[Architecture Open Questions]] for full
provenance): hosted-only vs. self-hosting (§1.2, resolved **hosted-only**); flat vs.
nested tenancy (§1.8, resolved **flat**); recurring bookings (§1.7, resolved **yes, in
v1**).

**Still requiring an owner decision before architecture approval**: none — all three
items that blocked approval per [[Architecture Open Questions]] are now resolved (above).
**Still open, but not architecture-blocking** (resolvable after approval, before schema
freeze, per [[Architecture Open Questions]]): seats (§1.7); round-robin assignment (§1.7).
A public developer API (§1.14) may remain deferrable specifically because the recommended
architecture maintains one clean API boundary regardless of which alternative is chosen
(see [[Architecture Alternatives]]), so deferring that answer does not risk a costly
redesign the way schema-shaping decisions would.

**On payment's effect on the current architecture recommendation**: removing payments
from v1 scope does not change which alternative is recommended. It reduces the external
correctness surface (fewer state machines and reconciliation loops to build — see
[[Failure-Mode Analysis]]) and reinforces, rather than alters, the case for the
lowest-footprint alternative. Where it would have material effect, that is stated
explicitly in [[Failure-Mode Analysis]], [[Security Floor]], and [[Architecture
Recommendation]] rather than left implicit.

---

## 6. Phase 7/8A implementation constraints (added 2026-09-08, additive — nothing above this line is altered)

Appended by Phase 8A as part of its Phase 7 closure check, which found Phase 7's spike
findings had not yet been cross-referenced into this document. Nothing above this section
is rewritten; this section only adds a pointer and a compact summary so a future reader of
this file's requirement rows does not need to separately discover the spike evidence.

**Implementation constraints established by the four executed technical spikes** (full
detail: [[../Calquartz — Pre-Approval Technical Spikes|Pre-Approval Technical Spikes]],
each spike's "Result (2026-09-08, Phase 7)"; raw evidence: `4. Calquartz Spikes/evidence/`):

- Tenant-isolation backstop must set transaction-local context (`SET LOCAL`) inside the
  same transaction as every tenant-scoped query, enrolled via a build-time,
  schema-derived (fail-closed) mechanism, not a hand-maintained list — spike 1.
- The booking/occupancy invariant must use a database-level construct that correctly
  rejects overlapping-but-non-identical intervals (an exact-tuple key alone is
  insufficient — spike 2 reproduced this defect empirically) and must handle recurring
  series' atomicity as an explicit application-transaction-boundary choice, not a
  constraint-mechanism property.
- Slot/availability construction must use a wall-clock-correct method (not
  `startOf('day').plus({minutes})`) verified across DST transitions, including the
  single-occurrence-on-transition-date case within a recurring series; nonexistent/
  ambiguous local-time handling requires an explicit application-level policy, not a
  library default — spike 3.
- Durable background jobs require a Postgres-backed table with atomic claim, lease,
  fencing token, and `SKIP LOCKED`; the scheduled cross-tenant occurrence-materialization
  batch job requires incremental, not end-of-batch-only, checkpoint commits — spike 4.

**Stack-selection RECOMMENDATION (Phase 8A, not yet owner-approved)**: see
[[../engineering/Phase 8A — Implementation Stack Selection|Phase 8A — Implementation
Stack Selection]] for the full backend/data-access/date-time/job/frontend/auth/testing/
deployment recommendation built against the constraints above. **Not a decision** — this
row exists so a reader of this requirements document knows where the stack-level
follow-on work is recorded, not to assert the stack is chosen.
