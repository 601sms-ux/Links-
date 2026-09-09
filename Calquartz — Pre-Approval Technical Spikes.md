---
type: synthesis
created: 2026-09-07
updated: 2026-09-08
sources: ["wiki/projects/001-calendar-os/Calquartz — Architecture Decision Brief.md", "wiki/projects/001-calendar-os/architecture/Database-Level Tenant Isolation — RLS and Alternatives.md", "wiki/projects/001-calendar-os/reuse/Reuse Audit Open Questions.md", "wiki/projects/001-calendar-os/architecture/Infrastructure Constraint Analysis.md"]
tags: [architecture, spikes, calendar-os, calquartz, phase-5]
confidence: medium
---

# Calquartz — Pre-Approval Technical Spikes

Five spikes whose results materially affect how an approved architecture is built. **None
is implemented by this document.** Each is specified so it can be executed and judged
without further design work.

> **Headline finding, stated up front so it is not buried**: **no spike below blocks the
> architecture decision.** Every one affects *how* an alternative is implemented, not
> *which* alternative is right. Spikes 1–4 are post-approval, pre-implementation. Spike 5
> is an operability prerequisite for deployment, not for choosing.
>
> This matters because it means the decision is gated by **three owner answers**
> ([[Calquartz — Architecture Decision Brief]] §13), not by engineering investigation.

**Inclusion test applied**: a spike belongs here only if its result changes a decision that
must be made at or immediately after approval. Investigations that are merely interesting,
or that can be answered during ordinary implementation, are excluded — three candidates
were rejected on that basis and are listed at the end.

> **Update (2026-09-08, Phase 7) — all five spikes now executed.** Spikes 1–4 ran against
> a real, disposable local PostgreSQL 17.11 instance (see
> `4. Calquartz Spikes/` outside this vault for full source and raw output, and
> `evidence/spike-N-*.md` in that same folder for the durable evidence records this
> section summarizes). **None disproved an A/B1 defining property; no ADR-001 reopen
> condition was triggered by any spike.** Per each spike's own scope, these results are
> **implementation constraints and recommendations, not architecture or production-stack
> decisions** — see [[decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]]'s
> own framing of the difference. Each spike's section below is left as originally
> specified; results are appended under a new "**Result (2026-09-08, Phase 7)**"
> subheading in each, following the same pattern spike 5's result already used.

---

## Spike 1 — Tenant-isolation backstop under real connection pooling — **EXECUTED, 2026-09-08**

**Question.** Can a database-level tenant-isolation backstop be implemented such that
(a) transaction-scoped tenant context survives connection-pool reuse under the pooling
configuration Calquartz will actually run, (b) a table not enrolled in the mechanism is
**denied** rather than silently unprotected, and (c) the background worker's database
identity is narrower than the web application's?

**Hypothesis.** PostgreSQL RLS driven by a transaction-local setting works correctly under
transaction-mode pooling provided the context is set inside the same transaction as every
query, and enrolment can be made fail-closed by deriving the enrolled-table list from the
schema at build time rather than maintaining it by hand.

**Why it matters.** This is the leading candidate mechanism for security floor item 12, and
the Phase 4 evidence is specifically discouraging about the delivery mechanism rather than
the idea. SnagTime's backstop enrols models through a hand-maintained 27-name list, so a
model added to the schema but not to the list is accessed with **no tenant context
installed** — the failure mode on ordinary schema growth is silent and open. Separately,
its worker role holds `USING (true) WITH CHECK (true)` policies on six tables and never
installs a context at all. Both properties must be deliberately *not* reproduced.

**Evidence required.**
- A representative multi-tenant schema (three or four tenant-scoped tables plus one
  deliberately left unenrolled) on a real PostgreSQL instance.
- The actual pooling configuration under consideration, in transaction mode.
- Concurrent requests for different tenants interleaved across a pool small enough to force
  connection reuse between them.
- A separate, narrower database role standing in for the worker.

**Success criteria.**
1. No query ever returns another tenant's row, including immediately after a connection is
   reused by a different tenant's request.
