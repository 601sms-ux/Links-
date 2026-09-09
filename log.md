---
type: log
updated: 2026-09-07
---

# Log

Append-only. Newest at the bottom. `grep "^## \[" log.md | tail -5` for recent activity.

## [2026-07-15] schema | Vault initialized
Created `CLAUDE.md` schema, `index.md`, `log.md`.
Folder structure: `raw/` (immutable), `wiki/{sources,entities,concepts,questions,syntheses}`.
Moved the originating idea document to `raw/LLM Wiki.md` as source #1.

## [2026-07-15] ingest | LLM Wiki
Pages created: [[LLM Wiki]] (source), [[Compounding Wiki]], [[Retrieval-Augmented Generation]], [[Maintenance Burden]] (concepts), [[Obsidian]], [[Memex]] (entities).
Pages updated: index.md.
Contradictions: none — first source, nothing to contradict.
Notes: Ingested as the worked example of the ingest flow. Source is the vault's own design document, so [[Compounding Wiki]] is both a concept page and a description of this vault.

## [2026-09-06] schema | Second Brain reorganization — universal architecture established
Found two conflicting instruction sets at vault root: the original universal wiki schema
(`CLAUDE.md`, 2026-07-15) and two Cal.diy/SnagTime-hybrid-specific documents saved under
mismatched filenames — `Calquartz Instructions.md` (its own first line called itself
"# CLAUDE.md") and `Calquartz Second Brain Instructions.md` (its own first line called
itself "# SECOND_BRAIN_STORAGE.md") — that assumed the whole vault belonged to that one
project (sibling `2. Hybrid Scheduling SaaS/` workspace, `wiki/tenancy`, `wiki/booking`,
etc. at vault root).
Resolved by adding a `wiki/projects/<NNN-slug>/` convention to the root schema so the
vault stays universal and a project nests inside it instead of owning it.
Pages created: `wiki/projects/001-calendar-os/README.md`.
Pages moved: `Calquartz Instructions.md` → `wiki/projects/001-calendar-os/CLAUDE.md`;
`Calquartz Second Brain Instructions.md` → `wiki/projects/001-calendar-os/STORAGE.md`
(both renamed to match their own internal titles, annotated with a scope note, body
otherwise unedited).
Pages updated: root `CLAUDE.md` (added "Projects" section + folder-tree entry), `index.md`
(added Projects table).
Contradictions: not resolved by fiat — the project's proposed vault-wide topology is
flagged superseded in `STORAGE.md`'s scope note, not deleted or silently rewritten.
Notes: No repositories cloned, no Calendar OS code written, no "2. Hybrid Scheduling SaaS/"
workspace created — this pass is information architecture only, per instruction.

## [2026-09-06] ingest | SnagTime repository forensics (Project #001)
Cloned SnagTime read-only (shallow, `--depth 1`, branch `main`, HEAD `1c95490a...`) into
`2. Hybrid Scheduling SaaS/snagtime-reference/`, outside the Second Brain. Forensic pass run
by an isolated agent with zero knowledge of Cal.diy, to avoid cross-contamination.
Pages created: `raw/projects/001-calendar-os/repositories/snagtime/{SOURCE_MANIFEST,
FORENSIC_PASS_NOTES}.md`, `wiki/projects/001-calendar-os/SnagTime Forensic Analysis.md`.
Headline finding: tenant isolation enforced three layers deep (app + FORCE RLS + HMAC-signed
DB context) with a real 20-way concurrency test proving the double-booking guard; against
that, zero security headers beyond Referrer-Policy, a likely-broken Stripe webhook-dedup
check (cross-client `instanceof`), and no sweeper for abandoned PENDING_PAYMENT bookings
(permanent slot lock). Repo carries 1,886 occurrences of a prior product name ("tempocove")
baked into cookie names, DB roles and session GUCs — evidence this is a filtered export of
a private predecessor project, not original development history.
Contradictions: none to flag yet — single-repo pass.

## [2026-09-06] ingest | Cal.diy repository forensics (Project #001)
Cloned Cal.diy read-only (shallow, `--depth 1`, branch `main`, HEAD `e91bb0c3...`) into
`2. Hybrid Scheduling SaaS/cal-diy-reference/`, outside the Second Brain. Forensic pass run
by an isolated agent with zero knowledge of SnagTime.
Pages created: `raw/projects/001-calendar-os/repositories/cal-diy/{SOURCE_MANIFEST,
FORENSIC_PASS_NOTES}.md`, `wiki/projects/001-calendar-os/Cal.diy Forensic Analysis.md`.
Headline finding: two CRITICAL, live, reachable cross-tenant authorization bypasses,
introduced by the "community edition" de-featuring — a permission-check class was replaced
with an unconditional `return true` stub in eighteen files rather than the call sites being
removed or fixed, leaving any authenticated user able to read/modify/delete another
organization's team-owned event types and access team/org bookings by uid. A correct,
unused implementation (`eventOwnerProcedure`) sits beside the broken one in the same file.
No cross-tenant isolation test exists anywhere in the 407-file test suite that would have
caught this. Repo confirmed owned by the `calcom` GitHub org itself (not a third-party
fork); `apps/api/v2` carries an internal MIT-vs-UNLICENSED contradiction.
Contradictions: none to flag yet — single-repo pass.

## [2026-09-06] synthesis | SnagTime vs Cal.diy (Project #001)
Written after both independent forensic passes completed, reasoning only over their
already-filed findings (not a re-read of either repository).
Pages created: `wiki/projects/001-calendar-os/synthesis/SnagTime vs Cal.diy Synthesis.md`,
`wiki/projects/001-calendar-os/questions/Open Questions — Calendar OS Forensics.md`.
Pages updated: `wiki/projects/001-calendar-os/README.md` (status → Phase 2 complete),
`index.md` (Projects row).
Major incompatibilities flagged (not resolved): tenancy trust model (DB-enforced vs.
app-only), double-booking mechanism (per-minute occupancy rows vs. exact-tuple idempotency
key), API paradigm (one REST surface vs. three coexisting surfaces), and — most
important — Cal.diy's own feature-removal methodology (stub-to-always-true) identified as
the direct cause of its CRITICAL findings and flagged as never to be reused regardless of
what Calquartz chooses to strip from either source.
Earlier architectural hypotheses in this project's own `STORAGE.md`/`CLAUDE.md` (Postgres,
Redis, modular-monolith-plus-workers, RLS "where appropriate") were checked against evidence
rather than assumed: Postgres is supported by both repos' convergent choice; Redis is
supported by neither as a hard requirement; "modular monolith + workers" holds for SnagTime
but is contradicted by Cal.diy's three-scheduler accretion; RLS is shown valuable (SnagTime)
but not necessary for adoption (Cal.diy) — though its absence is directly implicated in
Cal.diy's CRITICAL bypasses.
Contradictions: none between the two repositories was silently resolved — all logged as
"major incompatibilities" in the synthesis doc, each requiring a Calquartz-specific design
decision in a later phase, not a merge.
Notes: no Calquartz architecture decided, no ADR written, no code written or copied, no
repository merged or modified. Per instruction, this phase stops here pending human review.

