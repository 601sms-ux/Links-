---
type: synthesis
created: 2026-09-08
updated: 2026-09-08
sources: ["wiki/projects/001-calendar-os/architecture/Failure-Mode Analysis.md", "wiki/projects/001-calendar-os/reuse/Test Reuse and Integration Risks.md", "wiki/projects/001-calendar-os/Calquartz — Pre-Approval Technical Spikes.md"]
tags: [engineering, booking-correctness, phase-5a, calendar-os, calquartz]
confidence: medium
provenance: hand-authored, operationalizes the existing Failure-Mode Analysis
classification: RECOMMENDATION
scope: calquartz-specific
status: DRAFT — not yet exercised against real Calquartz code (none exists); §7 status updated 2026-09-08 by owner decision (recurring bookings confirmed in v1); §1/§2/§5/§6/§8 updated 2026-09-08 with Pre-Approval spike results (evidence against a disposable spike database, not Calquartz production code)
---

# Calquartz Booking Correctness — Operational Review Process

Deliverable 7 of [[Phase 5A — Engineering Capability Foundation]]. Converts
[[../architecture/Failure-Mode Analysis|Failure-Mode Analysis]]'s scenario table into an
executable engineering/review process, organized under the categories the phase brief
names, each mapped to its governing invariant, its required test (citing
[[../reuse/Test Reuse and Integration Risks|Test Reuse and Integration Risks]] where one
is already specified), and its current resolution status.

> **Payments are out of scope for v1.** The two payment-related rows in the source
> Failure-Mode Analysis (abandoned-checkout slot lock, "paid after cancel" race) are
> retained there as future-version evidence and are **not** included below — do not
> reintroduce them into v1 engineering process, per this phase's explicit instruction.

---

## The internal/external distinction, restated because it governs everything below

**Internal invariants** (uniqueness, occupancy, idempotency-key uniqueness, valid state
transitions) are enforced by a database constraint inside Calquartz's own Postgres
instance — a constraint genuinely is the final arbiter here.

**External correctness** (calendar-provider state, webhook delivery, notification
delivery) depends on systems Calquartz doesn't control and needs explicit state machines
and reconciliation — no database constraint inside Calquartz's own instance can make an
external system's state match Calquartz's belief about it. Every category below is
tagged accordingly.

---

## 1. Double booking — INTERNAL

| Field | Detail |
|---|---|
| **Invariant** | At most one confirmed booking may exist for a host+time combination the product defines as conflicting |
| **Enforcement point** | A database-level uniqueness or exclusion constraint |
| **Current resolution status** | **Spike 2 executed, 2026-09-08 — see [[../Calquartz — Pre-Approval Technical Spikes\|Pre-Approval Technical Spikes]].** Empirically, the exact-tuple idempotency key admits overlapping-but-non-identical bookings (disqualified as a standalone mechanism); occupancy rows and the `tstzrange` exclusion constraint both pass under real concurrent load, with the exclusion constraint recommended for its far lower write amplification. **Still not selected — a recommendation, not a decision.** Do not implement any candidate as if it were chosen; the exact mechanism remains an explicit ADR-001 non-decision. |
| **Required test once implemented** | Full-path concurrent-booking test against the real datastore (not a mock or an isolated unique-index test), covering both identical-slot and overlapping-but-non-identical-slot races — per [[../reuse/Test Reuse and Integration Risks|Test Reuse and Integration Risks]] §I.3 and §I.7 (item 4). The Reuse Audit specifically found that SnagTime's own headline concurrency test races the raw occupancy insert, not the full `createBooking()` path — do not repeat that narrower scope. |

## 2. Idempotency — INTERNAL

| Field | Detail |
|---|---|
| **Invariant** | A retried or duplicated request must not create a second booking |
| **Enforcement point** | An idempotency key checked against a unique constraint before the booking is considered new |
| **Current resolution status** | Spike 2 executed (see row 1 above) but did not directly benchmark the idempotency-key mechanism itself — SnagTime's client-supplied `Idempotency-Key` + server-side request-fingerprint comparison remains the stronger of the two evidenced designs (it distinguishes "same request retried" from "different request under a reused key," which Cal.diy's server-derived key cannot) but is **still not selected**, and its specific concurrency behavior was not separately spiked |
| **Required test** | Same key + same payload replays the original result; same key + different payload is rejected — per Test Reuse and Integration Risks §I.7 (item 9) |

## 3. Retries — INTERNAL (application) / EXTERNAL (provider calls)

| Field | Detail |
|---|---|
| **Invariant** | A retried database operation must be safe to retry (itself idempotent); a retried external call must not double-execute its effect |
| **Enforcement point** | A narrow retry predicate for database transaction conflicts specifically (not a blanket "retry on any error"); a durable outbox record for external effects |
| **Current resolution status** | Open — no job/outbox design is selected yet, pending the architecture approval this whole engineering layer sits underneath |
| **Required test** | A retry-predicate test confirming only genuine write-conflict errors trigger a retry, and non-retryable errors surface immediately |

## 4. Transactional atomicity — INTERNAL