2. A query against the deliberately-unenrolled table is **refused**, not served.
3. The worker role can perform its intended writes and is refused writes outside them.
4. An admin/bypass path exists, is explicit, and is logged — not an ambient property of
   holding database credentials.
5. The cost of adding a new tenant-scoped table is bounded and mechanical.

**Failure criteria.** Any cross-tenant read or write under pool reuse; the unenrolled table
being served; the worker role being unable to do its job without broad grants; or the
mechanism proving undebuggable by one operator (a wrong-tenant or empty result that cannot
be traced to a policy in reasonable time).

**Effort.** 2–4 days.

**Architecture decision affected.** *Which* database-level backstop mechanism satisfies
security floor item 12 — RLS, a narrower alternative, or application-layer scoping with
enforced tooling plus compensating controls. **Not** which alternative (A/B1, B2, C, D) is
chosen: all four name a backstop, and [[Architecture Recommendation]] already states that
a pooling incompatibility would change the mechanism, not the alternative.

**Result (2026-09-08, Phase 7).** Every success criterion above was met under a real
60-request, 3-connection interleaved-tenant test against real PostgreSQL 17.11: zero
cross-tenant leaks under pool reuse; the deliberately-unenrolled table denied (and a
*second*, accidentally-unenrolled table caught by the same catalog-driven check — genuine,
unplanned corroboration); the worker role confirmed narrower than the app role and unable
to write outside its granted tables; the admin/bypass path confirmed explicit
(`BYPASSRLS`, a separate role) rather than ambient. **RLS remains the recommended
candidate — not selected as a decision.** Full evidence:
`4. Calquartz Spikes/evidence/spike-1-tenant-isolation-evidence.md`. **No ADR-001 reopen
condition triggered.**

---

## Spike 2 — Booking/occupancy invariant shape — **EXECUTED, 2026-09-08**

**Question.** Which database-level construct should enforce the double-booking invariant:
per-minute occupancy rows under a unique index, a `tstzrange` exclusion constraint over the
booking interval, or an exact-tuple idempotency key — and how does each behave under
concurrency, with buffers, and at realistic write volume?

**Hypothesis.** A `tstzrange` exclusion constraint gives the overlap coverage of SnagTime's
occupancy rows at Cal.diy's storage cost, and is the strongest candidate — but neither
evidence repository uses one, so there is no evidence either way and the interaction with
per-host before/after buffers is the part most likely to be awkward.

**Why it matters.** This is the single invariant the charter calls release-blocking, and
the two evidence repositories solve it incompatibly. SnagTime writes one row per occupied
minute (up to 960 rows per booking at its own configured maxima, rewritten on every
reschedule) and catches **any** overlap. Cal.diy writes one uuidv5 key per booking and
catches only **exact-duplicate** `(start, end, host)` tuples — 10:00–10:30 and 10:15–10:45
produce different keys and both insert. Phase 4 raised the exclusion constraint as an
uninvestigated third option.

**Evidence required.**
- All three constructs implemented against a representative booking table on PostgreSQL.
- N concurrent full booking-creation attempts at one slot, and separately at *overlapping
  but non-identical* slots — the case Cal.diy's mechanism silently admits.
- Buffers applied before and after each booking, since the effective interval is wider than
  the booking itself.
- Reschedule and cancel paths exercised, since occupancy must be released and re-acquired.
- Row counts and write volume measured per booking and per reschedule.

**Success criteria.** Exactly one booking commits per genuine conflict; every loser receives
the intended conflict response and not a 500; overlapping-but-non-identical attempts are
caught; buffers are honoured; and the write amplification per booking and per reschedule is
recorded for each construct.

**Failure criteria.** Any construct that admits two conflicting bookings; that requires a
lock ordering discipline to be safe; or whose reschedule path cannot release and re-acquire
atomically.

**Effort.** 2–4 days.

**Architecture decision affected.** The booking data model and the shape of the invariant.
**Not** the choice of alternative — all four place this constraint in PostgreSQL.