## [2026-09-06] phase | Phase 2 — Repository Forensics: COMPLETE
Date: 2026-09-06
Phase: Phase 2 — Repository Forensics (Project #001, Calendar OS)
Status: Complete
Scope: SnagTime and Cal.diy independently forensically analyzed (each pass blind to the
other repository) and cross-compared.
Evidence: Exact shallow-clone commits recorded — SnagTime `1c95490a4bfb498084fcb8295439befe194c229a`
(branch `main`), Cal.diy `e91bb0c38251b0ee6f87eec70ce4940822fb0cf3` (branch `main`). Source
manifests, forensic reports, cross-repository synthesis, and consolidated open questions
persisted under `raw/projects/001-calendar-os/` and `wiki/projects/001-calendar-os/`.
Result: Significant correctness, security, tenancy, testing, licensing, provenance, and
architectural trade-off evidence identified across both repositories.
Important findings: Cal.diy contains two CRITICAL reachable authorization bypasses
(stubbed permission checks in 18 files, two live); SnagTime contains notable
database-isolation and concurrency patterns alongside its own defects (a likely-broken
Stripe webhook dedup check, an unswept permanent-slot-lock path); licensing/provenance
questions remain unresolved for both (Cal.diy's `apps/api/v2` license contradiction,
SnagTime's undocumented chain of title).
Boundary: No repositories were merged or modified; no production code was written; no
Calquartz architecture was selected or frozen. The synthesis document was subsequently
corrected to strip language that could be read as architecture selection (convergent
evidence, PostgreSQL/Redis/RLS/job-execution/double-booking findings) and reframed as
open design questions and evidence-backed candidates for future evaluation, not decisions.
Next phase: A separate architecture-design investigation, using [[Open Questions —
Calendar OS Forensics]] as its starting agenda. Architecture decisions must not be
inferred from the Phase 2 synthesis.

## [2026-09-06] phase | Phase 3 — Calquartz Architecture Design: COMPLETE
Date: 2026-09-06
Phase: Phase 3 — Architecture Design and Evaluation (Project #001, Calendar OS)
Status: Complete
Scope: Established Calquartz's own requirements/constraints independent of either
evidence repository; designed four genuinely distinct, internally coherent architecture
alternatives; produced a Calquartz-specific threat model and a booking/payment/webhook/
calendar/job failure-mode analysis; evaluated the alternatives against one decision
matrix; classified all open questions (carried forward from Phase 2 plus new ones this
phase raised); produced a recommendation with an explicit strongest-argument-against and
strongest-alternative; prepared a ChatGPT adversarial-review packet.
Evidence: No new repository evidence gathered this phase — reasoned entirely from the
Phase 2 forensic findings (cited with provenance throughout) plus the project's own
charter. Confirmed, by active search of this workspace, that no server/infrastructure
inventory exists — recorded as an explicit UNKNOWN rather than invented.
Artifacts persisted: `wiki/projects/001-calendar-os/architecture/{Architecture
Requirements and Constraints, Architecture Alternatives, Multi-Tenancy Trust Model,
Failure-Mode Analysis, Threat Model, Architecture Decision Matrix, Architecture
Recommendation}.md`; `wiki/projects/001-calendar-os/questions/Architecture Open
Questions.md`; `wiki/projects/001-calendar-os/ChatGPT Architecture Review Packet.md`
(pointer); `raw/projects/001-calendar-os/chatgpt-consultations/
2026-09-06-calquartz-architecture-review.md` (full packet).
Result: A recommendation for Alternative B (seamed modular monolith, non-authoritative
Redis cache/queue, module-extraction seams, refined to include a PostgreSQL RLS tenancy
backstop as core rather than optional), reasoned from a near-tied decision matrix and the
charter's explicit solo-operator constraint. Alternative C (an isolated booking-engine
service) is named as the strongest alternative, and would become the stronger
recommendation if either of two unresolved product questions — self-hosting requirement,
public developer API for v1 — resolves in that direction.
Important findings: booking-correctness failure modes enumerated with one specific
do-not-repeat item carried from Phase 2 (SnagTime's abandoned-payment permanent-slot-lock
defect); the tenancy trust-model comparison shows a database-level backstop closes a
failure class application-only isolation does not, directly informed by Cal.diy's
CRITICAL findings; four product-scope questions (self-hosting, public API, recurring
bookings, tenant nesting depth) are flagged as most likely to change the recommendation
if answered.
Boundary: No Calquartz architecture has been approved, frozen, or decided. No ADR was
created. No code was written. Neither evidence repository was modified. The
recommendation is explicitly conditional and pending external review.
Next phase: Send the ChatGPT Architecture Review Packet for independent adversarial
review, then human approval, then ADRs — in that order. Architecture approval must not be
inferred from this phase's recommendation alone.

## [2026-09-06] phase | Phase 3 revision — independent review incorporated: COMPLETE
Date: 2026-09-06
Phase: Phase 3 revision — Independent Review Incorporation (Project #001, Calendar OS)
Status: Complete
Scope: One round of independent adversarial review (relayed by the owner) was
incorporated into the Phase 3 architecture analysis. Real production infrastructure (one
YouStable vProfessional VPS: 2 CPU, 8 GB RAM, 100 GB NVMe, 8 TB bandwidth; no further
infrastructure investment available) was recorded as a binding constraint.
Evidence: No new repository evidence gathered. Reasoned from the owner's infrastructure
disclosure and the review's specific critiques, re-examining the Phase 3 matrix and
recommendation against both.
Artifacts persisted: revised `wiki/projects/001-calendar-os/architecture/{Architecture
Requirements and Constraints, Architecture Alternatives, Architecture Decision Matrix,
Architecture Recommendation, Failure-Mode Analysis, Multi-Tenancy Trust Model}.md`; new
`wiki/projects/001-calendar-os/architecture/{Infrastructure Constraint Analysis, Redis
Justification Analysis, Database-Level Tenant Isolation — RLS and Alternatives, Security
Floor, Independent Review Response}.md`; revised `wiki/projects/001-calendar-os/questions/
Architecture Open Questions.md`; addendum appended to `raw/projects/001-calendar-os/
chatgpt-consultations/2026-09-06-calquartz-architecture-review.md` recording the review's
receipt (not rewritten — consultation records are preserved, not edited).
Result: The original Alternative A/B split was found to be an unfair comparison (B was
credited with module-boundary discipline a properly-engineered A should also have) and
the two were merged into one alternative, A/B1. Redis was separated out as its own
candidate (B2) and, worked through concretely against Calquartz's likely launch scale and
the fixed 8 GB VPS budget, classified as deferred infrastructure pending a named,
measured trigger — not adopted, not rejected outright. RLS's real engineering and
operational cost was enumerated in full (connection pooling, transaction-scoped context,
workers, admin operations, migrations, raw SQL, SECURITY DEFINER functions, testing,
bypass roles) and compared against alternative database-level mechanisms; the earlier
claim that it "costs nothing" was retracted. Security was reframed as a 13-item
non-negotiable gate rather than a weighted score, and all four alternatives were confirmed
to clear it. Alternative C's actual isolation properties were clarified (fault isolation
and a separable trust boundary — not independent scaling, which the single-VPS
infrastructure cannot provide) and decoupled from "needs a public API." Alternative D's
prior Reliability score was corrected: split into runtime reliability (not superior to the
other alternatives) and auditability/recoverability (genuinely superior, and retained as
its real advantage). Licensing/provenance was retired as a numeric matrix discriminator
and kept as a hard constraint. The revised recommendation remains A/B1 — a modular
monolith, Postgres-only, database-level tenant-isolation backstop as a named candidate —
with Redis and booking-engine extraction both explicitly deferred to concrete, named
trigger conditions rather than adopted speculatively or foreclosed structurally.
Important findings: the read-only VPS-utilization inventory this analysis depends on for
final operability confirmation has not been executed — no session has had access to the
actual machine; it is specified, not performed, in Infrastructure Constraint Analysis.
Boundary: No Calquartz architecture has been approved, frozen, or authorized for
implementation. No ADR was created. No code, migration, Redis installation, PostgreSQL
change, or deployment occurred. Neither evidence repository was modified.
Next phase: Owner review of the revised recommendation → explicit architecture approval
→ ADRs → implementation planning. Architecture approval must not be inferred from this
revision alone, and this revision has not itself been sent for a further round of
external review.

## [2026-09-06] phase | Phase 3 v1 scope correction — payments deferred: COMPLETE
Date: 2026-09-06
Phase: Phase 3 final scope correction (Project #001, Calendar OS)
Status: Complete
Scope: Explicit owner decision — payments are not part of Calquartz v1 (no payment
provider integration, no payment state machine, no payment webhooks, no subscription/
billing infrastructure in v1) — propagated consistently across all Phase 3 architecture
documents. No re-design performed: the merged A/B1 vs. B2 vs. C vs. D analysis and its
recommendation were re-examined against this change and found not to require re-scoring.
Evidence: No new repository evidence gathered. Reasoned from the owner's explicit v1
scope decision, checked against every Phase 3 document for contradictions.
Artifacts persisted: revised `wiki/projects/001-calendar-os/architecture/{Architecture
Requirements and Constraints (new §5: confirmed v1 / deferred / owner-decision-pending
scope table), Architecture Alternatives, Architecture Decision Matrix, Architecture
Recommendation, Failure-Mode Analysis, Security Floor, Threat Model, Multi-Tenancy Trust
Model, Infrastructure Constraint Analysis}.md`; revised `wiki/projects/001-calendar-os/
questions/Architecture Open Questions.md`. No Phase 2 forensic evidence was deleted —
payment-related findings in `SnagTime Forensic Analysis.md`/`Cal.diy Forensic
Analysis.md` and the payment-related rows in Failure-Mode Analysis/Threat Model are
retained, explicitly marked FUTURE VERSION rather than removed.
Result: Payments, subscriptions/billing, and payment webhooks are now documented as a
future-version capability, distinguished clearly from (a) historical repository evidence
about payment behavior in SnagTime/Cal.diy, which is unchanged, and (b) current v1 scope,
which excludes payments entirely. Confirmed v1 scope (auth, tenancy, authorization, event
types, availability, timezone/DST, calendar integration, public booking, booking,
cancellation, rescheduling, notifications, booking management, security/audit, backup/
recovery, observability) and deferred-from-v1 scope (payments/billing, Redis, a separate
booking engine, self-hosting-by-default) are now stated in one authoritative table in
Architecture Requirements and Constraints §5. Infrastructure language was also tightened
throughout: every "fits comfortably" claim was corrected to "fits within the advertised
resource ceiling; actual headroom UNKNOWN until the read-only inventory is performed,"
since that inventory has still not been executed.
Important findings: none material to the recommendation — the payment-deferral effect on
Architecture Decision Matrix's External correctness support row is a scope narrowing, not
a re-score (documented explicitly in that file). RLS remains a candidate mechanism, not a
selected one, pending its own technical spike; this pass did not perform that spike.
Boundary: No Calquartz architecture has been approved, frozen, or authorized for
implementation. No ADR was created. No code, migration, installation, or deployment
occurred. Neither evidence repository was modified.
Next phase: Owner review and explicit architecture approval of the (now v1-scope-
corrected) recommendation, then ADRs, then implementation planning. This pass only
established corrected v1 scope and prepared the recommendation for that approval step —
it is not itself an approval.

## [2026-09-06] phase | Phase 3 — Calquartz Architecture Design: OFFICIALLY CLOSED
Date: 2026-09-06
Phase: Phase 3 — Architecture Design and Evaluation (Project #001, Calendar OS) — closed
by the owner
Status: Complete. This is the authoritative closing entry for Phase 3, rolling up three
prior sub-passes recorded separately above (initial design and recommendation;
independent-review incorporation; v1 scope correction). Those entries remain the detailed
record; this entry is the single point a future session should read first to know where
Phase 3 landed.
Scope across the whole phase: established Calquartz's own requirements and constraints
independent of either evidence repository; designed and evaluated four genuinely distinct
architecture alternatives against one decision matrix; produced a Calquartz-specific
threat model, a booking/payment/webhook/calendar/job failure-mode analysis, and a
security floor treated as a gate rather than a score; incorporated one round of
independent adversarial review, correcting an unfair A-vs-B comparison, an unjustified
Redis inclusion, an uncosted RLS claim, a mis-scored Alternative D, and a mis-mapped
Alternative C; corrected v1 product scope to explicitly exclude payments/billing per
owner decision, without deleting the Phase 2 forensic evidence that scope decision leaves
behind for a future version.
Final recommendation standing at closure: **A/B1 — a well-designed modular monolith**
(one deployable, PostgreSQL as sole authoritative datastore, PostgreSQL-backed durable
jobs/outbox via one worker, deliberately enforced module boundaries with designed future
extraction seams, a database-level tenant-isolation backstop as a named candidate — not
yet RLS by default, pending a technical spike). Redis, a separate booking-engine service,
and payments are all explicitly deferred to named future triggers, not rejected outright.
Durable artifacts (all under `wiki/projects/001-calendar-os/` unless noted): `architecture/
{Architecture Requirements and Constraints, Architecture Alternatives, Multi-Tenancy Trust
Model, Database-Level Tenant Isolation — RLS and Alternatives, Redis Justification
Analysis, Failure-Mode Analysis, Threat Model, Security Floor, Infrastructure Constraint
Analysis, Architecture Decision Matrix, Architecture Recommendation, Independent Review
Response}.md`; `questions/Architecture Open Questions.md`; `ChatGPT Architecture Review
Packet.md` (pointer) with the full packet under `raw/projects/001-calendar-os/
chatgpt-consultations/`.
Boundary: **Closing Phase 3 is not the same as approving an architecture.** No Calquartz
architecture has been approved, frozen, selected, or authorized for implementation. No
ADR exists. No code, migration, installation, or deployment has occurred at any point in
this phase. Neither evidence repository has been modified or merged.
Next required action (owner, not Claude): explicit architecture approval — a distinct
decision, not implied by this closure. Only after that approval: ADRs recording the
actual decision, then implementation planning. Until that approval is given, no further
phase should be treated as begun.

## [2026-09-07] phase | Phase 4 — Calquartz Reuse & Extraction Audit: COMPLETE
Date: 2026-09-07
Phase: Phase 4 — Reuse and Extraction Audit (Project #001, Calendar OS)
Status: Complete
Scope: Forensic reuse analysis only. Forty subsystems audited independently in both
evidence repositories and each given one primary classification (REUSE / ADAPT /
REFERENCE ONLY / REJECT / BUILD FRESH), with per-candidate provenance, measured coupling,
covering tests, known defects, assumptions, required Calquartz changes, and fresh-vs-adapt
effort ranges. No Calquartz code written, no repository merged, forked or modified, no
dependency installed, no ADR created, no architecture approved.
Evidence: both read-only clones re-verified clean at their recorded commits — SnagTime
`1c95490a4bfb498084fcb8295439befe194c229a`, Cal.diy `e91bb0c38251b0ee6f87eec70ce4940822fb0cf3`.
Every classification is backed by directly-read source: line counts and import graphs were
measured per candidate module rather than estimated (SnagTime's whole server layer is 3,187
lines across 33 files; Cal.diy's `date-ranges.ts` has 4 imports against
`getUserAvailability.ts`'s 34 and `slots/util.ts`'s 45). Nothing was executed.
Artifacts persisted: new `wiki/projects/001-calendar-os/reuse/{Reuse Audit Summary,
SnagTime Reuse Candidates, Cal.diy Reuse Candidates, Build-Fresh and Rejected Components,
Reuse Security and Licensing Findings, Test Reuse and Integration Risks,
Engineering-Effort Comparison}.md`; new `wiki/projects/001-calendar-os/questions/Reuse
Audit Open Questions.md`; updated `wiki/projects/001-calendar-os/README.md`, root
`index.md`.
Result: the audit's central finding is that the two repositories are asymmetric in kind,
not merely in quality — **SnagTime is a code source and Cal.diy is a specification
source**. SnagTime's reusable value concentrates in about fifteen small, near-zero-coupling
modules (8 REUSE, 17 ADAPT); Cal.diy's is almost entirely non-code (3 REUSE, 22 REFERENCE
ONLY, 11 REJECT), with two genuinely extractable code assets. Estimated saving against an
all-fresh v1 build: roughly 25–45 engineer-days (~20–28%), concentrated in calendar
integration, the outbox/worker protocol, booking-correctness invariants, availability, and
the small utility layer — and explicitly negative in four named places where disentangling
costs more than rewriting.
Important findings: four material defects not recorded in Phase 2 — (a) an inferred DST
defect in SnagTime's `generateSlots`, the strongest availability candidate, where exact
minute addition from local midnight drifts across a transition and no DST test exists;
(b) Cal.diy's durable task queue claims nothing (`getNextBatch()` is a bare `findMany`),
so concurrent processors execute every job twice — this overturns a Phase 2 "directly
reusable" classification to REJECT; (c) Cal.diy's public `slots.getSchedule` accepts two
unauthenticated client-supplied flags that suppress external-calendar busy times, disclosing
a host's private busy times (bounded: not a booking-integrity bypass, and no cache
poisoning — both verified); (d) SnagTime's database-level tenant isolation does not cover
the worker role, and its contextual Proxy fails *open* when a model is added to the schema
but not to a hand-maintained 27-name list. Two Phase 2 UNKNOWNs were closed (the vendored
Day.js patch's contents; whether Cal.diy fails closed on calendar-provider failure).
Contradictions: two Phase 2 claims are not supported by the repository text at the same
commits — Cal.diy's CI required-checks gate blocks rather than passes on skipped jobs, and
SnagTime's "three-layer" tenant isolation is a property of the web role only. Both are
flagged with counter-evidence in appended `## Contradictions` sections on `Cal.diy Forensic
Analysis.md`, `SnagTime Forensic Analysis.md` and `synthesis/SnagTime vs Cal.diy
Synthesis.md`, and registered in `Reuse Audit Summary.md`. **No Phase 2 finding was deleted
or rewritten** — every original claim is left intact above the appended section.
Boundary: no Calquartz architecture approved, frozen or authorized for implementation; no
ADR; no application code; no schema or migration; no deployment; no repository modified,
merged or forked; no dependency installed; no product question silently resolved (five
owner decisions remain open and are now priced).
Architecture impact: MINOR — the standing Phase 3 recommendation (A/B1) is unchanged.
Three module-boundary refinements are recorded: the tenant-isolation backstop must fail
closed on schema growth, the job module's seam belongs between queue-core and effect
handlers, and slot generation should be exposed as a pure function.
Next phase: owner decisions (five), then explicit architecture approval, then ADRs, then
two small spikes this audit specified (database-level tenant isolation with the
fail-closed-on-new-model requirement; DST correctness), then implementation planning. This
audit is not an approval and does not authorize implementation.


## [2026-09-07] phase | Phase 5 — Architecture Decision Review Preparation: COMPLETE
Date: 2026-09-07
Phase: Phase 5 — Architecture Decision Review Preparation (Project #001, Calendar OS)
Status: Complete
Architecture approved? **NO — the architecture remains unapproved, pending independent
review and explicit owner approval.** Verified this pass: no `decisions/` folder exists, no
ADR exists anywhere in the vault, and a grep across every architecture and reuse document
found no approval language that was not an explicit negation.
Scope: condensed the Phase 1–4 evidence into a decision-ready package; re-audited the four
architecture alternatives for accidental bias; audited the decision matrix's treatment of
its own numbers; audited the Phase 4 reuse-savings estimate; extended the security floor;
re-classified which open questions actually block approval; and specified the pre-approval
technical spikes. No production code, no ADR, no migration, no dependency install, no
deployment; neither reference repository was modified.
Reviewed: Architecture Requirements and Constraints, Architecture Alternatives, Architecture
Decision Matrix, Architecture Open Questions, Architecture Recommendation, Failure-Mode
Analysis, Threat Model, Multi-Tenancy Trust Model, Security Floor, Database-Level Tenant
Isolation, Redis Justification Analysis, Infrastructure Constraint Analysis, Independent
Review Response, both Phase 2 forensic reports, the Phase 2 synthesis, all seven Phase 4
reuse documents, both question pages, the project README and this log.
Artifacts created: `wiki/projects/001-calendar-os/Calquartz — Architecture Decision
Brief.md` (18 sections, written to be reviewable without the forensic corpus);
`wiki/projects/001-calendar-os/Calquartz — Pre-Approval Technical Spikes.md` (five
specified spikes plus three candidates considered and excluded).
Artifacts updated: `architecture/Architecture Decision Matrix.md` (Phase 5 correction
appended — the 1–5 scale is now explicitly ordinal and qualitative, summation and totals
are forbidden, and the retirement of the prior 43-vs-42 totals is restated);
`architecture/Security Floor.md` (extended from 13 items to 16); `questions/Architecture
Open Questions.md` (Phase 5 status update appended); `reuse/Engineering-Effort
Comparison.md` (Phase 5 arithmetic correction appended); `wiki/projects/001-calendar-os/README.md`;
root `index.md`. **Nothing was deleted or rewritten** — every correction is an appended
section leaving the original text intact and readable, consistent with Phase 4's treatment.
Result — fairness audit of the alternatives: three of the four corrections the phase called
for had already been made during the Phase 3 revision and remain correctly applied (A and B
merged into A/B1; Redis constrained to non-authoritative use with named measurable
triggers; C's independent-scaling claim removed and its public-API mapping decoupled; D's
runtime reliability separated from its auditability, recoverability and replay properties).
One genuine gap was found and fixed: the decision matrix used a 1–5 numeric scale without
ever stating the numbers were ordinal judgements.
Result — reuse estimate audited and **withdrawn as stated**: Phase 4 claimed a saving of
25–45 engineer-days (~20–28%) from an all-fresh total of 102–178 against a reuse total of
77–133. Re-summing Phase 4's own per-subsystem table gives 126–222 all-fresh and 69–126
cheapest-path, implying 43–45%. The stated totals therefore do not reconcile with the table
they summarise, in either direction. Neither figure is trustworthy — summing 29 independent
range endpoints and differencing two such sums compounds error rather than cancelling it —
and four further reasons were recorded for treating the larger figure as optimistic. The
direction (reuse saves meaningful time) survives; the magnitude does not. Revised to
15–30%, confidence LOW, and explicitly demoted from the architecture decision entirely,
since reuse savings are essentially identical across all four alternatives and must not act
as a tie-breaker.
Result — security floor extended from 13 to 16 items, each added on Phase 4 evidence rather
than speculatively: browser-side security headers (neither repository satisfies this —
SnagTime ships Referrer-Policy alone; Cal.diy has a CSP and none of the other three);
boot-time production-configuration validation (Cal.diy's Dockerfile defaults two keys to
the literal string "secret"; SnagTime's boot gate is the demonstrated countermeasure); and a
dependency/licensing release gate (composition is UNKNOWN for both repositories). All four
alternatives clear all sixteen items, so the extension eliminates no alternative and does
not change the comparison — it raises the bar every alternative must clear.
Result — blocking questions reduced from five to three. Still blocking: hosted-only vs.
self-hosting; flat vs. nested tenancy; recurring bookings in v1. Reclassified to
post-approval, pre-schema-freeze: seats and round-robin — both are BUILD FRESH in each
repository and cost the same under every alternative, so neither can discriminate between
architectures, and holding the decision hostage to them would inflate a schema question
into an approval gate. Phase 3's original framing is preserved, not replaced.
Result — no technical spike blocks the decision. Five spikes are specified
(tenant-isolation backstop under real connection pooling; booking/occupancy invariant
shape, including the tstzrange exclusion constraint neither repository uses; SnagTime's
inferred DST defect; durable job claim semantics; and the read-only VPS inventory),
totalling 7–12.5 engineer-days. Every one affects how an approved architecture is
implemented, not which alternative is right. The read-only VPS inventory specified in
Phase 3 has **still never been executed**, so actual machine headroom remains UNKNOWN.
Recommendation standing at close: **A/B1 — a well-designed modular monolith**, unchanged
from Phase 3 and unchanged by anything found this phase. Named evidence that would move it:
confirmed self-hosting or an untrusted-caller public API (toward C); measured Postgres load
from the specific query classes Redis would offload (toward B2); audit/compliance becoming
materially more central (toward D); or the VPS inventory revealing the machine cannot host
the lightest alternative at all (an operability finding, not an architecture one).
Unresolved at close: the three owner decisions above; the tenant-isolation mechanism
(candidate, not selected); the double-booking invariant's shape; SnagTime's DST behaviour;
actual VPS utilisation; dependency licence composition for both repositories; SnagTime's
chain of title; and Cal.diy's `apps/api/v2` licence contradiction (which blocks nothing
currently proposed — no candidate originates there).
Boundary: no architecture approved, selected or frozen; no ADR created; no production code
written; no migration, dependency install, database change, system-configuration change or
deployment; no repository merged, forked or modified; no product question silently
resolved; payments not reintroduced into v1 scope at any point.
Next required action (owner, not Claude): read `Calquartz — Architecture Decision Brief`,
answer the three blocking questions, send the brief for independent adversarial review,
then record an explicit architecture decision. Only after that: the ADR, then the spikes,
then implementation planning.

## [2026-09-07] phase | Phase 6 — VPS Infrastructure Inventory: COMPLETE
Date: 2026-09-07
Phase: Phase 6 — Read-Only VPS/Server Infrastructure Inventory (Project #001, Calendar OS)
Status: Complete
Scope: executed, for the first time, the read-only infrastructure inventory task specified
in [[Infrastructure Constraint Analysis]] §2 (Phase 3) and named as spike 5 in
[[Calquartz — Pre-Approval Technical Spikes]] (Phase 5). SSH access to the production VPS
was found already configured locally (a working key and a known host); connectivity was
verified before any inventory command ran. All commands were observational: `SELECT`/`SHOW`
in PostgreSQL, `docker ps`/`inspect`/`system df` (no prune), `ss`/`ufw status`, `du`/`df`,
`systemctl list-units`, `curl` GETs against the public health endpoint. No install, upgrade,
restart, stop, delete, prune, migration, config change, firewall change, secret rotation, or
certificate change was performed.
Evidence: full findings persisted to `wiki/projects/001-calendar-os/architecture/Calquartz —
VPS Infrastructure Inventory.md`, dated and command-sourced throughout, with an explicit
integrity-verification section (§22) confirming container uptimes, file-modification times,
disk usage, package-manager history and backup-file states were unchanged from before to
after the session. No secret value (password, key, token, connection string) appears
anywhere in the report — where a credential was needed for a read-only query it was sourced
and used entirely inside the target container.
Result — the machine is real and matches its advertised specification: Ubuntu 24.04.4 LTS,
2 vCPU (Intel Xeon E5-2680 v4), 7.6 GiB RAM, 99 GB ext4 root filesystem, KVM guest, booted
2026-09-06. CPU and memory headroom are large and largely uncommitted (load average 0.06
across 1/5/15 minutes; 6.7 GiB of 7.6 GiB RAM available; 15.6 GB swap entirely unused). Disk
is the one constrained resource — 74G used of 99G (79%), 20G free — and the constraint is
54.12 GB of Docker BuildKit build cache (47.97 GB Docker-reported reclaimable), not
application data; the running application's own persistent footprint is a 68 MB PostgreSQL
volume and 13 MB of Caddy logs. Redis is confirmed absent. Only ports 22, 80 and 443 are
publicly listening, behind an active default-deny `ufw` firewall; PostgreSQL port 5432 is
not published to the host.
Important finding, not anticipated by any prior phase: **a complete, live, healthy
production deployment already exists on the VPS.** Four Docker containers (web, worker,
PostgreSQL 18.6, Caddy) have been running 22 hours under `restart=unless-stopped`, serving
`calquartz.com` under a valid Let's Encrypt certificate (expires 2026-12-03), with both
public health endpoints returning HTTP 200. The deployed application is **SnagTime**, not
Calquartz application code — Docker images, the compose project, and the database role
topology (`tempocove_owner`, `tempocove_app`, `tempocove_worker`, etc.) match
[[SnagTime Forensic Analysis]] exactly. SnagTime's row-level-security backstop is live and
enforced in production: 29 of 31 public tables carry `FORCE ROW LEVEL SECURITY` under 175
policies. The database contains zero business rows (0 users, 0 workspaces, 0 bookings, 0
event types) — the deployment has never been used by a real tenant. Automated encrypted
daily backups run via cron (`pg_dump` over the Unix socket → AES-256-CBC/PBKDF2 → chmod 600),
with three dated artifacts present; restore capability is explicitly UNVERIFIED, not
assumed working merely because files exist, per instruction. The deployed source tree at
`/opt/calquartz` is not a git repository — no commit, branch or remote is recoverable from
the host, so its exact provenance relative to the known SnagTime clone is UNKNOWN.
One security finding requiring attention, not remediated (read-only boundary): the 21 files
in `/opt/calquartz/secrets/` are mode 0644 (world-readable) inside a 0700 directory —
practical exposure is currently low (directory mode blocks traversal; only root has a shell
account on the host) but the files' own protection is absent and depends entirely on the
directory mode and on no copy ever being made. Flagged REQUIRES HUMAN DECISION, not
auto-fixed, since changing a file mode is a modification and this phase was read-only.
Also recorded: four distinct product names in simultaneous live use on one machine
(`snagtime` in images/containers, `tempocove` in the database/roles/GUCs/backup filenames,
`calquartz` in the hostname/directory/domain/cron job, and `Calendarr` in the Caddyfile's own
header comment and `CALENDARR_DOMAIN` variable) — a provenance/clarity issue consistent with
Phase 2's finding that SnagTime is itself a rebrand, now compounded by a third and fourth
name at deployment time. Filed as a human decision, not resolved.
Artifacts created: `wiki/projects/001-calendar-os/architecture/Calquartz — VPS
Infrastructure Inventory.md` (22 sections per the task's required structure, including
POTENTIALLY SAFE TO CLEAN, DO NOT TOUCH, REQUIRES HUMAN DECISION, and a factual-only
Architecture Implications section that selects nothing).
Artifacts updated: `architecture/Infrastructure Constraint Analysis.md` (§2's inventory
marked executed, pointing to the new report); `Calquartz — Pre-Approval Technical Spikes.md`
(spike 5 marked EXECUTED and closed, with its result summarized); `wiki/projects/
001-calendar-os/README.md`; root `index.md`. **Nothing was deleted or rewritten** — every
update is additive, consistent with the treatment used in Phases 4 and 5.
Boundary compliance verified in the report itself (§22): no package installed or removed
(confirmed against `dpkg.log`); no service restarted (uptime increased monotonically, no
reboot); no container stopped, started, or pruned (identical counts and "Up" durations
before and after); no file under `/opt/calquartz` modified (`find -newermt` empty); no
database DDL/DML/role/extension/setting change (read-only `SELECT`/`SHOW` only); no
firewall, SSH, or TLS change; no secret rotated or exposed in the report; no repository
(Calquartz, SnagTime, or Cal.diy) modified — this phase touched only the live VPS, which is
outside both reference-repository clones.
Architecture decision: **unaffected in substance, strengthened in confidence.** The
measured headroom removes the "actual headroom UNKNOWN" caveat that qualified every fit
claim in [[Architecture Recommendation]] and [[Calquartz — Architecture Decision Brief]],
and it removes the caveat in the direction that favors the standing recommendation — the
lightest-footprint alternative (A/B1) now has measured, not merely advertised, headroom.
**No architecture was approved, selected, or frozen by this phase.** The discovery of a live
SnagTime deployment is a new fact for the owner to act on, not an architecture selection —
it does not by itself make SnagTime "the" Calquartz codebase, and this report explicitly
declines to draw that conclusion.
Unresolved at close: whether backups actually restore (untested by design); the exact
provenance of the deployed source tree (no git history on host); whether any off-machine
backup copy exists; sustained behavior under real traffic (zero users on the deployment);
what the deployment is *for* (rehearsal, staging, soft launch, or abandoned experiment) —
explicitly a human decision, not inferred; and all three owner decisions and the four
remaining technical spikes carried forward unchanged from Phase 5.
Boundary: no architecture approved, selected, or frozen; no ADR created; no production code
written; no migration, dependency install, database change, system-configuration change,
package installation, service restart, or deployment performed; no repository merged,
forked, or modified; no secret exposed or rotated; no product question silently resolved.
Next required action (owner, not Claude): review this inventory, particularly §14 (existing
Calquartz/SnagTime evidence) and §20 (REQUIRES HUMAN DECISION — what the existing deployment
is for, the secret-file permissions, disk cleanup, and product naming), then proceed with
the Phase 5 approval checklist. No technical spike or implementation should begin before
that review.

## [2026-09-07] phase | Phase 7 — Legacy SnagTime Decommission: COMPLETE
Date: 2026-09-07, 16:44–16:47 UTC
Phase: Phase 7 — Legacy SnagTime Decommission (Project #001, Calendar OS)
Status: Complete
Owner decision: explicit, direct instruction to abandon the live SnagTime-derived
deployment discovered on the production VPS by the Phase 6 inventory, rather than adapt,
repair, migrate, or continue from it. Calquartz will be built fresh; SnagTime and Cal.diy
remain reference/knowledge sources per the existing forensic and reuse record, not
implementation foundations. This entry records execution of that instruction; it is not
itself a further decision.
Scope: a one-time destructive operation against the remote production VPS only. Preceded
by a final read-only safety re-check (business-data row counts re-verified at zero,
independently, immediately before deletion) and a full provenance capture written to
immutable raw evidence before any destructive command ran. Followed by a full clean-state
verification. The VPS operating system, SSH access, the authorized key, UFW, Docker
Engine itself, swap, NTP, and DNS were never touched, per explicit boundary. Neither
reference-repository clone (SnagTime, Cal.diy) nor any Second Brain document was modified
by the remote operation itself.
Provenance preserved before destruction: raw/projects/001-calendar-os/deployments/
legacy-snagtime-provenance-capture-2026-09-07.txt — full image digests, container
inspect output, compose project identity, database identity, sha256 hashes of the three
encrypted backups (content never decrypted), a complete source-tree listing (200+ files),
the TLS certificate fingerprint, and the verbatim (non-secret) content of the cron
definition and backup log. No secret value was captured or exposed at any point — every
credential needed for a read-only check was sourced and used entirely inside its
container.
Resources removed: the 4 running containers (web, worker, postgres, caddy); volumes
snagtime-production_postgres_data (the tempocove database), _caddy_config, _caddy_logs,
and one anonymous postgres-mount volume; the snagtime-production_default network; 7
SnagTime-tagged Docker images (6.24 GB); the entire 54.12 GB Docker BuildKit build cache;
the /etc/cron.d/calquartz-backup cron job; /var/log/calquartz-backup.log (content
preserved in the provenance capture first); and the entire /opt/calquartz directory
(source tree, all 21 files under secrets/, pg-ca-private/, the three encrypted backups,
both compose files).
Resources deliberately preserved, with reasoning recorded in the report: the
snagtime-production_caddy_data volume (holds the live, valid Let's Encrypt certificate
and ACME account for calquartz.com, expiring 2026-12-03 — domain infrastructure, not
SnagTime application data; destroying it would force re-issuance against Let's Encrypt's
weekly rate limit) and the caddy:2-alpine base image (generic, reusable, not
SnagTime-specific). Everything else generic to the VPS (OS, SSH, UFW, Docker Engine,
swap, NTP, DNS) was untouched by construction, not merely by outcome.
Result — disk reclaimed: 74G used (79%) before, 20G used (21%) after — 54 GB reclaimed,
consistent with the build-cache figure the Phase 6 inventory had already identified as
the dominant consumer. Result — clean-state verification: 0 containers, 1 image
(caddy:2-alpine, preserved), 1 volume (caddy_data, preserved), 0 B build cache; no
PostgreSQL process, container, or package remains; port 5432 no longer listening; only
port 22 (SSH) listening externally, UFW rules unchanged; /opt/calquartz confirmed
removed; no snagtime/tempocove process found anywhere on the host; uptime advanced
monotonically with no reboot; SSH access itself verified working throughout; swap
untouched at 0 used of 15.6 GB; https://calquartz.com/ now fails to connect, as expected,
while DNS for the domain remains untouched and still resolves to this host.
New provenance finding, not previously recorded anywhere in this project: the deployed
build's commit-like tag (7845e46b75bd83e844d8adb0f946c3b804532bce) does not match the
Phase 2 forensic commit (1c95490a4bfb498084fcb8295439befe194c229a) — the deployed tree
was never a git repository, so the relationship between the two cannot be established
further and is recorded as UNKNOWN rather than assumed. Also recorded before destruction:
the deployed tree was not a stock SnagTime clone — it included a docs/payments-restore/
archive of manually-removed Stripe code (consistent with this project's v1
payments-deferred scope) and genuine Calquartz-branded assets, custom terms/privacy
pages, and a deployment log, indicating real customization work had been done on top of
the SnagTime base before this decommission. That work is now destroyed per the owner's
explicit instruction; its prior existence is recorded so it is not silently lost from the
record.
Artifacts created: wiki/projects/001-calendar-os/architecture/Calquartz — Legacy SnagTime
Decommission Report.md (sections A–H exactly as specified: owner decision, provenance
preserved, resources removed, resources preserved, disk reclaimed, VPS state, remaining
unknowns, and an explicit Calquartz-starting-state statement); raw/projects/
001-calendar-os/deployments/legacy-snagtime-provenance-capture-2026-09-07.txt (immutable
raw evidence, header-annotated, collected before any destructive command).
Boundary compliance: no VPS OS reinstall, repartition, provider-configuration change, SSH
access change, UFW disable/removal, root-account change, key removal, VPS destruction,
DNS change, or unrelated system-service/user-data modification occurred at any point —
verified in the report's own §F and re-confirmed by this entry. No Calquartz application
was created, no database initialized, no code written, no dependency installed, no
repository cloned or merged into the production directory, no migration created, nothing
deployed, no production secret configured, no architecture decided, no ADR created, no
Redis or booking-engine work begun, no feature development started. Neither SnagTime nor
Cal.diy reference-repository clone was modified.
Calquartz starting state, stated explicitly per instruction: old SnagTime deployment —
RETIRED. Old SnagTime code — NOT the Calquartz foundation. Cal.diy — REFERENCE SOURCE.
SnagTime — REFERENCE SOURCE. Calquartz implementation — NOT YET BUILT. Production
deployment — NOT YET DEPLOYED. The VPS is clean and operational, available as a future
deployment target once an architecture is approved.
Boundary: no Calquartz architecture approved, selected, or frozen by this phase — the
standing (unapproved) recommendation in [[Calquartz — Architecture Decision Brief]] and
the three blocking owner decisions from Phase 5 are unaffected and unchanged by this
operation.
Next required action (owner, not Claude): none required by this phase specifically — the
decommission is complete and the VPS is in a clean, known state. The next substantive
step remains the Phase 5 architecture approval checklist (owner decisions, independent
review, explicit approval, ADR) before any Calquartz implementation begins. This phase
does not itself request or imply that implementation should now start.

## [2026-09-08] phase | Phase 5A — Engineering Capability Foundation: COMPLETE
Date: 2026-09-08
Phase: Phase 5A — Engineering Capability Foundation (Project #001, Calendar OS)
Status: Complete
Scope: not an implementation phase. No Calquartz application code was written and no
architecture alternative was assumed approved. Built the smallest reusable
engineering-capability layer for implementing Calquartz safely and verifiably once
building starts, per an explicit north-star test (a component must increase engineering
capability, increase verification confidence, preserve institutional knowledge, or reduce
human intervention, or it is rejected). Role separation was preserved throughout and
checked explicitly in a closing self-audit: Second Brain as institutional memory and
authority, ECC as engineering mechanism/orchestration source only, Claude as
implementation executor, ChatGPT as an unranked independent-reviewer process step (not a
knowledge-source tier), and the human as final authority for defined high-risk decisions.
Artifacts created (fourteen files, all under `wiki/projects/001-calendar-os/engineering/`,
all carrying a provenance/classification/scope/status frontmatter block):
`Phase 5A — Engineering Capability Foundation.md` (entry point, deliverables index,
role-separation table, a diagram of how the pieces compose, ten remaining unknowns, and a
self-audit against unnecessary complexity, duplicated authority, contradictory
instructions, unsafe automation, hidden external-model delegation, false confidence,
missing verification, missing provenance, and stale project assumptions);
`Authority, Risk, and Routing Model.md` (a six-rank source-of-truth precedence — explicit
human instruction, Second Brain DECISIONs, Second Brain RECOMMENDATIONs, Approved+Promoted
learned knowledge, actual source code/system state, ECC, external research, in that
order — a two-tier LOW/HIGH-CRITICAL risk classifier keyed to eight named domains and
defaulting to HIGH when unsure, and an automatic routing table mapping architecture/
feature/bug/migration/deployment/destructive-operation task categories to required review
chains and human-gate requirements); `ECC Adoption Map.md` (converts the prior read-only
ECC audit into a classification table covering roughly 30 named components plus the
broader rejected/deferred categories, with full ADOPT-by-reference records for
search-first, strategic-compact, and config-protection.js, full ADAPT drafts for seven
core agents, deferred-but-reasoned adaptation plans for database-reviewer and
database-migrations, and explicit non-adoption of security-review/verification-loop as
superseded by two new Calquartz-specific documents); seven full agent-definition drafts
under `engineering/drafts/agents/` (planner, architect, tdd-guide, code-reviewer,
security-reviewer, build-error-resolver, refactor-cleaner), each keeping ECC's verified
least-privilege tool grant and embedded Prompt Defense Baseline unchanged while rewriting
triggers and checklists to Calquartz's own domains, each explicitly forbidden from
reintroducing payments or silently assuming an unapproved architecture;
`Calquartz Security Floor — Operational Review Process.md` (the existing 16-item Security
Floor converted into a per-item "how to check" / "what a PASS requires" procedure, with
every row currently UNKNOWN against real code because none exists — stated explicitly
rather than left implicit); `Calquartz Booking Correctness — Operational Review Process.md`
(the Failure-Mode Analysis converted into ten operationalized categories — double booking,
idempotency, retries, transactional atomicity, worker failure, cancellation/reschedule
races, recurring bookings, timezone/DST, external calendar/notification state machines,
and payments explicitly excluded from v1 — with three categories explicitly marked
blocked on the still-unrun technical spikes 2 and 3, and recurring bookings explicitly
gated on the still-open owner decision); `Verification Evidence Format.md` (a durable
YAML-shaped record — what was checked, exact method, environment, timestamp, per-item
Security-Floor/Booking-Correctness results, limitations, unresolved failures never
silently dropped, human-gate status — with an explicitly-labeled illustrative-only example
so it cannot be mistaken for a real verification later); `Learning Lifecycle and Promotion
Policy.md` (Observation to Candidate to Explicit Validation to Approved to Promoted to
Deprecated/Rejected, with a hard 0.0 auto-application ceiling for seven guardrail domains
— tenancy, auth/authz, booking correctness, secrets, database schema, deployment,
destructive operations — regardless of confidence score, directly overriding ECC
continuous-learning-v2's own documented 0.7-confidence auto-approval default; plus a
negative-knowledge record shape for rejected approaches, extending this project's existing
Build-Fresh and Rejected Components precedent into the learning system); `Destructive-
Operation Protection Design.md` (a six-layer model — capability boundary, hard-coded
classification, mechanical keyword signal as a forcing function only, explicit
confirmation with live pre-action re-verification, least privilege at the point of action,
and a durable record — validated directly against the two real precedents this project has
already executed, the Phase 6 read-only VPS inventory and the Phase 7 SnagTime
decommission, rather than proposed in the abstract); `Engineering System Evaluation
Harness.md` (a manual, non-automated scorecard measuring routing correctness, false-
positive and false-negative routing with explicit note that false-negative is the more
dangerous asymmetry, verification completeness, unnecessary agent invocation, human-
intervention accuracy, and learning-promotion accuracy once that system is ever enabled);
`Synthetic Evaluation Tasks.md` (eight hypothetical tasks spanning ordinary feature,
security-sensitive feature, tenant-isolation bug, booking-concurrency bug, DST bug,
migration, destructive operation, and ambiguous architecture question — three dry-run
traced on paper against the routing model, producing one genuine design refinement: the
routing table's "architecture change" row needed to distinguish "cite the Second Brain's
existing answer" from "escalate to the owner," since not every architecture-shaped
question is actually unresolved).
Artifacts updated (additive only, no existing content rewritten): `wiki/projects/
001-calendar-os/README.md` (Phase 5A status paragraph and a new engineering/ documents
section); root `index.md`; this log entry.
ECC adoption, concretely: ten components adopted or adapted (search-first,
strategic-compact, config-protection.js by reference; the provenance-metadata requirement
as a standing policy; seven core agents fully drafted). Two components explicitly
superseded rather than adopted (security-review, verification-loop) because this phase's
own Calquartz-specific replacements cover the same ground with project-specific evidence
instead of generic content. deployment-patterns rejected as written (assumes multi-
instance infrastructure this project's own Infrastructure Constraint Analysis already
ruled out) with its concept deferred. All five ECC multi-* external-model-bridge commands
confirmed still rejected, unchanged from the prior audit, and verified absent from every
drafted artifact by explicit self-audit check. continuous-learning-v2 confirmed still not
enabled, per this phase's own explicit operating rule.
Boundary compliance: no ECC component was installed anywhere (every adapted artifact is
staged text in this vault, not a live Claude Code configuration); ~/.claude was not
touched; no production infrastructure was touched; no Calquartz application code was
written; no architecture alternative was assumed approved or silently selected (the
architect agent draft's central design exists specifically to prevent this); no ADR was
created; no recommendation was silently turned into a decision; FACT/INFERENCE/
RECOMMENDATION/DECISION/UNKNOWN distinctions were preserved throughout, most visibly in
the Security Floor and Booking Correctness documents' explicit "UNKNOWN until real code
exists" framing; payments were not reintroduced into v1 scope anywhere, checked explicitly
across all fourteen new artifacts in the closing self-audit.
Important findings: one design refinement (the architecture-routing row's "already
answered vs. genuinely open" distinction, found via Task 8's dry run); three of the ten
Booking Correctness categories and one Security Floor item remain explicitly blocked on
work this phase did not perform (spikes 1-3, still not run); the seven-domain guardrail
list used identically across the risk classifier, the destructive-operation gate, and the
learning policy was verified to actually match across all three documents rather than
assumed consistent.
Boundary: no architecture approved, selected, or frozen; no ADR created; no production
code written; no ECC component installed; no continuous learning enabled; no external-
model delegation used or introduced; no production infrastructure, VPS, or repository
modified; no Second Brain page rewritten (only cited).
Next required action (owner, not Claude): none required specifically by this phase — the
engineering-capability layer is staged and ready to be installed once a real Calquartz
repository exists, which remains gated on the same three owner decisions and the
independent-review/approval sequence already specified in the Architecture Decision
Brief. This phase does not itself request or imply that implementation should now begin.

## [2026-09-08] phase | Phase 5A consistency correction pass: COMPLETE
Scope: explicit owner-requested pass over Phase 5A's own artifacts — reconcile against
current Second Brain state, audit for internal contradictions, complete the untraced
synthetic evaluations. Not an architecture approval, not an ECC install, not the start of
Calquartz implementation.
Contradictions found and fixed (six, all within Phase 5A's own artifact set, none
requiring a change to the architecture recommendation or an invented resolution to an
open owner decision): (1) fabricated `§I.7.1`-style subsection citations against
[[Test Reuse and Integration Risks]], which has one subsection with a numbered list, not
ten subsections — fixed to `§I.7 (item N)` across four files; (2) [[ECC Adoption Map]]'s
own frontmatter used `scope: mixed` before its own §3 schema allowed that value — fixed by
adding `mixed` as a legal third value; (3) [[Learning Lifecycle and Promotion Policy]] §2
listed seven domains while claiming to be "the same fixed list" as the risk classifier's
nine — it was missing timezone/DST and security-controls-generally, a real gap that could
have let a DST-related learned instinct auto-apply despite spike 3 treating DST as
unresolved — fixed, with the seven→nine count propagated to two other files; (4)
[[Authority, Risk, and Routing Model]] §3.2 cited a nonexistent "§D" of the ECC Adoption
Map and overstated `database-reviewer`/`database-migrations` as "adapted" when both are
the map's own "deferred draft" status — fixed; (5) that same document's §2 said "eight
named domains" while §2.1 has always listed nine plus a catch-all, an off-by-one present
since the original pass and already propagated into the `planner` agent draft — fixed
both; (6) the Phase 5A master document's own self-audit claimed "fourteen new files" in
two places — an actual file count is seventeen (ten documents, seven agent drafts) — fixed
both, and this correction is itself recorded in the master document's self-audit section
precisely because a miscount inside a self-audit is the failure mode that section exists
to catch.
Correction to the prior Phase 5A log entry (this entry does not edit that entry, per the
vault's append-only rule for log.md — recorded here instead): that entry's "Important
findings" paragraph stated the seven-domain guardrail list "was verified to actually match
across all three documents rather than assumed consistent," and its boundary-compliance
paragraph cited "fourteen new artifacts." Both are now known inaccurate per findings (3)
and (6) above — the domain lists did not actually match, and the file count was wrong.
The historical entry is left unedited; this entry is the correction of record.
Synthetic evaluation: all eight tasks now dry-run traced (three were traced in the
original pass; Tasks 1, 2, 4, 5, 6 traced in this pass, per this task's own item 5). Three
further findings recorded, none folded silently into the routing model: two independent
traces (Tasks 8 and 4) exposed the same shape of gap — a routing row's chain assumes a
design question the process it routes to still calls open — generalized into one new
principle at [[Authority, Risk, and Routing Model]] §3.4 rather than patched per-row; one
trace (Task 2, Google OAuth login) found a genuine content gap in the Security Floor's
sixteen items (no coverage for OAuth-specific attack surface: redirect/callback and
CSRF-state validation, account-linking/identity-confusion) — left as a recorded finding
per the Security Floor process's own rule against quiet extensions, pending a future
owner-reviewed Floor extension; one trace (Task 6, a schema migration) confirmed the
already-known `database-reviewer` deferred-draft capability gap ([[ECC Adoption Map]]
§5.1) would bite on an ordinary, low-risk migration task, not only an exotic one.
Stale-assumption check (the task's own item 1): verified, not assumed, that none of the
four items the correction task specifically named as possibly stale — recurring bookings,
hosted-only-vs-self-hosting, the three owner decisions, any other resolved-but-shown-
UNKNOWN v1 scope item — have actually been resolved anywhere in the Second Brain since
Phase 5. Checked directly against [[../questions/Architecture Open Questions|Architecture
Open Questions]] (still lists exactly three blocking questions: hosted-only vs.
self-hosting, recurring bookings, flat vs. nested tenancy) and against this vault's own
`log.md` history. No staleness was found in Phase 5A's own artifacts regarding these four
items either — they were already correctly represented as open. Nothing was invented to
resolve them.
Artifacts updated (corrections only; no wholesale rewrite): [[Phase 5A — Engineering
Capability Foundation]], [[Authority, Risk, and Routing Model]], [[ECC Adoption Map]],
[[Learning Lifecycle and Promotion Policy]], [[Destructive-Operation Protection Design]],
[[Synthetic Evaluation Tasks]], `drafts/agents/planner.md`, `drafts/agents/tdd-guide.md`,
`Calquartz Booking Correctness — Operational Review Process.md`. This log entry.
Remaining unknowns: eleven (nine carried over unchanged from the original Phase 5A pass,
plus two new ones added this pass — whether §3.4's generalized principle holds against a
real task, and whether the Security Floor needs a 17th item for OAuth-specific surface).
Boundary compliance (all held): no ECC installed; no Calquartz repository created; no
Calquartz application code written; no continuous learning enabled; no production
infrastructure touched; no architecture alternative approved, selected, or assumed; no
recommendation silently turned into a decision; no owner decision invented or resolved by
assumption.
Next required action (owner, not Claude): none required by this pass specifically. The
same three owner decisions and independent-review sequence from the Architecture Decision
Brief still gate any move to implementation. Two new findings (routing-table refinement
at §3.4; the Security Floor's OAuth coverage gap) are recorded and available for a future
review pass whenever the owner wants to act on them — neither is blocking.

## [2026-09-08] phase | Phase 5A independent-review correction pass + Durable Decision Capture Policy established: COMPLETE
Scope: a second, independent review of Phase 5A (same day as the prior consistency
pass), owner-directed, covering five specific corrections plus establishment of a new
standing project rule. Not an architecture approval, not an ECC install, not the start of
Calquartz implementation.
Corrections made (four, all arithmetic/structural, none changing the architecture
recommendation or any owner decision): (1) [[Authority, Risk, and Routing Model]] §1 said
"six information sources" while its own table has always had seven rows (ranks 0-6) — an
off-by-one distinct from the earlier "seven"/"eight"/"fourteen" miscounts found in the
prior pass, caught only now because this was the first time the sentence was checked
against the table row-by-row rather than assumed correct — fixed. (2) The same document
used "architecture" as a §3.2 routing-table category without ever stating whether it was
a risk domain, a task class, or something else — genuinely ambiguous, not previously
resolved. Fixed by adding §2.4, which names three distinct things this project had been
calling "domain": risk domains (§2.1's fixed nine), task/routing classes (§3.2's left
column), and escalation triggers (the specific condition that forces a human gate).
Architecture is a routing class, explicitly NOT added to the nine risk domains. §3.2's
"Architecture change" row was split into "retrieval" (citing an already-answered
question — no human gate) and "new decision" (a genuinely open question — human gate,
always), preserving rather than weakening the principle that Claude never resolves a new
architectural question alone. (3) [[Learning Lifecycle and Promotion Policy]] §2's
nine-item domain list was re-verified item-by-item against
[[Authority, Risk, and Routing Model]] §2.1 (not re-asserted) — a table recording the
match is now in the document itself; one label was tightened ("Tenancy" → "Tenant
isolation") for exactness; an explicit line was added stating architecture is deliberately
absent from this list. (4) [[Synthetic Evaluation Tasks]] Dry-run trace 2 (Task 8)
originally wrote "domain = architecture" — corrected to "routing class = Architecture
change," and the trace's own "correction applied" note was updated to reflect that the
routing table now actually implements the retrieve-vs-decide split, rather than merely
carrying a marginal note about it (a status upgrade justified because the owner named the
same gap independently, which is different footing from one synthetic trace's opinion).
Final consistency sweep (fifth correction item): searched the whole engineering tree for
"six information sources," "seven/eight/nine/ten domains," "architecture" as a domain,
`risk_domains`, "guardrail domain(s)," "human gate," "owner decision," `UNKNOWN`, and
`DECISION`. Beyond the four fixes above, found and fixed three further propagations of
the same superseded counts into `README.md` (six-rank → seven-rank, twice; seven
guardrail domains → nine, once; "three dry-run traced" → "all eight now traced," twice) —
these were outside the `engineering/` folder strictly but are direct restatements of
Phase 5A facts in the project's own master-context document, in scope under the same
propagation discipline. No mislabeled `classification: DECISION` was found anywhere (every
Phase 5A artifact remains `RECOMMENDATION`, correctly, since nothing has been approved).
Every remaining "seven"/"eight"/"fourteen" occurrence found was historical wording
correctly preserved inside a dated correction note describing what the old, wrong number
used to say — none were left uncorrected as live claims.
New permanent rule established: [[Durable Decision Capture Policy]]
(`wiki/projects/001-calendar-os/engineering/Durable Decision Capture Policy.md`),
`classification: DECISION` (owner-directed, not Claude-proposed). Codifies: record an
owner decision in its one authoritative Second Brain home before the task that produced it
is complete; update Open Questions only once the decision is actually recorded; never
infer a DECISION from a casual preference, a standing RECOMMENDATION, or surrounding
context; when ambiguous, leave UNKNOWN and ask. Authoritative home: the policy document
itself (full text, one place). Discovery mechanism: a pointer (not a copy) in this
project's own `CLAUDE.md` (new §46, appended after the original 45 sections, which remain
unedited) and in `README.md`'s document index — both load automatically for any future
session working in this project, per this project's own directory-tree CLAUDE.md-loading
behavior, so the rule is discoverable without depending on this conversation.
No owner decision was made in this pass to apply the new policy to — the pass was
corrections and a process rule, not a scope decision. All three architecture-blocking
owner decisions (hosted-only vs. self-hosting, flat vs. nested tenancy, recurring
bookings) remain open, unchanged, per direct re-check against
[[Architecture Open Questions]].
Artifacts updated: [[Authority, Risk, and Routing Model]], [[Learning Lifecycle and Promotion Policy]],
[[Synthetic Evaluation Tasks]], `drafts/agents/architect.md`, `README.md`, `CLAUDE.md`
(project-level, §46 added). New artifact: [[Durable Decision Capture Policy]]. This log
entry.
Boundary compliance (all held): no ECC installed; no Calquartz repository created; no
Calquartz application code written; no continuous learning enabled; no production
infrastructure touched; no architecture alternative approved or selected; no architecture
recommendation changed; no OAuth item added to the Security Floor (left as an open finding
per its own governance rule); no owner decision invented or resolved by assumption.
Next required action (owner, not Claude): none required by this pass specifically — same
three owner decisions and independent-review/approval sequence from the Architecture
Decision Brief still gate implementation. The Durable Decision Capture Policy is now
active for any future session: the next time the owner makes a real decision (in this
project or elsewhere), that session is expected to follow it.

## [2026-09-08] phase | Phase 5B — Factual and Technical Prerequisites Before Architecture Approval: COMPLETE
Scope: close or materially reduce remaining factual/technical unknowns before independent
architecture review, per governing order FACTUAL VERIFICATION → OWNER DECISION →
TECHNICAL EVIDENCE → ARCHITECTURE REVIEW. Not an architecture implementation phase; no
architecture selected, approved, or frozen. Full record:
[[Phase 5B — Factual and Technical Prerequisites]].

**VERIFIED FACTS** (read-only SSH inspection, 45.58.59.114, 2026-09-08 ~15:30 UTC; exact
commands: `nproc`/`/proc/cpuinfo`, `free -h`, `swapon --show`, `df -h /`, `docker
--version`, `docker ps -a`, `docker images`, `docker volume ls`, `docker network ls`,
`docker system df`, `ss -tulpn`, `systemctl list-units --state=running`, `systemctl status
postgresql`, `which psql`, `docker ps -a --filter name=caddy`, `ls -la /etc/caddy`, `ls -la
/opt/`, `uptime`, `ufw status verbose`, `docker volume inspect
snagtime-production_caddy_data`): 2 CPU (Xeon E5-2680 v4, matches advertised); 7.6 GiB RAM,
570 MiB used, 7.0 GiB available; 15.6 GB swap, 0 used; disk 21% used (20 GB of 99 GB) —
**Phase 7's claimed post-decommission state independently re-verified, not merely
trusted, and confirmed holding with no drift** (Phase 6, one day earlier, measured 79%
used with 54 GB reclaimable build cache; that cache is now gone); Docker v29.8.0, 0
running containers, 1 image (`caddy:2-alpine`, preserved), 1 volume
(`snagtime-production_caddy_data`, preserved, created 2026-09-04 by the
`snagtime-production` compose project), only default networks, 0 build cache; only port 22
listening, UFW active (default-deny incoming, explicitly allows 22/80/443, nothing else);
no PostgreSQL installed (no systemd unit, no `psql` binary); no Caddy process running (no
`/etc/caddy`); no legacy `/opt/calquartz`-style tree; load average 0.08 — machine fully
idle. Applied per Durable Decision Capture Policy: corrected the Architecture Decision
Brief's stale §3 row 2 ("actual utilisation UNKNOWN, inventory never executed" — false
twice over, since Phase 6 ran it and this session re-ran it) and appended a new dated
update section to Infrastructure Constraint Analysis rather than editing its existing
2026-09-07 update.

**OWNER DECISIONS** (obtained directly this session via explicit question, using this
phase's own required framing; full text of the owner's answers recorded in
[[Architecture Open Questions]]'s Phase 5B resolution section, not only summarized here):
(1) **Hosted-only** — self-hosting is not a v1 requirement and must not add v1
requirements/complexity/support burden/security surface/testing obligations; a
portability *preference* for future optionality, not a v1 self-hosting *requirement*. (2)
**Recurring bookings: YES, in v1** — reverses the working assumption carried since Phase 3
("not in v1"); scope only, no implementation model selected. (3) **Flat tenancy** — no
org-of-orgs hierarchy in v1. All three: classified `DECISION`, authoritative home
[[Architecture Open Questions]], Decision Closure Test's eleven checkboxes verified for
each (see Phase 5B document §B). **Blocking owner decisions: 0** (down from 3). This does
**not** mean architecture is approved — Decision Brief §18 checklist items 5–10 remain
outstanding (owner reconfirmation of scope/constraints, security-floor/licensing
acceptance, independent review, the ADR itself).

**Propagation**: every document asserting "three still-open owner decisions" or an
equivalent present-tense claim was updated —
[[Calquartz — Architecture Decision Brief]] (§1, §2, §13, §17, §18),
[[Architecture Requirements and Constraints]] (§1 items 2/7/8, §4, §5),
[[Calquartz Booking Correctness — Operational Review Process]] (§7 and its summary
table), [[ECC Adoption Map]], `drafts/agents/architect.md`, `drafts/agents/planner.md`,
[[Synthetic Evaluation Tasks]], [[Durable Decision Capture Policy]],
[[Phase 5A — Engineering Capability Foundation]], [[Failure-Mode Analysis]] (the recurring-
bookings scenario row), and the project `README.md`/`CLAUDE.md` (new §47 pointer). Text
describing the pre-decision state was left unedited with an appended forward-note where it
was itself a historical record (e.g. the independent-review pass's own report, written
earlier the same day) rather than rewritten.

**TECHNICAL EVIDENCE** (sharpening passes over existing forensic evidence, not executed
code spikes — no stack exists to execute against; the four original Pre-Approval spikes
1–4 remain genuinely un-executed): tenant-isolation backstop — fourteen sub-questions
answered against existing evidence in
[[Database-Level Tenant Isolation — RLS and Alternatives]] and
[[Multi-Tenancy Trust Model]]; two remain genuinely UNKNOWN (pooling-compatibility
performance, stack compatibility), unchanged, correctly deferred to Pre-Approval spike 1.
Double-booking — booking semantics enumerated against Failure-Mode Analysis and Booking
Correctness §1–9; one new UNKNOWN identified as a direct consequence of the recurring-
bookings decision (per-occurrence uniqueness within a series), flagged as new required
scope for Pre-Approval spike 2, not designed. Timezone/DST — spike 3's hypothesis restated
unchanged (still unexecuted); two new evidence gaps identified (recurring-series DST
correctness; DST-boundary notification timing) that did not exist as named gaps before
this session. **No mechanism was selected in any of the three areas.**

**SECURITY FLOOR**: unchanged at sixteen items. OAuth finding (from the prior
independent-review pass) remains an open finding, not silently added as item 17, per that
process document's own governance rule.

**DURABLE DECISION CAPTURE — first operational test**: passed. A verified fact, an owner
decision, and a still-open UNKNOWN are each independently discoverable from their
authoritative Second Brain home without this conversation, per [[Phase 5B — Factual and
Technical Prerequisites]] §G.

**Remaining UNKNOWNs**: two non-blocking owner decisions (seats, round-robin) plus product
naming; four technical unknowns requiring the original Pre-Approval spikes; three
stack-dependent unknowns; four newly-identified evidence gaps (tenant/host/invitee/event
timezone modeling, DST notification timing, recurring-series DST correctness, the
pre-existing OAuth Security Floor gap). Full list: [[Phase 5B — Factual and Technical
Prerequisites]] §H.

**Architecture readiness: NO.** All three blocking owner decisions are resolved, but
Decision Brief §18 checklist items 5–10 (owner reconfirmation, independent review, the
ADR) have not happened. Full reasoning: [[Phase 5B — Factual and Technical Prerequisites]]
§I.

Boundary compliance (all held): no ECC installed; `~/.claude` untouched; no Calquartz
repository created; no Calquartz application code written; no production deployment; no
production application changes (the only production-adjacent action was read-only SSH
inspection — no write, restart, install, or config change); no architecture approved,
selected, or frozen; no destructive operation performed.
Next required action (owner, not Claude): reconfirm v1 scope/constraints and accept the
security floor/licensing position (Decision Brief §18 items 5–8); decide whether to send
the brief for independent adversarial review (item 9, recommended); then record the actual
architecture selection (item 10). None of this is performed by Claude autonomously per
this project's own architecture-approval boundary.

## [2026-09-08] phase | Phase 6 — Independent Adversarial Architecture Review: COMPLETE
Scope: the first genuine attempt to answer which architecture, if any, Calquartz should
approve for v1, run explicitly as an adversarial review instructed to try to break the
standing A/B1 recommendation rather than confirm it by default, against the three
product-scope decisions Phase 5B obtained (hosted-only, flat tenancy, recurring bookings
in v1) and the independently re-verified VPS state. Full record:
[[Phase 6 — Independent Adversarial Architecture Review]].

**Independent verdict: CONFIRMED.** A/B1 (the modular monolith) survives adversarial
examination. Eight distinct attack lines were run (full detail in the review document
§3); none forces a change to any of A/B1's five defining properties (one deployable, one
PostgreSQL, disciplined module boundaries, durable Postgres-backed job table with one
worker, a database-level tenant-isolation backstop candidate). Two attack lines produced
real findings, both scope clarifications rather than architecture changes: recurring
bookings requires a series/occurrence data model within the existing booking module
(no new deployable/datastore/persistence-paradigm needed), and introduces a previously-
unanticipated scheduled cross-tenant batch-job shape (occurrence materialization) that
widens [[Calquartz — Pre-Approval Technical Spikes]] spikes 2 and 4's required scope —
recorded as additions to those spikes' specifications, not new spikes.

**Recommendation, superseding one prior claim rather than deleting it**: the existing
"recurring bookings pulls modestly toward event-sourced Alternative D" framing (in
[[Calquartz — Architecture Decision Brief]] §16, [[Calquartz Booking Correctness — Operational Review Process]] §7, and [[Architecture Alternatives]]) does not survive scrutiny
and likely reverses — recurring bookings being *in scope* is not the same condition as
needing *full historical replay*, a plain series/occurrence CRUD model satisfies the
actual product requirement (partial-success as an explicit outcome) at a fraction of D's
cost, and neither reference repository offers an event-sourced recurring-booking
precedent, meaning D's true cost for this feature was likely under-priced by the existing
8–14 engineer-day estimate, which is CRUD-shaped. This correction is recorded in the
review document and cross-referenced from every document that stated the original
framing; none of those documents' original text was deleted.

Also found: the hosted-only decision directly forecloses Alternative C's clearest future
trigger (a confirmed self-hosting requirement) — C's case is measurably weaker after
2026-09-08 than before, not merely unchanged; flat tenancy narrows (does not resolve) the
risk surface of the still-open tenant-isolation backstop spike; flat tenancy has no
differential effect on the four alternatives (checked, not assumed); Security Floor item
15's boot-config-validation control is now known to defend a single known operator, not
an unknown third party, per the hosted-only decision — easier to verify, not less
required. "Insufficient evidence" was considered and rejected as a verdict: the remaining
unknowns are implementation-level (tenant-isolation mechanism, booking-constraint shape,
DST behavior, job semantics), not architecture-level, per this phase's own explicit
architecture/implementation boundary.

**Security review**: architectural security capability (all four alternatives can
structurally satisfy all 16 Security Floor items) kept separate from implementation
verification (impossible before code exists — explicitly, no claim is made that any
Security Floor item has passed). No Floor item count changed; the pre-existing OAuth
coverage gap remains an open finding, not reintroduced as a decision.

**This remains a RECOMMENDATION, not a DECISION.** The independent review's own verdict,
and the resulting recommendation, are stated repeatedly throughout the review document as
non-binding. **Owner approval has not been requested or obtained during this phase** —
the exact approval question is recorded in the review document §10, unanswered.

Major remaining UNKNOWNs: two non-blocking owner decisions (seats, round-robin); four
technical unknowns requiring the still-unexecuted Pre-Approval spikes (1–4, two now with
widened scope); three stack-dependent unknowns; all explicitly implementation-level
per §16 of this phase's own governing instruction, not blocking for architecture
approval.

Artifacts updated (new content appended or corrections made; no historical claim
deleted): new document [[Phase 6 — Independent Adversarial Architecture Review]];
[[Architecture Recommendation]] (pointer + one row annotated, not deleted);
[[Architecture Decision Matrix]] (pointer, no cell changed); [[Calquartz — Architecture Decision Brief]] (§1 update note); [[Calquartz — Pre-Approval Technical Spikes]] (spikes 2
and 4 scope-widened); project `README.md`, `CLAUDE.md` (§47), root `index.md`. This log
entry.

Boundary compliance (all held): no Calquartz repository created; no application code
written; no ECC installed; continuous learning disabled; `~/.claude` untouched; no
production changes (document analysis only, no VPS access this phase); no destructive
operations; no architecture silently converted into DECISION — the RECOMMENDATION/
DECISION distinction is stated explicitly and repeatedly throughout the review document,
not only in a single disclaimer.
Next required action (owner, not Claude): answer the exact approval question in
[[Phase 6 — Independent Adversarial Architecture Review]] §10 — approve or reject A/B1 as
the Calquartz v1 architecture. Only after an explicit answer does the Durable Decision
Capture Policy apply to convert this recommendation into a recorded DECISION and permit an
ADR to be drafted. Nothing in this phase requests or implies that implementation should
now begin.

## [2026-09-08] phase | Phase 6 correction pass — governance wording tightened before owner approval: COMPLETE
Scope: a narrowly-scoped documentation correction pass, requested before asking the owner
to approve A/B1, to close two governance ambiguities the independent review's own
language could invite: (1) "all four alternatives clear the security gate" being
misreadable as a Security Floor pass, and (2) an eventual owner "yes" being misreadable as
approving unverified implementation detail. Not a reopening of the architecture review;
no new research, no spikes run, no repository/ECC/VPS touched.
Security Floor wording: [[Calquartz — Architecture Decision Brief]] §8's "Gate result"
sentence, §4's "all four clear the security gate" line, and §16's recommendation
paragraph were each amended in place with an explicit clarification (preserved, not
deleted) stating this is an **architectural capability assessment**, not a Security Floor
pass, and that no Calquartz implementation has passed the Floor because no Calquartz code
exists. The same clarifying pointer was added to [[Architecture Decision Matrix]]'s phase
boundary and [[Architecture Recommendation]]'s two matching sentences (§1 and "What this
recommendation does not claim"). **No Security Floor item was added, removed, or
reworded** — still sixteen items; the OAuth gap remains an open finding, not a
seventeenth item.
Owner approval language: [[architecture/Phase 6 — Independent Adversarial Architecture Review]] §10 gained an explicit acceptance statement ("I approve A/B1... subject to the
documented... gates... I understand... failure of an implementation gate may require
revisiting an architectural assumption rather than silently working around the failure")
and an explicit current-status line, **RECOMMENDATION — OWNER APPROVAL OUTSTANDING**,
also promoted into the document's own frontmatter `status` field. [[Calquartz — Architecture Decision Brief]] §18 checklist items 9 (now marked done, pointing to the Phase 6
review) and 10 (now carrying the same explicit non-implementation-approval framing) were
updated to match. **A/B1 remains classified RECOMMENDATION; no DECISION was recorded; the
owner was not asked to approve during this pass.**
Technical-spike boundary: added an explicit "reopen, don't route around" condition for
each of the four still-unexecuted spikes (tenant-isolation backstop, booking/occupancy
constraint, DST behavior, durable job semantics — the last two Phase 6-widened for
recurring bookings) to the Phase 6 review §8 and a pointer in
[[Calquartz — Pre-Approval Technical Spikes]]'s boundary section: none currently blocks
A/B1's approval, and this pass does not reverse that, but a spike result proving one of
A/B1's five named defining properties cannot be satisfied is grounds to reopen the
architecture, not to silently implement around the gap. This does not convert any of the
four into a new architecture decision.
Consistency sweep: searched the full project for the ambiguous "clear the security gate"/
"clears all sixteen" phrasing (found and fixed in the three files above; none elsewhere)
and for present-tense Security-Floor-pass or implementation-approval language in
`README.md`, `CLAUDE.md`, root `index.md`, and [[questions/Architecture Open Questions]]
— none found needing correction; all already correctly frame the architecture as
unapproved and the Floor as unexercised against real code. No historical Phase 3/4/5
reasoning was rewritten — every correction in this pass amends currently-authoritative,
actively-maintained documents in place with a dated clarification, consistent with how
prior passes this project handled the same class of ambiguity (the "seven"/"eight"/
"six information sources" corrections earlier the same day).
Artifacts updated: [[Calquartz — Architecture Decision Brief]] (§4, §8, §16, §18),
[[Architecture Decision Matrix]], [[Architecture Recommendation]] (§1 area, "What this
recommendation does not claim"), [[architecture/Phase 6 — Independent Adversarial Architecture Review]] (§8, §10, frontmatter status), [[Calquartz — Pre-Approval Technical Spikes]] (boundary section). This log entry.
Boundary compliance (all held): no repository created; no application code written; no
ADR created; no ECC installed; continuous learning disabled; `~/.claude` untouched; no
VPS/production changes; no technical spikes executed; no architecture approval recorded;
no RECOMMENDATION or UNKNOWN silently converted to DECISION.
Next required action (owner, not Claude): unchanged from the prior entry — answer the
approval question in [[architecture/Phase 6 — Independent Adversarial Architecture Review]] §10. This pass does not ask that question in this task; it only tightens the wording
the owner will read when they answer it.

## [2026-09-08] phase | Architecture decision: A/B1 approved — ADR-001 recorded: COMPLETE
**What was decided**: the owner approved A/B1 (the modular monolith) as Calquartz's v1
architecture, using substantially the acceptance statement drafted in
[[architecture/Phase 6 — Independent Adversarial Architecture Review]] §10 (given by
voice, transcribed with some noise, reconstructed and confirmed against that statement —
see [[decisions/ADR-001 — A-B1 Modular Monolith Architecture]] for the exact text
recorded), and explicitly affirmed the condition that technical-spike or implementation
evidence disproving an A/B1 defining property reopens the architecture rather than being
silently worked around.
**When**: 2026-09-08, in conversation.
**Who/what authority**: the project owner, explicitly, per
[[engineering/Authority, Risk, and Routing Model]] rank 0 — a live instruction meant to
persist, written into the Second Brain at rank 1 per that document's own rule.
**What prior UNKNOWN/open question it resolves**: [[Calquartz — Architecture Decision Brief]] §18 checklist item 10 ("record the decision — select one of A/B1, B2, C, D, or
reject all four") — the single remaining item blocking architecture approval after Phase
5B closed the three product-scope decisions and Phase 6's independent review confirmed
the recommendation. Not a re-decision of anything already settled; the first and only
architecture-selection decision this project has made.
**Authoritative home**: [[decisions/ADR-001 — A-B1 Modular Monolith Architecture]] —
new file, new `decisions/` folder (first in this project, per this vault's own convention
that decisions get their own folder once a project accumulates real ones). Classified
`DECISION`. Contains: selected option, rejected alternatives (B2/C/D, each deferred to
named triggers, not rejected in principle), explicit non-decisions, binding constraints,
four reopen triggers, and the implementation gates approval does not assert are
satisfied — all per the owner's own explicit request for this structure.
**Durable Decision Capture Policy checklist, applied**: owner's statement treated as
authoritative ✓ · canonical home identified and populated before this task is complete ✓
· dependent context updated (below) ✓ · classification preserved (`DECISION`, not
silently inferred — the owner's own words were explicit, not a casual preference) ✓ ·
this log entry ✓ · discovery pointer added to `CLAUDE.md` (below) ✓.
**Propagation**: [[Calquartz — Architecture Decision Brief]] (§1 status heading and table,
§17, §18 items 10–13); [[architecture/Architecture Recommendation]] (top banner, closing
disclaimer); [[architecture/Phase 6 — Independent Adversarial Architecture Review]]
(frontmatter status, intro banner, §9, §10, §11); [[questions/Architecture Open Questions]] (new closing section); project `README.md` (status line, new "Architecture
decision" document-index section, Next steps); project `CLAUDE.md` (§47 pointer list, ADR
now listed first); root `index.md`. Every prior-state claim ("no architecture has been
approved," "this question has not been answered," "NOT YET CREATED") was preserved as the
historical record of what was true before this decision, with an explicit note that it no
longer describes the current state — none was silently rewritten to read as if the
decision had always been known.
Boundary compliance (all held): no Calquartz repository created; no application code
written; no ECC installed; continuous learning disabled; `~/.claude` untouched; no VPS or
production infrastructure touched; no technical spike executed; no implementation
authorized or begun by this decision or by ADR-001 — approval selects the architecture,
not an implementation stack, and is not an assertion that any implementation is safe or
complete.
Next required action (owner, not Claude): none required to keep the project moving — the
architecture is now approved. When implementation planning actually begins, it starts
with the four still-unexecuted Pre-Approval spikes (1–4), each carrying an explicit
reopen condition recorded in ADR-001. Nothing in this entry requests, authorizes, or
implies that implementation, repository creation, or ECC installation should now begin.

## [2026-09-08] phase | Phase 7 — Execute Technical Spikes 1–4: COMPLETE
Scope: execute the four queued Pre-Approval Technical Spikes (tenant isolation, booking/
occupancy invariant, DST, durable jobs) as engineering investigation, per explicit task
instruction, to turn remaining high-risk unknowns into evidence-backed implementation
constraints. Not application implementation. Full spike code, raw output, and durable
evidence records: `4. Calquartz Spikes/` (outside this vault, alongside the reference-repo
clones and the ECC evaluation clone, per this project's existing convention for non-wiki
material) — `evidence/spike-{1,2,3,4}-*-evidence.md` are the authoritative per-spike
records this entry summarizes.

**Environment**: no PostgreSQL or Docker existed on the development machine; the
production VPS was correctly treated as off-limits per the task's own boundary (and has
no PostgreSQL installed regardless, per Phase 5B's findings). The owner was asked how to
proceed (a genuine environment gap, not a design choice) and chose a disposable local
install. EnterpriseDB's own binary CDN returned HTTP 403 from this environment's network
egress; PostgreSQL 17.11's official Windows build was instead obtained via Maven Central
(`io.zonky.test.postgres:embedded-postgres-binaries-windows-amd64:17.11.0`, a
redistribution of the same upstream binaries), run as a standalone process on
`127.0.0.1:5599` only (never exposed to the network), with a dedicated `calquartz_spike`
database and role namespace, no Calquartz credentials/schema/data anywhere in it. Stopped
and **fully deleted** after evidence capture, per the owner's own instruction — only the
spike code and durable evidence records persist.

**Spike 3 (DST) — hypothesis CONFIRMED.** Real Luxon execution (no PostgreSQL needed):
`startOf('day').plus({minutes})` misplaces the first slot by exactly ±1 hour on every
tested transition (`America/New_York`, `Europe/London`, both directions), and the drift
propagates through the entire day's schedule. New: in a recurring series, only the
occurrence landing exactly on the transition date is affected. New: Luxon silently
resolves nonexistent local times (shifts forward) and silently picks one interpretation
of ambiguous local times — an open product question this spike surfaces, does not
resolve. Two correct constructions demonstrated. Evidence:
`evidence/spike-3-dst-evidence.md`.

**Spike 2 (booking/occupancy invariant) — three candidates tested under real 20-way
concurrent load.** The exact-tuple key empirically reproduces the Cal.diy-shaped defect
(admits overlapping-but-non-identical bookings, no buffer support) — disqualified. Both
the occupancy-row and exclusion-constraint candidates pass every correctness, buffer,
cancel/reschedule, and — newly required — recurring-series test (atomic all-or-nothing
creation, per-occurrence independent creation with correct partial-failure isolation,
concurrent overlapping-series creation). New finding: both atomicity models the Booking
Correctness process left open turn out to be an application-transaction-boundary choice,
not a constraint-mechanism property. Recommended: the exclusion constraint, for
dramatically lower write amplification (1 row vs. up to thousands per booking). A
methodological bug in the spike's own first draft (a same-row CAS test that conflated
safe sequential updates with an actual slot-conflict race) was found and corrected before
being reported, not left in. Evidence: `evidence/spike-2-booking-invariant-evidence.md`.

**Spike 1 (tenant isolation) — RLS confirmed leak-free under real pooled connection
reuse.** 60 interleaved tenant requests over a 3-connection pool, zero leaks. The
deliberate misuse of plain `SET` (not `SET LOCAL`) was reproduced to confirm the failure
mode is real, not hypothetical. An unenrolled table with RLS-but-no-policy was correctly
denied; a second, unenrolled table with RLS never enabled reproduced the exact
SnagTime-shaped cross-tenant exposure. A catalog-driven mechanical enrollment check
(`pg_class.relrowsecurity`) caught both the deliberately-planted test case **and an
unplanned real omission in the spike's own schema** (the `tenants` table) — stronger
evidence for the recommendation than the planted case alone. Worker role confirmed
narrower than the web role; admin bypass confirmed explicit and separate. Two test-harness
bugs (a shared-transaction capability check that produced false negatives after the first
denial; cross-test connection-pool contamination) were found, understood, and fixed before
reporting — recorded, not hidden. Evidence:
`evidence/spike-1-tenant-isolation-evidence.md`.

**Spike 4 (durable jobs) — all four original failpoints and the new recurring-batch
shape confirmed.** Concurrent claim race: exactly 1 winner of 10. Crash-after-claim:
correctly reclaimed after real lease expiry, fencing token advanced, effect applied
exactly once via an idempotent effect log. Stale-effect: a fenced-out worker's old token
correctly detectable as stale. Graceful shutdown: released without consuming retry
budget. Retry exhaustion: reached `dead` at exactly `max_attempts`. No starvation of an
unrelated job by a long-held claim. **New, mandatory extension**: the same protocol,
unmodified, correctly handles the scheduled cross-tenant occurrence-materialization batch
shape — many-tenant/many-series processing, mid-batch crash-and-resume-from-checkpoint
(dependent on committing progress incrementally, not only at completion), duplicate-
batch-execution prevention, and non-starvation of ordinary jobs alongside a large batch.
Evidence: `evidence/spike-4-durable-jobs-evidence.md`.

**Architecture status: A/B1 remains approved, unchanged.** None of the four spikes
demonstrated that an A/B1 defining property cannot be satisfied — **ADR-001 was not
reopened.** Every spike produced implementation constraints and recommendations (exact
mechanism selections remain explicit ADR-001 non-decisions), consistent with the phase's
own instruction not to let implementation difficulty alone trigger a reopen.

Propagation: [[Calquartz — Pre-Approval Technical Spikes]] (each spike's own Result
section added, headers marked EXECUTED, Sequencing/Boundary sections updated),
[[Calquartz Booking Correctness — Operational Review Process]] (§1/§2/§5/§6/§8 and the
summary table), [[Database-Level Tenant Isolation — RLS and Alternatives]] (its own
recommended-spike section updated with the result), [[decisions/ADR-001 — A-B1 Modular Monolith Architecture]] (reopen-triggers and "what happens next" sections noted as
satisfied, decision itself unchanged), project `README.md`, `CLAUDE.md` (§47), root
`index.md`. No historical Phase 3–6 reasoning was rewritten; every update is additive or
explicitly marked as superseding a specific prior claim.

Boundary compliance (all held): no Calquartz application code created; no production
repository created; no production deployment; no VPS/production changes (the only
network/infrastructure action was installing and later fully removing a disposable local
PostgreSQL instance on the development machine itself, with explicit owner authorization
obtained first); no ECC installed; continuous learning disabled; `~/.claude` untouched; no
architecture silently changed; ADR-001 modified only to add pointer notes to now-available
spike evidence, its actual decision (A/B1, approved) unchanged.
Next required action (owner, not Claude): none required to keep the project moving.
Implementation planning is the next named step (Decision Brief §18 item 13) but is not
begun, authorized, or implied by this phase.

## [2026-09-08] phase | Phase 8A — Implementation Stack Selection: COMPLETE (RECOMMENDATION, not a decision)

Scope: select the implementation stack for Calquartz v1 (backend/runtime, PostgreSQL
access layer, date/time library, durable-job implementation, frontend, auth/authz
approach, testing stack, deployment shape) against actual requirements and Phase 7 spike
evidence, per explicit task instruction. Stack-selection and engineering-foundation
planning only — no Calquartz code written, no repository created.

**Phase 7 closure check (bounded, per task instruction)**: all four spikes confirmed
recorded as executed with correctly classified results (FACT/OBSERVATION stayed empirical,
RECOMMENDATION stayed a recommendation, no spike silently became a DECISION); ADR-001
confirmed unchanged as approved direction; no stale doc found claiming spikes unexecuted;
disposable PostgreSQL environment confirmed gone; no VPS/production changes; no ECC
installation. One genuine, minor propagation gap found: spike implementation constraints
had not yet been cross-referenced into Architecture Requirements and Constraints. Fixed
additively (new §6, dated and attributed to Phase 8A) — no prior content in that document
rewritten, no spike work reopened.

**Stack RECOMMENDATION (owner approval required, NOT a decision)**: TypeScript/Node.js,
Fastify, `pg` + Kysely (raw SQL for RLS/exclusion-constraint/job-table DDL — not Prisma,
whose ORM abstraction was found to fight three of the four spike-validated mechanisms),
node-pg-migrate migrations; Luxon for date/time, reusing spike 3's validated
wall-clock-correct construction; a hand-rolled Postgres job table implementing spike 4's
exact claim/lease/fence/`SKIP LOCKED`/incremental-checkpoint protocol (not an adopted
job library, since none was spike-tested against the fencing/checkpoint requirements);
Next.js (React) frontend sharing TypeScript types with the backend; session-based
cookie auth with `users`/`memberships` kept schematically separate (specific library
deferred); Vitest + real-PostgreSQL integration tests + Playwright E2E, built as
descendants of the spike 1/2/4 test harnesses; single Docker image, web+worker processes,
on the existing single VPS, no Redis, no Kubernetes, no additional infrastructure.
Includes an explicit final adversarial self-check (strongest argument against the
recommendation, weakest-handled requirement, dependency liability risk, ORM/date-library/
job-guarantee failure points, reopen triggers, missing evidence).

Pages created: [[wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack Selection|Phase 8A — Implementation Stack Selection]].
Pages updated: [[wiki/projects/001-calendar-os/questions/Architecture Open Questions|Architecture Open Questions]]
(added a Phase 8A cross-reference note; no product-scope question closed — stack selection
is an engineering decision, not a product-scope one), [[wiki/projects/001-calendar-os/architecture/Architecture Requirements and Constraints|Architecture Requirements and
Constraints]] (additive §6, Phase 7 spike constraints + Phase 8A stack-selection pointer).

Boundary compliance (all held): no Calquartz application code created; no production
repository created; no ECC installed; continuous learning disabled; `~/.claude` untouched;
no VPS/production changes; ADR-001 not modified, not converted from architecture decision
into implementation authorization; the stack selection above is explicitly labeled
RECOMMENDATION — OWNER APPROVAL REQUIRED throughout, never DECISION.

Remaining UNKNOWNs (explicit, not resolved by this phase): auth library selection;
nonexistent/ambiguous local-time product policy; recurring-series atomicity model
(all-or-nothing vs. partial success — an application choice per spike 2, not fixed by this
stack); whether Prisma's RLS/transaction friction would actually manifest in Calquartz's
specific query patterns (inferred from general ecosystem reports, not Calquartz-tested);
whether a job library could later replace the hand-rolled protocol; real load/performance
characteristics on the actual VPS (untested).

Next required action (owner): review and, if satisfactory, explicitly approve the Phase
8A stack recommendation, per the Durable Decision Capture Policy — approval is not implied
by this phase and must be given explicitly before any implementation planning begins.

## [2026-09-08] phase | Phase 8A — Adversarial Correction Pass: COMPLETE

Scope: targeted adversarial review and correction of the existing Phase 8A stack-selection
document — not a restart, not a Phase 8B, no code, no repository, no VPS/production
changes. Reviewed the Phase 8A document in full against ADR-001, Architecture Open
Questions, Architecture Requirements and Constraints, the Security Floor, the
Multi-Tenancy Trust Model, the Failure-Mode Analysis, the Threat Model, the Durable
Decision Capture Policy, and the four Phase 7 spike evidence files under
`4. Calquartz Spikes/evidence/`.

**Corrected**: (1) tightened the Prisma reassessment so the RLS/`SET LOCAL` friction claim
is explicitly classified as general-ecosystem-documentation-plus-INFERENCE about
Calquartz's patterns, not Calquartz-specific evidence — the original wording ("not a
rumour") read as stronger than the evidence supports. (2) Separated the durable-job
PROTOCOL (spike-4-validated FACT) from the IMPLEMENTATION MODEL (application permanently
owning the full job framework — a weaker RECOMMENDATION resting on inference, not
spike-validated as uniquely correct) from ALTERNATIVE LIBRARIES (not spike-tested, not
introduced). (3) Made the Next.js/Fastify justification explicit via an adversarial
A/B/C comparison (Next.js alone vs. Next.js+Fastify vs. separate frontend+Fastify),
concluding the combination remains the strongest candidate but for a stated,
non-manufactured reason (shared application/domain layer between web and worker
processes) rather than an unexamined default.

**Verified unchanged (no correction needed)**: the Luxon/timezone reasoning already rested
on spike 3's own evidence, not on SnagTime precedent — checked explicitly, not found. The
Security Floor status was already correctly stated as all-sixteen-UNKNOWN, no
Calquartz-code-exists claim contradicted. The auth-library deferral was already
intentional and correctly scoped; this pass additionally states explicitly that concrete
library selection belongs to a future Phase 8B, not this document. The recurring-booking
sanity check passed — no claim anywhere states recurring bookings are "solved." The
deployment/capacity distinction was already correctly drawn (architectural fit vs. no load
testing performed). The original candidate set was found adequate — no materially
different candidate would plausibly change the recommendation, and none was added.

**Recommendation status: did not change.** No stack element in the original §18 was
added, removed, or swapped. What changed is classification precision on two points (the
Prisma downgrade's evidentiary basis, and the durable-job implementation model's
weaker-than-implied justification), both now stated explicitly rather than left implicit.

**UNKNOWNs**: unchanged in substance from the original phase's §20; this pass adds explicit
reopen triggers (§17 of the correction pass) tied to future implementation-time evidence,
and separates the job-model UNKNOWN into protocol/implementation-model/library-alternative
components for clarity. No new UNKNOWN was resolved; none was newly discovered beyond
sharper phrasing of what §20 already named.

**Owner decision appropriateness**: no owner decision is made or newly ripe by this pass.
The stack remains RECOMMENDATION — OWNER APPROVAL REQUIRED, exactly as before. Architecture
Open Questions and Architecture Requirements and Constraints were reviewed and found not to
require material additive changes — the original Phase 8A propagation into those documents
already correctly scoped stack-selection as an engineering decision, not a product-scope
one, so neither was touched in this pass.

Pages updated: [[wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack Selection|Phase 8A — Implementation Stack Selection]]
(new dated `## Adversarial Correction Pass (2026-09-08)` section appended; frontmatter
`updated:` field annotated; no prior section deleted or rewritten).

Boundary compliance (all held): no Calquartz application code written; no application
repository created; no VPS/production changes; no ECC installed; no `.claude/agents` or
`.claude/skills` created; continuous-learning-v2 not enabled; `~/.claude` untouched;
ADR-001 not modified; no stack element converted to DECISION; no second Phase 8A document
created; Phase 8B not begun.

Next required action (owner): unchanged — review and, if satisfactory, explicitly approve
the Phase 8A stack recommendation (as corrected by this pass) per the Durable Decision
Capture Policy.

## [2026-09-08] phase | Phase 8B — Minimum ECC Engineering Foundation: COMPLETE
Date: 2026-09-08
Phase: Phase 8B — Minimum ECC Engineering Foundation (Project #001, Calendar OS)
Status: Complete
Part 0 — decision capture: **the owner explicitly approved the Phase 8A stack
recommendation, in chat, as a DECISION**, scoped exactly to the §18 stack-element list
(TypeScript/Node.js, Fastify, `pg`+Kysely, `node-pg-migrate`, Luxon, PostgreSQL-backed
durable jobs per the Phase 7 protocol, Next.js/React, session-cookie auth architecture
with library deferred, Vitest+real-PostgreSQL+Playwright, single Docker image/web+worker/
no Redis/no Kubernetes) — not a wider approval of any implementation subdetail. Recorded
per the Durable Decision Capture Policy: [[wiki/projects/001-calendar-os/engineering/Phase
8A — Implementation Stack Selection|Phase 8A]]'s frontmatter changed RECOMMENDATION→
DECISION (scoped) and a dated "Owner Approval (2026-09-08, Phase 8B)" section was
appended (no §1–23 substantive text altered); [[wiki/projects/001-calendar-os/questions/
Architecture Open Questions|Architecture Open Questions]] received a short appended
cross-reference note. Explicitly **not** upgraded to DECISION by this action (per direct
instruction): the concrete auth library; the ambiguous/nonexistent local-time policy; the
recurring-series atomicity model; the exact tenant-isolation, exclusion-constraint, and
durable-job implementations; whether a job library could replace the app-owned protocol;
real VPS load/performance; seats/round-robin; calendar/notification providers; a public
developer API; embeds; the canonical product name — all remain explicitly UNKNOWN.
ADR-001 was not modified and remains the sole architectural DECISION of record.

Part 1 — ECC installation: found that [[wiki/projects/001-calendar-os/engineering/Phase
5A — Engineering Capability Foundation|Phase 5A]] had already designed nearly the entire
generic-plus-Calquartz-specific ECC foundation (authority hierarchy, risk/routing model,
verification format, learning-lifecycle policy, and seven full agent drafts) as staged
text — none of it previously installed anywhere. This phase installed, rather than
redesigned, that material: copied the seven staged agent drafts (`planner`, `architect`,
`tdd-guide`, `code-reviewer`, `security-reviewer`, `build-error-resolver`,
`refactor-cleaner`) verbatim into `.claude/agents/` at the workspace root (Claude Code's
real, current installation mechanism — no Calquartz repository was created; none was
required), updating only each file's status line and, for `build-error-resolver`, filling
in the previously stack-pending build/typecheck/lint/test commands now that the stack is
approved. No skill, workflow, or hook was installed — every candidate (`database-reviewer`,
`database-migrations`, `security-scan`/AgentShield, a destructive-operation hook, a
workspace-root `CLAUDE.md`) was evaluated against the adversarial-minimization test and
found premature, speculative, or scope-contaminating, and deliberately deferred/rejected
with reasoning recorded.
Artifacts created: [[wiki/projects/001-calendar-os/engineering/Phase 8B — Minimum ECC
Engineering Foundation|Phase 8B — Minimum ECC Engineering Foundation.md]] (24-section
required artifact: purpose, scope, boundaries, existing-state findings, generic/
Calquartz-specific capability split, routing/risk/verification models, the five hard
gates re-affirmed unchanged, authority hierarchy, human gates, skill/agent/workflow
inventories, automatic-routing behavior, context discipline, failure containment, learning
boundary, exact installation changes with verification evidence, rejected/omitted
capabilities with reasoning, remaining UNKNOWNs, a 10-task synthetic routing evaluation,
and an adversarial review); seven files under `.claude/agents/` (workspace root, outside
the vault).
Artifacts updated: `Phase 8A — Implementation Stack Selection.md`, `Architecture Open
Questions.md` (Part 0, above); this log entry.
Synthetic evaluation result: 10 hypothetical tasks (README wording; user-profile field;
booking-conflict logic; recurring-booking generation; timezone conversion; tenant-
membership authorization; worker retry behavior; a database migration; an external
webhook; a security header) were hand-traced against the installed routing model. All 10
received a risk tier and routing chain of the correct order of magnitude — no LOW task
triggered a multi-agent chain, no HIGH task fell back to the LOW baseline — and no routing
step claimed a gate had passed merely because a specialist agent exists or was named. One
genuine, correctly-surfaced (not hidden) capability gap was found: task 8 (database
migration) still routes through a not-yet-installed `database-reviewer` agent, a
deliberate, previously-recorded deferral, not a new discovery.
Adversarial review top findings: no mechanical enforcement exists that the routing table
is actually followed (it is convention, read and applied, the same as this project's own
`CLAUDE.md` files — the same limitation the task spec's own failure-containment section
anticipates); an installed agent's existence or invocation could be mistaken, by a future
session reporting results, for proof a gate (e.g. the Security Floor) passed — every
agent's own text disclaims this, but the disclaimer does not travel automatically into a
future summary; none of the seven installed agents generalizes beyond Calquartz, so a
future project under this workspace needs its own agent set from scratch.
Boundary compliance (all held): no Calquartz application feature implemented; no
application code written; no Calquartz repository created; no VPS, infrastructure, or
production system touched; no credential created; auth, booking, recurring-booking, RLS,
the exclusion constraint, and the durable-job system were not implemented; continuous
learning was not enabled; ADR-001 was not modified; Phase 9 was not started.
Next required action (owner): none required by this phase specifically. The next
substantive step is real Calquartz implementation planning, which should route its first
task through the now-installed `planner` agent and the Authority/Risk/Routing Model,
consistent with the remaining UNKNOWNs this phase and Phase 8A both name explicitly.

## [2026-09-09] phase | Foundation Implementation Plan — Adversarial Correction Pass: COMPLETE
Date: 2026-09-09
Phase: Planning-correction pass over `Calquartz — Application Foundation Implementation
Plan` and its existing Adversarial Review (Project #001, Calendar OS)
Status: Complete
Scope: a surgical, planning-only correction pass — no application code, repository,
migration, dependency install, VPS/production change, ECC agent modification, hook
installation, or CLAUDE.md change occurred. Re-read ADR-001, Phase 8A (incl. its own
Adversarial Correction Pass and Owner Approval), Architecture Open Questions, Architecture
Requirements and Constraints, the Security Floor, the Booking Correctness — Operational
Review Process, the Destructive-Operation Protection Design, the Durable Decision Capture
Policy, the Authority/Risk/Routing Model, the Verification Evidence Format, and the Phase
7 spike evidence directly, rather than trusting the existing plan's or its Adversarial
Review's own account of them, per the task's explicit instruction not to trust prior
summaries.
Evidence: no new spike or technical evidence was generated. Reasoned entirely from the
already-recorded authoritative Second Brain documents named above, cross-checked against
the existing plan (`wiki/projects/001-calendar-os/engineering/Calquartz — Application
Foundation Implementation Plan.md`) and its existing Adversarial Review, both already
present in the vault from earlier in this session.
Artifacts updated: `wiki/projects/001-calendar-os/engineering/Calquartz — Application
Foundation Implementation Plan.md` — appended a new `## Adversarial Correction Pass —
2026-09-09` section covering sixteen corrections (blocker sequencing narrowed from a
blanket WP0 gate to three scoped gates; the durable-job schema reclassified from a full
human-decision gate to a confirm-not-decide step against spike 4's already-specified
shape; the licensing/provenance and canonical-product-name blockers reclassified as
already closed by the plan's own accepted defaults, not open; the tenant-isolation
scaffolding's `withTenant`/`withSystem` naming assessed and recommended renamed to
mechanism-neutral terms; a previously-missing ambiguous/nonexistent-local-time policy
gate added; `packages/kernel` and `packages/obs` downgraded to local files for having no
cited requirement at all, `packages/config`/`packages/time` downgraded to conventions/
modules, `packages/contracts` explicitly preserved as Phase-8A-approved shared-typing
scope rather than eliminated; the transaction-ownership correction verified not to
silently decide recurring-series atomicity, with three distinct layers named explicitly;
package-boundary risks named for `packages/db` and `packages/jobs` with the latter's
"application permanently owns the job framework" caveat carried forward as unsettled per
Phase 8A's own correction pass; the three-entrypoint Docker topology's provenance
corrected to cite Phase 8A §5's Next.js/Fastify split rationale rather than ADR-001
alone; verification requirements strengthened with named proofs across tenant isolation,
database, jobs, security, time, and booking domains, including a DST/time proof Phase 8A
§22 already requires and the plan had dropped; a final complexity audit applying the
charter's own "maximum useful capability per unit of complexity" standard; every
still-open UNKNOWN restated without resolution; and WP0's authorization boundary
restated precisely as "potentially allowed now" vs. "must wait" categories). Frontmatter
`updated:` field amended to record the same-day append, per the file's existing
convention (matching Phase 8A's own pattern).
Pages created: `wiki/projects/001-calendar-os/engineering/Calquartz — Foundation Work
Packages (WP0–WP9).md` — a ten-work-package (WP0–WP9) breakdown reconstructed from the
plan's own summarized content plus this pass's own re-derivation from ADR-001, Phase 8A,
the Security Floor, and the Booking Correctness process, since the original live
`planner`-agent transcript that first produced a WP0–WP9 structure is not durably stored
anywhere in this vault. The file states this provenance plainly in its own opening
section — it is a reconstruction, not a recovered transcript, and the only concrete
WP-number-to-content anchor surviving from the original specialist output is the WP4/WP5
fragment already quoted in the plan's own §4 (tenant/membership schema with enrolment;
worker-role separation and job-table confirmation). Every package carries its own risk
tier, human-gate status, blocking condition, and required verification proofs, labeled
FACT/RECOMMENDATION/gate/UNKNOWN throughout.
Findings from the existing Adversarial Review: substantially corroborated by this pass's
own independent re-reading of the authoritative sources (not merely adopted on the
review's own authority) — its Review 2 (blocker misclassification), Review 3 (RLS-shaped
naming risk), Review 5 (package-boundary precision, including the `packages/jobs`
caveat-propagation gap), Review 6 (three-entrypoint provenance), Review 7 (missing
verification proofs, including the dropped DST requirement), and Review 11 (the WP0–WP9
provenance gap) all held up under independent verification against the cited primary
sources and are reflected in the corrections above. No finding of the existing Review was
rejected outright; several were narrowed or reclassified (the durable-job schema
blocker's severity; the licensing/product-name blockers' already-closed status) rather
than accepted verbatim.
No architecture, stack, or product-scope decision was changed, added, or reopened by this
pass — ADR-001 and Phase 8A's owner-approved §18 stack list remain exactly as they were.
No item was moved from UNKNOWN to DECISION; where a plausible default is newly named
(the local-time policy default), it is recorded explicitly as RECOMMENDATION, not
DECISION, per the Durable Decision Capture Policy's own rule 9–10.
Boundary compliance (all held): no Calquartz repository created; no application code
written; no migration executed; no dependency installed; no VPS or production system
touched; no `.claude/agents/` file modified; no CLAUDE.md modified; no hook installed;
ADR-001 not modified; the Authority/Risk/Routing Model, Verification Evidence Format, and
Security Floor documents not modified; no architecture decision changed.
Next required action (owner): review the corrected WP0 authorization boundary (Foundation
Implementation Plan, correction 16) and, if satisfactory, resolve or explicitly default
the local-time policy (correction 6) before any booking-domain work (WP6) begins; resolve
the tenant-isolation mechanism before WP4; resolve repository hosting/CI before WP1.
Foundation genesis work scoped as "potentially allowed now" (WP0's local-only portion,
WP2's tooling wiring, WP3, WP5's non-credential portions, WP8's non-boot-gate-specifics
portions) may proceed under the project's already-granted engineering autonomy without
further per-item sign-off.

## [2026-09-09] phase | Post-Audit Correction Pass — Foundation Implementation Plan + WP0–WP9 file: COMPLETE
Purpose and scope: apply, to the current prescriptive text of the Foundation
Implementation Plan and its WP0–WP9 companion file, the corrections that the WP0
Pre-Authorization Evidence Audit's §9(1) identified as still outstanding — items that
existed only as recommendations on paper (inside the plan's Adversarial Correction Pass
discussion) but had not yet been carried into the documents' own scope-defining text.
Both target files were verified directly (not from a prior summary) as last-modified
2026-09-09 ~15:09 local before this pass began: the Foundation Implementation Plan at
15:09:32 and the Foundation Work Packages (WP0–WP9) file at 15:09:56, confirming the
state this pass is logging against.
Authoritative sources reviewed during this pass, re-derived from the two target files'
own current citations (not guessed): ADR-001 — A/B1 Modular Monolith Architecture; Phase
8A — Implementation Stack Selection (including its own Adversarial Correction Pass and
Owner Approval); the Security Floor; the Authority, Risk, and Routing Model; the
Verification Evidence Format; Calquartz Booking Correctness — Operational Review
Process; Calquartz — Pre-Approval Technical Spikes (spikes 1, 3, 4 specifically); the
Foundation Implementation Plan's own Adversarial Review; and the WP0 Pre-Authorization
Evidence Audit whose §9(1) findings this pass corrects for.
Provenance note, stated plainly: the Foundation Work Packages (WP0–WP9) document is a
reconstruction/synthesis, not a recovered historical planner-agent transcript. Verified
directly in this pass: no transcript of the original live `planner`-agent invocation
that first produced a WP0–WP9 breakdown exists anywhere in this vault, under either
`wiki/projects/001-calendar-os/` or `raw/projects/001-calendar-os/`. This is not a
softened characterization — the transcript does not exist in durable storage, full
stop, and the WP0–WP9 file's own "Provenance" section states this fact directly.
Corrections actually made in this pass, verified present in the current file text
before being logged as done (all six confirmed by direct re-read):
1. DB tenant-context seam renamed from `withTenant`/`withSystem` to
   `withTenantContext`/`withElevatedAccess` directly in the Foundation Implementation
   Plan's §2 repository tree, with `SET LOCAL`, RLS-specific policies/roles, and
   `BYPASSRLS` explicitly prohibited from WP0's scope. RLS remains a possible,
   spike-1-validated candidate, not a decision.
2. The boot gate's three refusal conditions named explicitly wherever the plan
   references the gate generically: (a) missing/placeholder/insufficiently-random
   signing secret, (b) an unverified-TLS database URL, (c) an active demo/local
   provider fallback. Naming is documentation only; the gate's implementation stays in
   WP8, not WP0.
3. `packages/contracts` made explicitly transport-neutral in §2's own text — domain-
   shaped types only, no REST/tRPC/GraphQL/HTTP-status/router assumptions — without
   selecting among those paradigms (Phase 8A §19 remains UNKNOWN).
4. The integration-test harness's exact implementation shape marked UNKNOWN directly in
   §2's prescriptive text, not merely in the correction-pass discussion — no harness
   code exists yet, and the plan no longer implies mechanism-neutrality has already been
   proven.
5. `packages/jobs`'s permanent-ownership caveat (hand-rolled protocol vs. eventually
   adopting a library such as `pg-boss`/`graphile-worker`) added directly at §2's point
   of definition, not only in the correction-pass discussion.
6. Playwright/E2E scaffolding removed from WP0's scope — deferred to the work package
   that delivers the first real page. The eventual Playwright requirement itself (Phase
   8A §18 DECISION) is unaffected; only its WP0-time scaffolding moved.
Explicitly not corrected because evidence didn't justify it: none. Direct inspection of
both files' current "Post-Audit Correction Pass" / correction-note sections found no
item from the Evidence Audit's §9(1) list that was declined or left unaddressed — all
items named there as outstanding were applied. This absence-of-declined-items is itself
stated here rather than left implicit.
Provenance limitations: this pass, like the ones before it, relies on the plan's and
companion file's own cited readings of ADR-001/Phase 8A/the Security Floor rather than
re-deriving every cited fact from first principles in this specific pass; the underlying
reconstruction-not-transcript gap named above is not newly introduced by this pass and
is not closed by it — no transcript can be recovered that was never durably stored.
Risk/gate implications: none of the six corrections changes any risk tier, human-gate
determination, or blocker classification already recorded by the prior Adversarial
Correction Pass — they carry existing RECOMMENDATION-tier corrections into the
documents' own prescriptive text so a future reader relying on WP0's scope section alone
(not the full correction-pass discussion) sees the corrected scope directly.
Remaining UNKNOWNs, verified against the files' own current UNKNOWN lists: the tenant-
isolation mechanism (RLS vs. an alternative); the recurring-series atomicity model; the
ambiguous/nonexistent local-time product policy (a default is recommended, not decided);
the concrete auth library; whether a job library eventually replaces the application-
owned protocol; the API paradigm; the calendar provider; the notification provider; the
canonical product name (beyond the accepted working identifier "calquartz").
Implementation boundary: no code, repository, migration, credential, or production/VPS
change was authorized by this pass. Only the two named Second Brain documents' own
prescriptive text was edited, in-place, additively, per this vault's append-only-pass
discipline for correction sections.
Verification/evidence actually produced: none. This was a planning/documentation
correction pass; no Calquartz code exists to verify, and no Verification Evidence Record
applies.
Final status: COMPLETE. Both target files now carry all six corrections directly in
their prescriptive scope text, not only in discussion sections.
Next required action (owner): unchanged from the prior log entry — review the corrected
WP0 authorization boundary and resolve, or explicitly default, the tenant-isolation
mechanism (before WP4), the local-time policy (before WP6), and repository hosting/CI
(before WP1). A companion independent final pre-authorization audit
([[wiki/projects/001-calendar-os/engineering/Calquartz — Final Pre-Authorization
Audit|Calquartz — Final Pre-Authorization Audit]]) was produced in the same session,
logged separately below if a further entry exists at the time of reading, and should be
read before any WP0 work begins.

## [2026-09-09] phase | Final Independent Adversarial Audit — Calquartz Foundation Planning (second opinion): COMPLETE
Date: 2026-09-09
Phase: Final, independent adversarial pre-authorization review of the Calquartz foundation
planning artifacts (Project #001, Calendar OS), explicitly instructed to treat the prior
[[wiki/projects/001-calendar-os/engineering/Calquartz — Final Pre-Authorization
Audit|Final Pre-Authorization Audit]]'s own verdict as unverified and re-derive an
independent judgment from the current on-disk files rather than from any prior pass's
summary. Review only — no implementation, repository, migration, credential, or
infrastructure/deployment work of any kind was performed or authorized.
Files inspected (read directly, this pass, not from a prior summary): [[wiki/projects/
001-calendar-os/engineering/Calquartz — Application Foundation Implementation
Plan|Foundation Implementation Plan]] (incl. its Adversarial Correction Pass and
Post-Audit Correction Pass sections); [[wiki/projects/001-calendar-os/engineering/
Calquartz — Foundation Work Packages (WP0–WP9)|Foundation Work Packages (WP0–WP9)]];
[[wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation
Implementation Plan — Adversarial Review|the existing Adversarial Review]]; [[wiki/
projects/001-calendar-os/engineering/Calquartz — WP0 Pre-Authorization Evidence
Audit|the WP0 Pre-Authorization Evidence Audit]]; [[wiki/projects/001-calendar-os/
engineering/Calquartz — Final Pre-Authorization Audit|the prior Final Pre-Authorization
Audit]] (the object of this re-check); [[wiki/projects/001-calendar-os/decisions/ADR-001
— A-B1 Modular Monolith Architecture|ADR-001]]; [[wiki/projects/001-calendar-os/
engineering/Phase 8A — Implementation Stack Selection|Phase 8A]] (incl. its own
correction pass and Owner Approval); [[wiki/projects/001-calendar-os/engineering/Phase
8B — Minimum ECC Engineering Foundation|Phase 8B]] (partial, ECC-installation scope
only); [[wiki/projects/001-calendar-os/architecture/Security Floor|Security Floor]];
[[wiki/projects/001-calendar-os/engineering/Authority, Risk, and Routing Model|Authority,
Risk, and Routing Model]]; [[wiki/projects/001-calendar-os/engineering/Verification
Evidence Format|Verification Evidence Format]]; [[wiki/projects/001-calendar-os/Calquartz
— Pre-Approval Technical Spikes|Pre-Approval Technical Spikes]] (all four executed-spike
Results); [[wiki/projects/001-calendar-os/architecture/Phase 6 — Independent Adversarial
Architecture Review|Phase 6]]; [[wiki/projects/001-calendar-os/engineering/Phase 5B —
Factual and Technical Prerequisites|Phase 5B]]; [[wiki/projects/001-calendar-os/
questions/Architecture Open Questions|Architecture Open Questions]]; [[wiki/projects/
001-calendar-os/architecture/Architecture Requirements and Constraints|Architecture
Requirements and Constraints]]; [[wiki/projects/001-calendar-os/engineering/Calquartz
Booking Correctness — Operational Review Process|Booking Correctness — Operational
Review Process]]; [[wiki/projects/001-calendar-os/architecture/Multi-Tenancy Trust
Model|Multi-Tenancy Trust Model]]; [[wiki/projects/001-calendar-os/architecture/
Database-Level Tenant Isolation — RLS and Alternatives|Database-Level Tenant Isolation]];
[[wiki/projects/001-calendar-os/architecture/Threat Model|Threat Model]]; [[wiki/
projects/001-calendar-os/architecture/Failure-Mode Analysis|Failure-Mode Analysis]];
[[wiki/projects/001-calendar-os/engineering/Calquartz Security Floor — Operational
Review Process|Security Floor — Operational Review Process]]; [[wiki/projects/
001-calendar-os/engineering/Destructive-Operation Protection Design|Destructive-
Operation Protection Design]]; [[wiki/projects/001-calendar-os/engineering/Durable
Decision Capture Policy|Durable Decision Capture Policy]]; [[wiki/projects/
001-calendar-os/engineering/ECC Adoption Map|ECC Adoption Map]] (partial); the project's
own `CLAUDE.md`; the vault root `CLAUDE.md`; and this `log.md` file in full (both to
extract prior phase history and to verify, by direct grep, which of the audited passes
do and do not have their own log entries). File modification timestamps were checked
directly (not assumed) for the five core documents under audit, confirming the claimed
same-session sequence (Adversarial Review 13:15, Plan and WP0–WP9 file both 15:09, the
WP0 Pre-Authorization Evidence Audit 15:00, the prior Final Pre-Authorization Audit
15:29, all 2026-09-09 local) with no drift.
Verdict: **READY WITH REQUIRED CORRECTIONS** — same top-line verdict as the prior Final
Pre-Authorization Audit, independently re-derived rather than adopted. 0 Critical, 0
High, 3 Medium, 2 Low findings (the prior audit's own count was 0/0/2/2; every one of its
four findings was independently re-confirmed present and accurate, none was downgraded or
dismissed, and one new Medium finding was found this pass that neither prior review
identified).
Findings summary: (M1, confirmed, sharpened) WP0/WP3's "No human gate" rationale for the
tenant-context-seam scaffolding does not merely omit its reasoning, as the prior audit
found — it is directly inconsistent with the same document's own WP3/Evidence-Audit
classification of the identical scaffolding item as HIGH domain 1 (tenant isolation); the
concrete prohibition list (no `SET LOCAL`, no RLS-specific policies/roles, no
`BYPASSRLS`) remains intact and is the actual operative safeguard, so this stays Medium,
not High. (M2, confirmed, unchanged) no WP in the WP0–WP9 file explicitly names itself as
the home for recurring-booking series/occurrence scope, though recurring bookings are a
DECIDED v1 requirement. (M3, **new this pass**) WP7's own "Blockers: none" line and its
Summary-table "Blocked by" cell omit WP7's transitive dependency on WP4 (whose schema
WP7's own Scope text presupposes already exists) and therefore on blocker 1 (the
tenant-isolation mechanism decision) — found by applying the same "does this WP's scope
presuppose something a stated blocker gates?" test uniformly across all ten WPs rather
than only WP0/WP3. (L1, L2, confirmed, unchanged) logger-placement cross-reference
ambiguity between WP0 and WP8; the integration-test harness's foundation-necessity argued
from a "Phase 8A §12 requires it" premise rather than a stricter minimality-first one —
both non-blocking. A further, non-severity-tagged provenance observation: direct grep
across the full `log.md` found that neither the WP0 Pre-Authorization Evidence Audit nor
the prior Final Pre-Authorization Audit has its own dedicated log entry — both are
referenced only as pointers inside the "Post-Audit Correction Pass" entry above. This
does not affect either prior audit's content-level accuracy (both were independently
re-verified against current file state this pass and found accurate) — it is a
vault-logging-hygiene gap in those passes' own process, noted here rather than silently
left undiscovered, and not remediated beyond this note (remediating it would require
creating additional log entries beyond the one entry this pass's own task boundary
permits). No Critical or High finding was identified; two candidates (the M1 and M3
areas) were deliberately stress-tested for High classification, per this task's explicit
instruction not to preserve the prior verdict's momentum, and did not qualify — in both
cases the concrete, explicit prohibition/gate text protecting against unsafe action is
intact; only the surrounding rationale/completeness text is defective.
Files changed by this audit: exactly two — this `log.md` entry (vault root), and one new
file created **outside** the vault, at `C:\Users\HP\Desktop\Calquartz — Final Adversarial
Audit Handoff.md` (Windows Desktop) — an exhaustive, self-contained handoff document
(final verdict; executive rationale; Critical/High/Medium/Low findings with evidence;
every required correction; every owner decision required and what may proceed without
it; a WP0 item-by-item gate table; a WP0–WP9 sequencing assessment; booking-correctness,
tenant-isolation/security, verification, provenance, and minimality assessments; an
explicit confirm/downgrade/upgrade/dismiss comparison against the prior Final
Pre-Authorization Audit's own four findings; exact wording recommendations; a tiered
"Recommended Upload Set" for an external reviewer; and an evidence-status discipline
(Observed/Documented/Inferred/Reconstructed/Recommended/Unknown) applied throughout).
**The Desktop file is explicitly outside this vault and is a handoff artifact for an
external reviewer, not vault content** — it is not linked from `index.md` or this
project's `README.md`, consistent with its purpose. No other file was created, edited, or
deleted. In particular, the Foundation Implementation Plan, the WP0–WP9 companion file,
the existing Adversarial Review, the WP0 Pre-Authorization Evidence Audit, and the prior
Final Pre-Authorization Audit were all read only and remain exactly as they were before
this pass began.
Boundary compliance (all held): no Calquartz application code written; no repository
created, cloned, or modified; no database, migration, or schema created or altered; no
credential, secret, or production/VPS/deployment configuration created, read, or
modified; no dependency installed; no architecture, stack, or product decision made,
changed, or silently converted from RECOMMENDATION/UNKNOWN to DECISION; no owner decision
invented, inferred, or assumed. **No implementation was performed at any point in this
pass.**
Next required action (owner, not Claude): apply the five corrections named in this
audit's handoff document (§7/§19 there) to the Foundation Implementation Plan and the
WP0–WP9 file's own prescriptive text — none is a precondition for WP0's already-narrowly-
scoped local-only portion to proceed, per this audit's own §9 finding, but all five
should be applied before the corrected documents are treated as final. Separately,
unchanged from every prior pass: resolve or explicitly default the tenant-isolation
mechanism (before WP4), the ambiguous/nonexistent local-time policy (before WP6), and
repository hosting/CI (before WP1); confirm (not design) the durable-job table migration
against spike 4's specification (before WP2's migration is applied). This audit does not
authorize WP0 or any other work package to begin — authorization remains governed by the
WP0–WP9 file's own per-package human-gate determinations, unchanged by this pass.

## [2026-09-09] phase | Surgical Correction Pass — Foundation Implementation Plan + WP0–WP9 file (M1/M2/M3/WP8-wording): COMPLETE
Date: 2026-09-09, ~20:05–20:07 local (immediately following the "Final Independent
Adversarial Audit" entry directly above).
Phase: Surgical correction pass on the Calquartz Foundation Implementation Plan and its
WP0–WP9 companion file (Project #001, Calendar OS). Explicitly scoped as planning-prose
correction only, not implementation, not architecture selection, not a restart of either
document.
Purpose: apply the corrections identified by the two prior independent audits — the
[[wiki/projects/001-calendar-os/engineering/Calquartz — Final Pre-Authorization
Audit|Final Pre-Authorization Audit]] (M1, M2) and the "Final Independent Adversarial
Audit" logged directly above this entry (M1 sharpened, M2 confirmed, M3 new) — plus their
shared external handoff document, `C:\Users\HP\Desktop\Calquartz — Final Adversarial
Audit Handoff.md` (outside this vault, five named corrections at its §7/§19), plus one
additional defect (WP8's stale verification wording) identified directly against the
current file text at the start of this pass, not attributed to either prior audit.
Corrections actually made, each verified present in the current file text after editing
(re-read in full before this log entry was written):
1. **WP0/WP3 tenant-context human-gate reasoning (M1)** — both sections previously
   asserted or implied "No human gate because this is LOW/reversible work not touching a
   HIGH domain," which was factually inconsistent with the same documents' own
   classification of tenant isolation as HIGH/CRITICAL domain 1 (Authority, Risk, and
   Routing Model §2.1, fixed and not overridable by reversibility). Rewritten in both
   sections to state explicitly: the work does touch domain 1; the "No" gate holds only
   because the work is simultaneously local-only, unmerged, undeployed, mechanism-neutral,
   and explicitly prohibited from implementing `SET LOCAL`, RLS-specific policies/roles,
   `BYPASSRLS`, or any other mechanism-specific tenant-isolation behavior; the gate
   activates before merge/deploy (per the Authority/Risk/Routing Model §3.2's standard
   framing) and also immediately if mechanism-specific behavior is introduced earlier.
   The underlying tenant-isolation mechanism decision itself was not touched — RLS remains
   a possible, spike-1-validated candidate, not selected.
2. **Recurring-series/occurrence scope ownership (M2)** — no work package previously
   stated which package, if any, owns the recurring-booking series/occurrence data model,
   despite recurring bookings being a DECIDED v1 requirement (ADR-001). Added an explicit
   paragraph to WP6's Scope, and a matching sentence to "What this document does not do,"
   stating that this design is deferred to the first booking-domain vertical slice, a
   future work package outside WP0–WP9's own scope — chosen over homing it in WP6, with
   the reasoning for that choice stated in the text itself. The exact data model and the
   recurring atomicity model (all-or-nothing vs. explicit partial success) remain UNKNOWN,
   unchanged.
3. **WP7's inherited dependency on WP4 / blocker 1 (M3)** — WP7's Scope previously stated
   `users`/`memberships` were "already built in WP4" while its own Blockers line and the
   Summary table's "Blocked by" cell said "None" — a direct contradiction with WP4's own
   entry, which is blocked by blocker 1 (the tenant-isolation mechanism). Corrected by
   splitting WP7 into an architecture-scaffolding sub-scope (no dependency, may proceed
   without the auth library or WP4 having completed) and an actual-integration sub-scope
   (depends on WP4, which inherits blocker 1) — both the Scope text and the Summary table
   row were updated. Auth-library selection remains explicitly UNKNOWN.
4. **WP8 boot-gate verification wording** — WP8's Scope already named the boot gate's
   three refusal conditions (missing/placeholder/insufficiently-random signing secret;
   unverified-TLS database URL; active demo/local provider fallback — Security Floor item
   15's own evidence base, re-verified directly against that document in this pass and
   confirmed to match verbatim), but its Verification-required section still said
   "currently unspecified which three" and listed a different, non-matching informal
   example set. Corrected to state the same three conditions in both places.
Secondary cleanups applied in the same pass: (A) logger ownership — WP0 now states it
creates `packages/kernel`/`packages/obs` as empty stub files only, and WP8 now states it
implements the logger's real allowlist-by-construction behavior inside the file WP0
already scaffolds, not a second creation (closes L1 from both prior audits). (B) Phase 8A
provenance for the three-entrypoint shape — the Plan's own `docker/` line asserted the
three-entrypoint topology without citing Phase 8A's Next.js/Fastify-split rationale
(Phase 8A's own Adversarial Correction Pass, item 5, re-verified directly against that
document in this pass), which the Plan's own prior "correction 10" discussion had already
flagged as needed but never carried into the prescriptive `docker/` line itself; now
cited alongside ADR-001's process-count requirement. (C) jobs permanent-ownership caveat
— checked at every place `packages/jobs` is described (Plan §2, WP0, WP2); already present
and correctly worded everywhere; no change required. (D) package minimality — checked
against the existing Post-Audit correction table; already correct; no change required.
(E) verification-proof wording — added an explicit transaction-rollback (primitive-level)
proof to WP0, which was entirely absent from the WP0–WP9 file despite being named in the
Plan's own correction 11; tightened WP2's job-protocol proof to explicitly name
claim/lease/fence/`SKIP LOCKED` behavior rather than a generic "protocol works" reference.
Not addressed, correctly: (L2) the integration-test-harness minimality question — named
by both prior audits as optional/non-blocking, left as-is, not a defect.
Provenance strengthening: the WP0–WP9 file's own Provenance section now states plainly
that the document, together with the Plan, is the proposed implementation-planning
artifact submitted for owner authorization — not a decision, not evidence of the original
`planner` transcript, and not itself an approval of anything it describes. No historical
claim was added or altered; this is a status statement, not a provenance fact.
Files changed: exactly two — [[wiki/projects/001-calendar-os/engineering/Calquartz —
Application Foundation Implementation Plan|the Foundation Implementation Plan]] (one
edit: the `docker/` line's citation) and [[wiki/projects/001-calendar-os/engineering/
Calquartz — Foundation Work Packages (WP0–WP9)|the WP0–WP9 companion file]] (the four
corrections plus the secondary cleanups above). No other file was modified in producing
these edits. This vault is not under git version control (verified directly this session
— `git status` from both the vault root and this project directory returns "not a git
repository"), so no commit or diff exists for these changes; the claim above rests on
direct authorship (no other file was opened for writing during this pass) plus
corroborating filesystem evidence (mtime check across `wiki/projects/001-calendar-os/`
found only these two files newer than the session's preceding audit artifacts).
Risk/gate implications: none of the corrections changed any risk tier, human-gate
determination ("Yes"/"No"), or blocker classification — every edit added reasoning,
named an existing gap explicitly, or split an existing blocker without loosening it.
Verified directly: no WP's gate moved from Yes to No or vice versa.
Remaining UNKNOWNs, verified unchanged against the corrected files' own text: the
tenant-isolation mechanism; the exact recurring-series/occurrence data model; the
recurring atomicity model; the ambiguous/nonexistent local-time product policy; the
concrete auth library; job library vs. hand-rolled; the booking conflict/exclusion
mechanism; the public API paradigm; the calendar provider; the notification provider;
seats; round-robin/team assignment; the canonical product name (beyond the accepted
working identifier "calquartz"). None was resolved, narrowed, or converted from
RECOMMENDATION to DECISION by this pass.
Boundary: no Calquartz application code was written; no repository was created, cloned,
or modified; no database, migration, or schema was created or altered; no credential,
secret, or production/VPS/deployment configuration was created, read, or modified; no
dependency was installed; no architecture, stack, or product decision was made or
changed. Only prose in the two named Second Brain documents was edited.
Verification/evidence actually produced: none — planning-prose correction only; no
Calquartz code exists to verify.
Final status: COMPLETE. Both target files now carry the M1/M2/M3 corrections, the WP8
wording fix, and the secondary cleanups directly in their prescriptive text.
Next required action: an independent final pre-authorization audit of the corrected
state (this log's next entry, if produced in the same session) — not a further planning
rewrite. Separately, unchanged from every prior pass: resolve or explicitly default the
tenant-isolation mechanism (before WP4), the local-time policy (before WP6), and
repository hosting/CI (before WP1); confirm (not design) the durable-job table migration
against spike 4's specification (before WP2's migration is applied).
**Correction note, same session, immediately after this entry**: the owner explicitly
clarified that the independent final pre-authorization audit anticipated above is to be
performed separately (via the GitHub reference copies and ChatGPT, not by Claude in this
same correction-pass session) and instructed Claude to stop after this log entry. One
audit document was briefly created and then removed in this same session before that
instruction was seen — see the entry immediately below for the exact record. No audit was
left standing in this vault as a result of this pass.

## [2026-09-09] schema | Desktop cleanup — stray .md files relocated into the Second Brain
Date: 2026-09-09, ~21:2x local (immediately following the Surgical Correction Pass entry
above, by explicit owner instruction: "look at the desktop for the .md files and place
them in the right folder for our second brain — having them on the desktop serves no
purpose").
Scope: housekeeping only. No planning content was edited; no architecture or product
decision was touched.
Also recorded here: in the same short window, an audit document titled `Calquartz —
Post-Correction Final Pre-Authorization Audit.md` was briefly created under
`wiki/projects/001-calendar-os/engineering/` as part of this session's own independent
audit attempt, then deleted before this entry, once the owner clarified (see the
correction note in the entry directly above) that the independent final audit is to be
performed separately and should not be produced by this same correction-pass session. It
was never linked from `index.md` or any project `README.md`, and no other document in
this vault cites it by name — its removal leaves no dangling reference.
Found on Desktop (`C:\Users\HP\Desktop\`, outside the vault): seven `.md` files.
Action taken, by file:
- `Calquartz — Application Foundation Implementation Plan.md`,
  `Calquartz — Foundation Work Packages (WP0–WP9).md`,
  `Calquartz — VPS Infrastructure Inventory.md`,
  `Calquartz — Legacy SnagTime Decommission Report.md` — each byte-for-byte diffed
  against its existing canonical counterpart already durably filed in this vault
  (`wiki/projects/001-calendar-os/engineering/` for the first two,
  `wiki/projects/001-calendar-os/architecture/` for the latter two) and found
  **identical**. These were stray export copies with no unique content. **Deleted from
  the Desktop**, not moved — moving would have meant overwriting an identical file.
- `Calquartz — Final Adversarial Audit Handoff.md`, `Calquartz_Foundation_Audit_Agent_Handoff.md`,
  `CHATGPT_HANDOFF_CONTEXT.md` — three external-agent/ChatGPT handoff-context documents
  with no existing counterpart anywhere in this vault. Filed per this project's own
  Consultation Protocol (root `CLAUDE.md` §40; the existing
  `raw/projects/001-calendar-os/chatgpt-consultations/2026-09-06-calquartz-architecture-review.md`
  precedent) into
  `raw/projects/001-calendar-os/chatgpt-consultations/`, **moved with their original
  filenames preserved** (not renamed to the date-prefixed convention of the one existing
  file in that folder, to avoid inventing a naming scheme not explicitly requested).
Result: the Desktop now contains zero `.md` files (verified directly: `find` over the
Desktop root, non-recursive, returned nothing). All three relocated files are treated as
raw evidence from this point forward, per this vault's Rule 1 (`raw/` is read-only —
never edited, never deleted, never reformatted after filing).
Boundary: no wiki content was edited; no planning document (Plan, WP0–WP9, ADR-001, or
any authoritative source) was modified; no code, migration, credential, or production
change occurred; no architecture or product decision was touched.
Next required action: unchanged — see the Surgical Correction Pass entry above. The
independent final pre-authorization audit remains outstanding and will be performed
outside this session, per the owner's explicit instruction.