| Field | Detail |
|---|---|
| **Invariant** | A booking's full effect (the row, its uniqueness/occupancy record, same-transaction side effects) must be inside one atomic transaction boundary; no partial booking state may be visible to another transaction |
| **Enforcement point** | A single database transaction wrapping booking creation |
| **Current resolution status** | A modeling discipline independent of which architecture alternative is chosen — applies identically under A/B1, B2, C, or D per the Failure-Mode Analysis's own alternative-specific notes, with one exception: if C is ever approved, the transaction boundary must not silently span the service boundary (a booking transaction must not depend on a synchronous cross-service call completing first) |
| **Required test** | A test that forces a mid-transaction failure and confirms no partial row/occupancy state persists |

## 5. Worker failure — INTERNAL

| Field | Detail |
|---|---|
| **Invariant** | A crashed worker's claimed-but-incomplete job must become reclaimable without double-executing an already-completed effect |
| **Enforcement point** | A durable job record with a claim/lease mechanism and idempotent job effects |
| **Current resolution status** | **Spike 4 executed, 2026-09-08 — see [[../Calquartz — Pre-Approval Technical Spikes\|Pre-Approval Technical Spikes]].** The claim/lease/fence protocol (atomic claim, fencing token, lease expiry/reclaim) empirically satisfies all four required failpoints under real concurrent workers, and extends without modification to the newly-required scheduled cross-tenant batch-materialization job shape. SnagTime's protocol remains the stronger of the two evidenced designs and is now spike-confirmed, not merely evidenced by forensic reading — but the exact production implementation is **still not selected**, an explicit ADR-001 non-decision. |
| **Required test** | The four named failpoints from Test Reuse and Integration Risks §I.4: crash after claiming a lease; crash between an accepted external effect and the local commit; a stale effect arriving after its subject moved on; shutdown after claiming (must release **without** consuming retry budget). All four demonstrated empirically in spike 4. |

## 6. Cancellation / rescheduling races — INTERNAL

| Field | Detail |
|---|---|
| **Invariant** | A cancellation racing a fresh booking attempt for the same slot must produce one coherent outcome; a reschedule is subject to the same double-booking invariant as a fresh booking (moving into a slot is not exempt) |
| **Enforcement point** | A version-guarded compare-and-swap on the booking's own state for the cancellation race; the same uniqueness/exclusion mechanism as booking creation, scoped to the new time, inside the same atomic operation that releases the old time, for reschedule |
| **Current resolution status** | The compare-and-swap *pattern* is evidenced and sound (SnagTime demonstrates it working) and is now spike-confirmed: spike 2 (2026-09-08) empirically demonstrated that two different bookings concurrently rescheduled into the same target slot produce exactly one winner, under both viable candidate constraints. Its exact production shape remains an explicit ADR-001 non-decision. |
| **Required test** | Per Test Reuse and Integration Risks §I.7 (item 10): cancellation-during-reschedule race; reschedule racing a fresh booking attempt on the target slot |

## 7. Recurring bookings — CONFIRMED IN V1, ENFORCEMENT MECHANISM NOT YET DESIGNED

**Status changed 2026-09-08 (Phase 5B), by explicit owner decision — see
[[../questions/Architecture Open Questions|Architecture Open Questions]]'s Phase 5B
resolution section for provenance.** This row previously read "UNRESOLVED, OWNER DECISION
PENDING." The scope question is now closed: recurring bookings **are** in v1. Per Phase
5B's own explicit instruction, this closes the scope question only — **no technical
implementation is selected by this update**, and none should be inferred from it.