**Scope widened (2026-09-08, Phase 6 independent architecture review)**: recurring
bookings were confirmed in v1 scope the same day (see
[[questions/Architecture Open Questions|Architecture Open Questions]]'s Phase 5B
resolution). This spike's evidence-gathering must now additionally cover: the chosen
constraint's write pattern under a recurring series (potentially many occurrence rows/keys
created together, not one row per booking-creation request); whether the constraint holds
per-occurrence when a series partially fails; and reschedule/cancel semantics for a single
occurrence within a series versus the whole series. **Executed 2026-09-08 (Phase 7) — see
Result below, which covers both the original specification and this scope addition
together.** Full reasoning for the widened scope:
[[architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6 —
Independent Adversarial Architecture Review]] §5, §8.

**Result (2026-09-08, Phase 7).** All three candidates tested under real concurrent load
(20 simultaneous connections): the exact-tuple key (candidate 3) **reproduces the
Cal.diy-shaped defect empirically** — it admits overlapping-but-non-identical bookings and
has no buffer representation, disqualifying it as a standalone mechanism. The occupancy-
row candidate (1) and the exclusion-constraint candidate (2) both pass every correctness,
buffer, and reschedule test, including the newly-required recurring-series scenarios
(atomic all-or-nothing series creation, per-occurrence independent creation with
partial-series-failure isolation, and concurrent overlapping-series creation) — **both
atomicity models the Booking Correctness process left open (all-or-nothing vs. explicit
partial success) turn out to be an application-transaction-boundary choice, not a
constraint-mechanism property**, a genuine new finding. **Recommended: the exclusion
constraint (candidate 2)** — identical correctness to the occupancy-row candidate at
roughly 1/60th the write amplification for a typical booking. **Recommendation, not a
decision.** Full evidence:
`4. Calquartz Spikes/evidence/spike-2-booking-invariant-evidence.md`. **No ADR-001 reopen
condition triggered.**

---

## Spike 3 — SnagTime DST behaviour — **EXECUTED, 2026-09-08**

**Question.** Does slot generation built as "local midnight plus N minutes" actually
misplace slots across a daylight-saving transition, and what construction is correct?

**Hypothesis.** It does. SnagTime's `generateSlots` sets a day cursor with
`startOf("day")` in the schedule's zone and derives each availability window with
`plus({ minutes: startMinute })`. Luxon adds *exact* duration for minute units, so on a
spring-forward day a 09:00 local window should emit at 10:00 local, and on a fall-back day
at 08:00.

**Why it matters.** This is the audit's highest-confidence unverified defect, and it sits
inside the strongest availability reuse candidate. Cal.diy's `processWorkingHours` corrects
exactly this hazard explicitly, with an inline comment naming the 60-minute offset — strong
corroboration that the hazard is real for this design shape. SnagTime has **no DST test**:
its availability test file has four cases, none crossing a transition. Phase 4 could not
execute the check because installing dependencies was outside its boundary.

**Evidence required.** A schedule with a 09:00–17:00 window in `America/New_York`; slot
generation across the March and November transition days; the same in `Europe/London` for a
different transition date; and the same three cases run against a Cal.diy-style
offset-corrected construction.

**Success criteria.** The defect is either **reproduced** — the first slot's local
wall-clock time is not 09:00 on a transition day — or **refuted**, with the observed
behaviour recorded either way. A correct construction is demonstrated passing all cases.
The Cal.diy DST test matrix is confirmed portable as a specification.

**Failure criteria.** The behaviour cannot be reproduced deterministically, or a correct
construction cannot be demonstrated without adopting Cal.diy's patched date library
wholesale.

**Effort.** 0.5–1 day. This is the cheapest spike and it resolves the audit's most
consequential open inference.

**Architecture decision affected.** Whether SnagTime's slot generator is adoptable (Phase 4
classified it ADAPT *conditional on this fix*), and which date-library semantics Calquartz
must guard against. **Not** the choice of alternative — timezone handling is explicitly
architecture-independent.

