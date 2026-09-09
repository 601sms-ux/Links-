---
type: question
created: 2026-09-06
updated: 2026-09-08
sources: ["wiki/projects/001-calendar-os/questions/Open Questions — Calendar OS Forensics.md", "wiki/projects/001-calendar-os/architecture/Architecture Requirements and Constraints.md", "wiki/projects/001-calendar-os/architecture/Database-Level Tenant Isolation — RLS and Alternatives.md"]
tags: [questions, architecture, calendar-os, calquartz]
confidence: high
---

# Architecture Open Questions (Phase 3, revised)

> **Correction, per independent review**: the prior version of this document did not
> distinguish which questions genuinely block architecture approval from which can be
> safely deferred with a stated working assumption. This revision does that explicitly,
> for every question, in four parts: **why it matters**, **whether it can be safely
> deferred**, **what assumption is made if deferred**, and **what future change would be
> needed if it resolves differently**. Architecture approval should not be blocked merely
> because every product detail is unknown — only the "must be resolved" questions below
> gate approval.

## Must likely be resolved before approval

### 1. Hosted-only vs. self-hosting

- **Why it matters**: changes operator-trust assumptions throughout the design. A
  self-hosted product must defend against an unknown, potentially much-less-trusted
  operator (which is why SnagTime's evidenced design invests so heavily in a
  production-boot configuration gate); a hosted-only product only needs to defend the
  owner's own, known deployment.
- **Can it be deferred?** Not comfortably — it changes how much of the [[Security Floor]] and [[Database-Level Tenant Isolation — RLS and Alternatives]] work is "defend
  against us making a mistake" versus "defend against an unknown third party."
- **Working assumption if deferred anyway**: hosted-only (the charter's "commercial SaaS
  operation," tenant branding, and custom-domain language all point this direction, per
  [[Architecture Requirements and Constraints]] §1.2).
- **What would change if this resolves to "self-hosting required"**: the recommendation
  would need revisiting toward a stricter production-configuration-gate design
  (evidenced pattern: SnagTime's boot-time refusal to start on ~20 misconfiguration
  classes) and would weigh Alternative C's trust-boundary separation more favorably.

### 2. v1 recurring-booking scope

- **Why it matters**: directly shapes the booking data model and which failure-mode
  scenarios in [[Failure-Mode Analysis]] are load-bearing on day one.
- **Can it be deferred?** Yes, in the sense that a v1 without recurring bookings is a
  coherent, shippable product — but the schema decisions made without this answer are
  harder to retrofit than to design in from the start if the answer turns out to be
  "yes, needed."
- **Working assumption if deferred**: not in v1 scope, consistent with SnagTime's
  evidenced (simpler) scope rather than Cal.diy's (more complex, and itself
  non-atomic/under-validated per [[SnagTime vs Cal.diy Synthesis]] §4).
- **What would change if resolved as "needed for v1"**: the booking data model needs a
  recurrence concept designed in before implementation begins, and the failure-mode entry
  for "recurring bookings conflict partway through a series" becomes load-bearing rather
  than deferred.

### 3. v1 seats/round-robin scope

- **Why it matters**: seats (multiple attendees per slot) and round-robin/collective team
  assignment are genuinely different data-model and concurrency concerns from a single
  host/single-invitee booking (evidenced concretely in Cal.diy's seat path, the one place
  in either evidence repository using a pessimistic lock rather than a unique constraint,
  per [[SnagTime vs Cal.diy Synthesis]] §4).
- **Can it be deferred?** Yes, more comfortably than recurring bookings — seats can
  plausibly be added later without redesigning the core booking table, provided the
  initial schema doesn't actively preclude it (e.g. doesn't hard-code "one attendee per
  booking" in a way that's expensive to relax).
- **Working assumption if deferred**: not in v1 scope.
- **What would change if resolved as "needed for v1"**: the booking-correctness section
  needs an explicit seat-capacity invariant and, per evidenced practice, likely a
  pessimistic lock (`SELECT ... FOR UPDATE`) specifically for the seat-count check, which
  a pure unique-constraint approach does not handle as directly.