| Field | Detail |
|---|---|
| **Invariant** | Either every occurrence in a series is validated and committed atomically, or partial success is an explicit, surfaced product outcome — not a silent gap |
| **Enforcement point** | Still architecturally unresolved by design — neither reference repository provides an adoptable implementation (SnagTime has none; Cal.diy's is a documented, non-atomic, first-occurrence-only-validated compromise). Being in v1 scope does not resolve *how*; it only removes the possibility of deferring the question. |
| **Current resolution status** | **In v1 scope (confirmed).** Requires its own design pass, informed by — not copied from — either reference repository, before implementation. This design pass has not happened and is explicitly out of scope for Phase 5B (which closes factual/decision prerequisites, not architecture or implementation). Routes through [[../engineering/Authority, Risk, and Routing Model|Authority, Risk, and Routing Model]] §3.2's "Database schema/data migration" and "Bug fix, booking concurrency" rows once real design work begins, since a recurring-series invariant is both a schema question and a booking-correctness question. |
| **Required test** | Cannot be fully specified until the atomicity/partial-success model is chosen. Once designed: a test asserting the chosen model explicitly (e.g., "all-or-nothing per series" vs. "explicit partial-success with surfaced failure"), not merely that "some occurrences got created." |

## 8. Timezone / DST — INTERNAL (representation) + cross-cutting

| Field | Detail |
|---|---|
| **Invariant** | Slot generation and stored booking time must agree on wall-clock meaning before and after a DST transition; a booking made before a transition must not silently shift by an hour when read back after it |
| **Enforcement point** | All correctness-relevant time comparisons in UTC; wall-clock presentation via explicit, tested conversion at defined boundaries only |
| **Current resolution status** | **Spike 3 executed, 2026-09-08 — hypothesis CONFIRMED.** [[../Calquartz — Pre-Approval Technical Spikes|Spike 3]] empirically confirmed a Luxon `startOf("day").plus({minutes})` construction (the shape SnagTime's own slot generator uses) misplaces slots by exactly ±1 hour across a transition, on both `America/New_York` and `Europe/London`, in both directions — and the drift affects the entire day's schedule, not just the first slot. **New finding**: in a recurring series, only the occurrence landing exactly on the transition date is affected. **Do not adopt a day-cursor-plus-minutes construction for slot generation, regardless of eventual date library** — the defect is in the arithmetic shape, demonstrated independent of which library implements it. Two correct constructions were demonstrated (direct wall-clock build; offset correction). |
| **Required test** | A schedule with a 09:00–17:00 window in `America/New_York` and `Europe/London`, slot generation run across March and November transition days, asserting the first slot's local wall-clock time is unchanged across the transition — per Test Reuse and Integration Risks §I.7 (item 3) |

## 9. External calendar / notification state machines — EXTERNAL

| Field | Detail |
|---|---|
| **Invariant** | The system must not present availability that doesn't account for calendar state it cannot currently verify (fail closed); a delayed or duplicated webhook must not corrupt current state |
| **Enforcement point** | Explicit state machines with defined, one-directional transitions; explicit reconciliation between believed and actual provider state; each handler re-derives current state from the event's own payload plus current stored state rather than assuming arrival order |
| **Current resolution status** | No alternative provides this "for free" — it's required implementation work regardless of which architecture is approved. SnagTime's fail-closed pattern (a 503 rather than a silently-empty calendar, enforced by a negative-regex CI assertion) is the stronger of the two evidenced designs; Cal.diy's default is also fail-closed but has a client-controllable suppression flag on a public endpoint that must **not** be reproduced (per the Reuse Audit's finding in Cal.diy Reuse Candidates §D.3.7/§G.3) |
| **Required test** | Inject a failing provider and assert the response is a distinguishable error, not an empty slot list — per Test Reuse and Integration Risks §I.7 (item 7) |

## 10. Payment provider state — **OUT OF SCOPE FOR v1, NOT OPERATIONALIZED HERE**

Retained in [[../architecture/Failure-Mode Analysis|Failure-Mode Analysis]] as future-version
evidence (including the specific abandoned-checkout permanent-slot-lock do-not-repeat
finding). Not converted into an operational check in this document, per explicit
instruction not to reintroduce payments into v1 engineering process. Revisit this document
when a future version actually scopes payments in.

---

## Summary table — what's resolved vs. open

| Category | Domain | Status |
|---|---|---|
| Double booking | Internal | **Spike 2 executed — exclusion constraint recommended; not yet selected** |
| Idempotency | Internal | Mechanism evidenced (SnagTime pattern), not separately spiked, not yet selected |
| Retries | Internal/External | Open — pending job architecture (spike 4 confirmed the underlying job mechanism; retry-predicate design itself not spiked) |
| Transactional atomicity | Internal | Resolved as a discipline, architecture-independent |
| Worker failure | Internal | **Spike 4 executed — claim/lease/fence protocol confirmed; exact production implementation not yet selected** |
| Cancellation/reschedule races | Internal | Pattern resolved (CAS) and now spike-confirmed (spike 2); exact shape not yet selected |
| Recurring bookings | Product scope + Internal | Confirmed in v1 (2026-09-08); enforcement mechanism spike-tested (spikes 2 & 4 both cover recurring-series scenarios) but not yet selected |
| Timezone/DST | Internal | **Spike 3 executed — hypothesis confirmed; correct construction demonstrated, not yet selected as the shipped implementation** |
| External calendar/notification | External | Pattern resolved (fail-closed + reconciliation); implementation pending |
| Payments | Out of scope | Not operationalized for v1 |

**Update (2026-09-08, Phase 7)**: all four spikes this document's "three of ten
categories" line referenced have now executed — see
[[../Calquartz — Pre-Approval Technical Spikes]] for each spike's Result and
`4. Calquartz Spikes/evidence/` for full evidence records. **None of the categories above
moved from "open" to "resolved" as a result** — every spike produced a recommendation
and implementation constraints, per ADR-001's own distinction between architecture
approval and implementation-gate satisfaction; the exact production mechanism for each
category remains an explicit non-decision until real implementation work selects one.

---

## Related

- [[Phase 5A — Engineering Capability Foundation]]
- [[../architecture/Failure-Mode Analysis]] — the source this operationalizes
- [[../reuse/Test Reuse and Integration Risks]] — the ten named tests this process cites throughout
- [[../Calquartz — Pre-Approval Technical Spikes]] — spikes 2 and 3 gate most of this document
- [[Verification Evidence Format]] · [[Authority, Risk, and Routing Model]]