**Result (2026-09-08, Phase 7).** **Hypothesis CONFIRMED empirically.** The
`startOf('day').plus({minutes})` construction misplaces the first slot by exactly ±1 hour
on every transition tested (`America/New_York` and `Europe/London`, both directions) —
and the drift propagates through the *entire* day's schedule, not just the first slot. Two
independently-correct constructions were demonstrated (direct wall-clock build; Cal.diy-
style offset correction). **New, recurring-series-specific result**: only the single
occurrence landing exactly on a transition date is misplaced in a recurring series —
occurrences before and after are correct — meaning a test that only checks a series'
first occurrence would miss this defect even when the series spans a transition. Also
newly found: Luxon silently resolves a nonexistent local time (shifts forward) and
silently picks one interpretation of an ambiguous local time, neither with a warning or
error — an open product-level question this spike surfaces but does not resolve.
**SnagTime's slot generator is adoptable only with this fix; its construction shape must
not ship as-is regardless of eventual date library**, since the defect is in the
arithmetic pattern, not the library. Full evidence:
`4. Calquartz Spikes/evidence/spike-3-dst-evidence.md`. **No ADR-001 reopen condition
triggered** — this is an architecture-independent correctness requirement, as already
stated above.

---

## Spike 4 — Durable job claim semantics — **EXECUTED, 2026-09-08**

**Question.** Does a PostgreSQL-backed job table with an atomic compare-and-swap claim, a
fencing token and a lease actually deliver exactly-one-execution under concurrent workers,
worker crash, and graceful shutdown?

**Hypothesis.** It does, and the four failpoints SnagTime tests are the right acceptance
criteria: crash after claiming (reclaimed exactly once after lease expiry), crash between
an accepted external effect and the local commit (reconciled without duplicate authority),
a stale effect arriving after its subject moved on (suppressed), and shutdown after
claiming (released **without** consuming retry budget).

**Why it matters.** Every alternative needs durable jobs, and the two evidence repositories
sit at opposite extremes. SnagTime's protocol is the audit's strongest ADAPT candidate.
Cal.diy's `getNextBatch()` is a bare `findMany(… take: 1000)` with no lease, no lock and no
status transition — two concurrent processors execute every task twice, and its
`@@unique([referenceUid, type])` deduplicates *enqueueing*, not *execution*. Proving the
correct shape before building on it is cheap; discovering the wrong one in production is
not.

**Evidence required.** A job table and two worker processes on real PostgreSQL; jobs with
observable side effects so double-execution is detectable; a kill applied mid-claim; a
`SIGTERM` applied mid-claim; and lease expiry forced.

**Success criteria.** Every job executes exactly once under two concurrent workers; a killed
worker's claimed job becomes reclaimable after lease expiry and is then executed exactly
once in total; a gracefully-stopped worker's job is released without consuming a retry
attempt; and a stale effect is suppressed rather than applied.

**Failure criteria.** Any double execution; a job permanently stranded after a crash; or a
graceful shutdown consuming retry budget such that repeated deploys exhaust a job's
attempts.

**Effort.** 2–3 days.

**Architecture decision affected.** The job/outbox module's internal design and the
queue-core versus effect-handler seam Phase 4 recommends drawing. **Not** the choice of
alternative — though a clean result here further weakens B2, since it removes the last
speculative argument for a queue accelerator.

**Scope widened (2026-09-08, Phase 6 independent architecture review)**: recurring
bookings being confirmed in v1 introduces a job *shape* this spike's original four
failpoints (crash-after-claim, crash-between-effect-and-commit, stale-effect-after-move-on,
shutdown-after-claim) were not designed around — a **scheduled, cross-tenant batch job**
(e.g. "materialize the next N days of occurrences across all active series"), as opposed
to a per-event job triggered by a single request. This spike must additionally establish:
whether the existing claim/lease/fence protocol extends cleanly to a batch job touching
many tenants' rows in one run; what happens if such a job is interrupted partway through a
large batch; and whether this job shape risks becoming a resource hot-spot in the
single-worker-process model. **Executed 2026-09-08 (Phase 7) — see Result below, which
covers both the original specification and this scope addition together.** Full reasoning
for the widened scope: [[architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6 — Independent Adversarial Architecture Review]] §3 finding 2.