### 4. Flat vs. nested tenant model

- **Why it matters**: changes the tenancy data model directly — a flat model cannot be
  cheaply upgraded to org-of-orgs later without a real migration (evidenced concretely:
  Cal.diy's own migration from user-level to profile-level org identity is, per its own
  schema, still unfinished years in, per [[Cal.diy Forensic Analysis]] §Data Model).
- **Can it be deferred?** Riskier to defer than the two above, specifically because of
  that evidenced migration-difficulty precedent.
- **Working assumption if deferred**: flat (one-level tenant), consistent with the
  charter's own conceptual model (§9), which reads as flat, not nested.
- **What would change if resolved as "nested needed"**: the tenancy schema needs a
  self-referential structure designed in before significant data accumulates, precisely
  to avoid Cal.diy's evidenced unfinished-migration outcome.

## Important but potentially deferrable

### 5. Public developer API for v1

- **Why it matters**: per the corrected Alternative C description, a public API does not
  by itself require C — but if a public API is required *and* is expected to be used by
  less-trusted third parties, that combination strengthens the case for C's trust-boundary
  separation specifically.
- **Deferrable?** Yes — a monolith (A/B1/B2) can add a versioned public API later without
  restructuring the application, since the API paradigm decision was already scoped to
  "one API surface" independent of which alternative is chosen.