**Result (2026-09-08, Phase 7).** **All original success criteria met** under real
concurrent workers and real lease-expiry timing (not simulated): exactly one winner among
10 workers racing to claim one job; a "crashed" worker's claim is correctly reclaimed
after lease expiry with the fencing token advanced and the effect applied exactly once
(via an idempotent effect log, not the claim mechanism alone); a stale, fenced-out
worker's write attempt is detectably rejectable by fencing-token comparison; graceful
shutdown returns a job to `pending` without incrementing its attempt count; retry
exhaustion correctly reaches a terminal `dead` state rather than retrying forever; a
long-held claim on one job does not block an unrelated job from being claimed
(`SKIP LOCKED`). **The scheduled cross-tenant batch-materialization shape was tested and
requires no new mechanism** — the same claim/lease/fence protocol, unmodified, correctly
handles 5-tenant/20-series batch processing, mid-batch crash-and-resume-from-checkpoint
(critically dependent on committing progress incrementally, not only at batch completion),
duplicate-batch-execution prevention, and non-starvation of ordinary jobs alongside a
large batch job. **This further weakens B2's case, per the standing recommendation's own
"a clean result here further weakens B2" reasoning** — the batch shape does not need Redis
or a separate scheduler either. Full evidence:
`4. Calquartz Spikes/evidence/spike-4-durable-jobs-evidence.md`. **No ADR-001 reopen
condition triggered.**

---

## Spike 5 — Read-only VPS inventory — **EXECUTED, 2026-09-07**

**Not a code spike.** Specified in full in [[Infrastructure Constraint Analysis]] §2. **Executed 2026-09-07** — full results in [[Calquartz — VPS Infrastructure Inventory]].

**Question.** What is actually running on the VPS today, and how much of its 2 CPU / 8 GB /
100 GB is already committed?

**Hypothesis.** None. This is measurement, and inventing an expectation would defeat its
purpose.

**Why it matters.** Every operability statement in [[Calquartz — Architecture Decision Brief]] rests on an advertised specification. "2 CPU / 8 GB" reads very differently if the
box is idle than if it is already running other services near their limits.

**Evidence required.** CPU count and load; RAM and swap in use; disk usage by mount; any
running containers with their configured limits; any existing PostgreSQL or Redis, with
version and memory configuration; what terminates TLS today; whether any backup already
runs; and which ports are actually listening.

**Success criteria.** A recorded, dated snapshot of all of the above, filed as raw
evidence, with a stated conclusion about whether headroom exists for one web process, one
worker and one PostgreSQL instance configured for the tenant-isolation mechanism chosen.

**Failure criteria.** Insufficient headroom for the lightest alternative — in which case the
finding is that existing load must be addressed before Calquartz is deployed at all, not
that a different architecture should be chosen.

**Effort.** Under 0.5 day.

**Constraints — read-only, and enforced.** Observation only. **Prohibited**: deletion,
restart, stop, prune, package installation or removal, configuration change, and deployment
of anything. No `docker prune`, no `FLUSHALL`, no schema or config changes. `SELECT` and
`INFO` queries only.

**Result.** CPU and memory headroom are large and largely uncommitted (load 0.06; 6.7 GiB of 7.6 GiB RAM free). Disk is the one constrained resource, at 79% used — but the constraint is 54 GB of reclaimable Docker build cache, not application data, and every alternative's own persistent-data footprint is trivial by comparison. A finding this spike did not anticipate: a live SnagTime production deployment already exists on the machine, serving `calquartz.com` with a valid certificate and an empty database — see [[Calquartz — VPS Infrastructure Inventory]] §14 and §20. That finding is a human decision, not an architecture one.

**Architecture decision affected.** None directly, confirmed. The measured headroom does not flip the choice — it removes the last UNKNOWN the recommendation rested on and confirms the lightest-footprint alternative was the right call. **Spike 5 is closed.**

---

## Sequencing

Spikes 1–4 are independent and can run in any order. Recommended order by
value-per-day: **3** (cheapest, resolves the audit's top open inference), then **2** (the
release-blocking invariant), then **1** (the security-floor mechanism), then **4**.
Spike 5 can run at any time and should run before any deployment commitment.

Total: **7–12.5 engineer-days** across all five, as originally estimated for a full
production-grade execution. **Actual (2026-09-08, Phase 7): all five executed in one
session** against a disposable local PostgreSQL instance, run in the order 3, 2, 1, 4 as
this section already recommended — the original day-estimates were sized for
production-hardened spike work (broader edge-case coverage, load testing at realistic
scale, checked into a real CI pipeline); this pass produced genuine empirical evidence
answering each spike's stated question, explicitly scoped narrower than that — see each
spike's own "Limitations" in its evidence record
(`4. Calquartz Spikes/evidence/spike-N-*-evidence.md`) for exactly what remains
unverified relative to the original, fuller specification.

---

## Candidates considered and excluded

Recorded so their exclusion is deliberate rather than an oversight.

| Candidate | Why excluded |
|---|---|
| API paradigm evaluation (REST vs. tRPC vs. GraphQL) | Does not affect approval. The architectural requirement is *one* API surface, not three; which one is an implementation choice made after approval, and reversible at a bounded cost. |
| Date-library selection (Luxon vs. Day.js vs. Temporal) | Subsumed by spike 3, which tests the *construction*, not the library. Whichever library is chosen must pass the same DST matrix. |
| Event-sourcing prototype for Alternative D | Would be justified only if D were a live contender on grounds this analysis supports. It is not: D's real advantage is auditability, achievable inside A/B1 via an append-only audit table. Prototyping D before that premise is challenged would be building to justify a choice rather than to test one. |

---

## Boundary

**No spike in this document has been implemented, and none may begin before the
architecture decision is recorded** (spike 5 excepted, which is read-only observation and
may be run at any time). Spikes are engineering investigation, not implementation: each
produces a recorded finding, not production code.

**Added 2026-09-08 (Phase 6 governance-tightening pass)**: none of spikes 1–4 currently
blocks A/B1's approval (per
[[architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6 —
Independent Adversarial Architecture Review]] §8), but a result demonstrating that no
candidate mechanism can satisfy one of A/B1's named defining properties — a database-level
tenant-isolation backstop (spike 1), a PostgreSQL-enforced booking invariant (spike 2), a
correct DST construction across a recurring series (spike 3), or the durable job-table
model extended to scheduled cross-tenant batch work (spike 4) — is a reason to reopen the
architecture review, not to quietly implement a workaround outside what was approved. See
that review §8 for the specific condition per spike.

**Update (2026-09-08) — the architecture decision this section's original sentence
gated on has now been recorded**: [[decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]] approves A/B1, satisfying this section's original precondition ("none may begin
before the architecture decision is recorded").

**Update (2026-09-08, Phase 7) — spikes 1–4 have since been executed**, per explicit
task instruction, against a disposable local PostgreSQL instance — see each spike's own
"Result (2026-09-08, Phase 7)" subsection above and
`4. Calquartz Spikes/evidence/` for full evidence. **None triggered the reopen condition
named above.** Executing these spikes did not authorize, and was not, implementation of
Calquartz itself — no Calquartz repository was created, no application code was written;
the disposable PostgreSQL instance used for spikes 1/2/4 was deleted after evidence
capture (see `4. Calquartz Spikes/evidence/00-environment-setup.md`).

## Related

- [[Calquartz — Architecture Decision Brief]] — §14 summarises these
- [[Database-Level Tenant Isolation — RLS and Alternatives]] — spike 1's cost enumeration
- [[Reuse Audit Open Questions]] — spikes 2 and 3 answer Q1, Q4 and Q5 there
- [[Test Reuse and Integration Risks]] — spikes 2 and 4 map onto tests to be carried forward
- [[Infrastructure Constraint Analysis]] — spike 5's full specification