- **Assumption if deferred**: no public developer API in v1.
- **What would change if resolved as "yes"**: re-examine whether the API's expected
  trust level (are callers other businesses' backends? untrusted third-party apps?)
  pushes toward C; if callers are trusted/limited, a monolith-hosted public API remains
  sufficient.

### 6. SMS/other notification channels

- **Why it matters**: does not change architecture shape, only which provider
  integrations the notification module needs.
- **Deferrable?** Yes, entirely.
- **Assumption if deferred**: email only for v1.
- **What would change if resolved as "yes"**: add a provider integration to the
  notification module; no architectural change.

### 7. Additional calendar providers beyond an initial choice

- **Why it matters**: affects the calendar-integration module's scope, not the chosen
  alternative's structure.
- **Deferrable?** Yes.
- **Assumption if deferred**: start with one provider (most plausibly Google, the one
  provider both evidence repositories support and the one with the most rigorously
  evidenced correct integration pattern in SnagTime).
- **What would change if resolved as "broader coverage needed sooner"**: adopt a
  provider-abstraction interface earlier (informed by, not copied from, Cal.diy's
  eleven-provider interface shape) rather than hard-coding a single provider's API calls
  throughout.

### 8. Embeds

- **Why it matters**: primarily a frontend/component surface question.
- **Deferrable?** Yes, entirely — low architecture sensitivity.
- **Assumption if deferred**: not in v1.
- **What would change if resolved as "yes"**: add embeddable widget components; no core
  architectural change.

### 9. Exact payment processor — **resolved by deferral, no longer an open question**

Payments are not part of Calquartz v1 at all (owner decision, 2026-09-06 — see
[[Architecture Requirements and Constraints]] §5), so there is no processor to choose for
v1. This question moves to future-version scope: when payments are built, the invariants
in [[Failure-Mode Analysis]] (external state machine, reconciliation, idempotent webhook
processing) remain provider-agnostic, so the processor choice at that time is an
implementation detail, not an architectural one.

## Deferred from v1 (decided, not open questions)

These are no longer questions requiring an answer — they are settled for v1 by explicit
decision or by this phase's architectural findings, listed here for a single place to
check current scope:

- **Payments, subscriptions/billing, payment webhooks** — owner decision, 2026-09-06.
- **Redis** — architectural finding, [[Redis Justification Analysis]]; revisit only on a
  named, measured trigger.
- **A separate booking-engine service** — architectural finding,
  [[Architecture Recommendation]]; revisit only if self-hosting or an untrusted-caller
  public API is confirmed, or a second machine is actually provisioned with measured need.
- **Self-hosting** — working assumption (hosted-only) unless the owner decides otherwise;
  see question 1 above, which remains genuinely open, unlike the three items above.

## Carried forward unchanged from Phase 2 (not architecture-blocking)

Legal/provenance questions (Cal.diy's `apps/api/v2` license contradiction, SnagTime's
chain of title, dependency-license enumeration for both) remain open and are **retained as
a hard constraint / separate workstream**, not folded into architecture scoring — see
[[Open Questions — Calendar OS Forensics]] for the full list and [[Architecture Decision Matrix]] for why licensing was removed as a numeric discriminator. None of these block
architecture approval, since no alternative here is defined as "reuse repository X's
code."

## The one item this document explicitly does not resolve

Whether the database-level tenant-isolation backstop should be RLS specifically requires
a technical spike (connection-pooling compatibility), not an owner decision — see
[[Database-Level Tenant Isolation — RLS and Alternatives]]. This is implementation-adjacent
work for the phase after architecture approval, not an "owner-required" product question.

---

## Phase 5 status update (2026-09-07)

Appended by the Phase 5 decision-review preparation. **Nothing above this line is deleted
or rewritten** — Phase 3's original framing is the historical record and remains readable
as written. This section records only where Phase 4 evidence has changed a question's
*status*, and why.

### Reclassified: questions 3 (seats) and 5-equivalent (round-robin) no longer block approval

**Phase 3 position**: questions 2, 3 and 4 were all listed under "must likely be resolved
before approval", with seats and round-robin bundled into question 3.

**What changed**: Phase 4 audited both subsystems against both repositories and found them
**BUILD FRESH in each** — SnagTime implements neither; Cal.diy's seat path is 295 lines
with 15 imports (only a ~40-line locked-count core transfers) and its round-robin selector
is 889 lines with 13 imports and a documented FIXME that it blocks the wrong users for team
events. Phase 4 also priced them identically across all four architecture alternatives.

**Consequence**: neither question can discriminate between A/B1, B2, C and D, because no
alternative handles either differently and no reuse decision hinges on either. They remain
genuinely open and genuinely important — but as **schema decisions to resolve immediately
after approval and before schema freeze**, not as architecture blockers. Holding the
architecture decision hostage to them inflates a data-model question into an approval gate,
which [[Calquartz — Architecture Decision Brief]] §13 declines to do.

**Unchanged**: the substance of Phase 3's answers above. Seats still need a counter-shaped
invariant and a pessimistic lock (Cal.diy's pattern and its race test are the reference);
the initial schema still must not hard-code one attendee per booking; round-robin still
needs its own design pass. Only the *timing* of the decision has moved.

### Confirmed still blocking: questions 1, 2 and 4

**Question 1 (hosted-only vs. self-hosting)** — unchanged and still blocking. Phase 5 adds
one consequence: this question now also determines the weight of **security floor item 15**
(boot-time production-configuration validation, added in Phase 5). Under hosted-only it
protects the owner from their own mistake; under self-hosting it defends against an unknown
operator and becomes close to essential.

**Question 2 (recurring bookings)** — unchanged and still blocking, and Phase 4 sharpened
the cost. Neither repository provides an adoptable implementation: SnagTime has none at all,
and Cal.diy's is a serial, non-transactional loop validating only the first occurrence.
Phase 4 priced it at **8–14 engineer-days fresh in any alternative**, with **no reuse
offsetting it**. It is also the one product question with any pull on the alternative
choice — D's event log is the most natural home for "partial success of a series is an
explicit, surfaced outcome" — though that pull is modest and does not by itself move the
recommendation.

**Question 4 (flat vs. nested tenancy)** — unchanged and still blocking. Phase 4 reinforced
the evidence behind it: Cal.diy's own migration from user-level to profile-level
organisation identity is still unfinished, with redundant `@@unique` constraints on both
models and a deprecation comment years old. That remains the strongest available argument
for deciding nesting depth before data accumulates rather than after.

### Unchanged: the deferrable list

Questions 5–8 (public developer API, SMS and other notification channels, additional
calendar providers, embeds) are unaffected by Phase 4 and remain safely deferrable on the
reasoning already recorded above. Question 9 (payment processor) remains closed by the v1
payment deferral and is **not reopened**.

### Unchanged: the tenant-isolation mechanism is still not an owner question

The closing section above states that whether the backstop should be RLS specifically
requires a technical spike rather than an owner decision. That stands, and Phase 5 has now
specified the spike in full — see [[Calquartz — Pre-Approval Technical Spikes]] spike 1.
Phase 4 added one requirement to it that was not in Phase 3's framing: **the mechanism must
fail closed when a new table is added**, because SnagTime's implementation fails open on
exactly that path (a hand-maintained 27-name enrolment list). Phase 4 also established that
SnagTime's backstop does not cover its worker role at all, so the worker's isolation story
is now an explicit part of what the spike must settle.

### Net effect on approval

**Blocking owner decisions: 3** (questions 1, 2 and 4 above — self-hosting, recurring
bookings, tenancy shape). **Blocking technical spikes: 0** — no spike result changes which
alternative is chosen. See [[Calquartz — Architecture Decision Brief]] §13–14.

---

## Phase 5B resolution (2026-09-08)

Appended by Phase 5B ("Close Factual and Technical Prerequisites Before Architecture
Approval"). **Nothing above this line is deleted or rewritten** — the original questions,
their "why it matters"/"can it be deferred"/"working assumption" framing, and the Phase 5
status update are the historical record and remain readable exactly as written. This
section records that the owner has now explicitly answered all three questions that were
"confirmed still blocking" above, applying the
[[../engineering/Durable Decision Capture Policy|Durable Decision Capture Policy]]
established the same day.

**All three questions previously listed under "Confirmed still blocking: questions 1, 2
and 4" are now closed by explicit owner decision, obtained directly (not inferred from
conversation, prior recommendation, or the working assumptions this document itself
proposed):**

### Question 1 (hosted-only vs. self-hosting) — CLOSED

**DECISION (owner, 2026-09-08)**: **hosted-only.** Calquartz v1 is operated as hosted SaaS
on infrastructure the owner controls; customer/tenant self-hosting is not a v1 product
requirement and must not add v1 requirements, architecture complexity, support burden,
security surface, or testing obligations. The owner's own stated nuance, preserved
verbatim: implementation should remain "reasonably portable" with "clean deployment
boundaries" in case self-hosting becomes commercially necessary later — this is a
portability *preference* for how hosted-only is implemented, not a v1 self-hosting
*requirement*, and this document does not conflate the two.

This happens to match the working assumption this document already carried ("hosted-only
... the charter's 'commercial SaaS operation'... language all point this direction"), but
per the Durable Decision Capture Policy §9, a working assumption matching the eventual
answer is not itself why this is now closed — it is closed because the owner explicitly
decided it, on 2026-09-08, not because the prior assumption turned out to be right.

**Authoritative home**: this entry, plus
[[../architecture/Architecture Requirements and Constraints|Architecture Requirements and
Constraints]] §1 item 2 and §5.

### Question 2 (recurring bookings in v1) — CLOSED

**DECISION (owner, 2026-09-08): YES — recurring bookings are required in Calquartz v1.**
This **reverses** the working assumption this document and the Architecture Decision
Brief carried since Phase 3 ("not in v1"). Per Phase 5B's explicit instruction, this
decision closes only the **scope** question — recurring bookings are now a v1
requirement — and does **not** select a technical implementation. The booking data
model must be designed to accommodate recurring bookings from the start (the retrofit
risk this document's own Phase 3 framing warned about no longer applies, since the answer
is now known before schema design begins); the specific atomicity/partial-success model
remains an explicit open design question — see
[[../engineering/Calquartz Booking Correctness — Operational Review Process|Booking
Correctness — Operational Review Process]] §7, updated in the same pass to reflect
"confirmed in v1, design pending" rather than "unresolved, owner decision pending."

**Consequence for Phase 4's cost evidence**: the 8–14 engineer-day estimate (fresh, in any
alternative, no reuse offsetting it) is no longer a hypothetical cost of "if recurring
enters scope" — it is now a confirmed v1 cost. This does not change the architecture
recommendation (§16 of the Decision Brief already treated D's modest pull toward this
scope as insufficient to move the recommendation on its own), but it is now load-bearing
budget/scope information rather than a contingency.

**Authoritative home**: this entry, plus
[[../architecture/Architecture Requirements and Constraints|Architecture Requirements and
Constraints]] §1 item 7 and §5;
[[../engineering/Calquartz Booking Correctness — Operational Review Process|Booking
Correctness — Operational Review Process]] §7.

### Question 4 (flat vs. nested tenancy) — CLOSED

**DECISION (owner, 2026-09-08): flat tenancy.** Each customer/workspace is an independent
tenant with users/memberships within it; no organization-of-organizations hierarchy above
the workspace level is a v1 requirement. This matches the working assumption this
document already carried, for the same reason noted under Question 1: closed because the
owner explicitly decided it, not because the assumption was correct in advance. The
evidenced argument this document already recorded (Cal.diy's own still-unfinished
user-to-profile migration) remains the reasoning on record for why this was worth
deciding explicitly before schema design, rather than left as an assumption a future
session might silently treat as more settled than it was.

**Authoritative home**: this entry, plus
[[../architecture/Architecture Requirements and Constraints|Architecture Requirements and
Constraints]] §1 item 8 and §5.

### Net effect on approval, updated

**Blocking owner decisions: 0.** All three questions that blocked architecture approval
per the "Net effect on approval" section above are now resolved. **At the time this
section was written, this did not yet mean the architecture was approved** — see the
update immediately below for what has since changed.

Question 3 (seats/round-robin) is **unaffected** — it was already reclassified in the
Phase 5 update above as "resolve after approval, before schema freeze," not blocking, and
remains open on that same footing.

---

## Architecture approval (2026-09-08, ADR-001)

The three questions this document exists to resolve were never, themselves, "select an
architecture" — they were prerequisites to that separate decision, per
[[../Calquartz — Architecture Decision Brief|the Architecture Decision Brief]] §18's
checklist. That separate decision has now also happened: after an independent adversarial
review ([[../architecture/Phase 6 — Independent Adversarial Architecture Review|Phase
6]]) confirmed the standing A/B1 recommendation against the three decisions above, the
owner explicitly approved it. Full record:
[[../decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]]. This document's
own scope (product/tenancy-scope owner decisions) is unaffected in substance by that
approval — it is noted here only so a reader of this page's "no architecture approved"
framing above is not left with a stale impression of the project's current state.

---

## Phase 8A note (2026-09-08) — implementation-stack UNKNOWNs now tracked separately

Stack selection is an engineering decision, not a product-scope decision, so Phase 8A
resolves none of the product-scope questions tracked in this document — none is closed or
reclassified by it. The implementation-stack UNKNOWNs Phase 8A itself surfaced (auth
library selection, the nonexistent/ambiguous-local-time product policy, the recurring-
series atomicity model, whether a durable-job library eventually replaces a hand-rolled
protocol) are tracked in
[[../engineering/Phase 8A — Implementation Stack Selection|Phase 8A — Implementation
Stack Selection]] §19–20, not duplicated here, per the vault's "one fact, one home" rule.
The nonexistent/ambiguous-local-time policy in particular is a genuine open product
question this document had not previously named — noted here as a cross-reference only,
not resolved.

## Phase 8B note (2026-09-08) — stack recommendation approved as DECISION; UNKNOWNs unchanged

The owner has approved the Phase 8A stack recommendation as a DECISION (see
[[../engineering/Phase 8A — Implementation Stack Selection|Phase 8A]]'s own "Owner
Approval" section for the exact scope). This does **not** resolve, close, or reclassify
any question tracked in this document, and does not resolve any of the implementation-
stack UNKNOWNs Phase 8A itself lists in its §19–20 (auth library, ambiguous/nonexistent
local-time policy, recurring-series atomicity model, job-library-vs-hand-rolled) — those
remain exactly as open as the Phase 8A note above already stated. Cross-reference only.

