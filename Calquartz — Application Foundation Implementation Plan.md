---
type: synthesis
created: 2026-09-09
updated: 2026-09-09 (Adversarial Correction Pass appended, same day; Post-Audit Correction Pass appended, same day, following the WP0 Pre-Authorization Evidence Audit)
sources: ["live planner-agent output, this session, 2026-09-09", "live architect-agent output, this session, 2026-09-09", "wiki/projects/001-calendar-os/decisions/ADR-001 — A-B1 Modular Monolith Architecture.md", "wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack Selection.md", "wiki/projects/001-calendar-os/architecture/Security Floor.md", "wiki/projects/001-calendar-os/engineering/Authority, Risk, and Routing Model.md", "wiki/projects/001-calendar-os/engineering/Verification Evidence Format.md", "wiki/projects/001-calendar-os/reuse/Reuse Audit Summary.md"]
tags: [engineering, planning, foundation, calendar-os, calquartz]
confidence: medium
provenance: hand-authored synthesis of two live specialist-agent invocations (planner, architect), this session, 2026-09-09; not itself a specialist output
classification: RECOMMENDATION — no owner approval sought or given in this pass; contains explicit blockers requiring owner decisions before implementation
scope: calquartz-specific
status: DRAFT — plan only; no application code, repository, or governance file was created or modified in producing it
---

# Calquartz — Application Foundation Implementation Plan

**Plan only.** No Calquartz application code was written. No repository was created.
No governance document, ADR, agent file, or Second Brain decision record was modified.
This is a RECOMMENDATION, not a decision — see §5 for the specific items requiring
owner sign-off before any implementation begins.

## 1. Purpose and scope

Plan the minimum Calquartz application-repository foundation needed before real
feature work (booking, tenancy, auth) begins, consistent with the approved A/B1
architecture (ADR-001) and the approved implementation stack (Phase 8A §18, DECISION
as of the Phase 8B owner-approval note). Two specialist agents were consulted live
during this planning pass — see §4 for the full routing/evidence record. This document
is my own synthesis of their two outputs plus my own reasoning; §4 marks precisely
which claim below came from which source.

## 2. Proposed repository shape

One repository, one pnpm workspace, one deployable Docker image:

```
calquartz/
  apps/
    api/        Fastify HTTP process — request handling only (see §3, correction 1)
    web/        Next.js — UI/SSR only
    worker/     job runner, same image, different entrypoint
  modules/      domain modules, created on demand, empty at foundation time
  packages/
    contracts/  types-only, shared between apps/web and modules/* (see §3, correction 2).
                Domain-shaped, transport-neutral types only — must NOT encode REST,
                tRPC, GraphQL, HTTP-status, router, or other transport-specific
                assumptions. The API paradigm remains UNKNOWN (Phase 8A §19); this is a
                constraint on the package's shape, not a selection among paradigms.
    kernel/     errors, result types, ids — no dependencies
    config/     schema-validated env, the single place any secret is read
    db/         pool + Kysely instance + the mechanism-neutral tenant-context seam
                (`withTenantContext` / `withElevatedAccess`; see §3, correction 5). WP0
                must NOT implement `SET LOCAL`, RLS-specific policies/roles,
                `BYPASSRLS`, or any other mechanism-specific tenant-isolation
                implementation — RLS remains a POSSIBLE, UNRESOLVED candidate, not a
                decision.
    jobs/       claim/lease/fence/SKIP LOCKED/checkpoint protocol only (spike-4-
                validated), zero domain handlers. Whether the application permanently
                owns a hand-rolled job framework is NOT decided (Phase 8A's own
                Correction Pass) — the protocol is FACT-backed; the permanent-ownership
                question is not.
    time/       sole Luxon importer, with ambiguous/nonexistent-time cases surfaced
                explicitly in its return types rather than resolved silently
    obs/        allowlist-by-construction logger, request/job id propagation
  migrations/   node-pg-migrate, SQL-first, one flat migration history
  test/
    integration/  real-Postgres harness (template-DB-clone-per-file, not
                   transaction-wrapped — see §4, planner finding). Its exact
                   implementation shape is UNKNOWN until harness code exists and is
                   checked — nothing has been built yet, and this plan does not claim
                   mechanism-neutrality has already been proven. Must not silently
                   introduce RLS-specific test mechanics.
    e2e/          Playwright — implementation/scaffolding deferred out of WP0 (no real
                   page exists yet to exercise); relocated to the work package that
                   delivers the first real page (see the WP0–WP9 companion file). The
                   eventual Playwright requirement itself is not removed from the
                   broader plan (Phase 8A §18 DECISION), only its WP0 scaffolding.
  docker/        one Dockerfile, three entrypoints (api/web/worker)
```

Boundary rules enforced by lint/dependency-cruiser, not convention alone:
`apps/web` never imports `packages/db` or `modules/*` (only `packages/contracts`);
only `packages/db` may construct a pg pool or issue a raw query; `packages/jobs`
contains protocol only; module tables and module-specific SQL are owned by their
module, not centralized in `packages/db`.

## 3. Corrections `architect`'s review made to `planner`'s original structure

Two changes from the specialist review are incorporated as corrections, not
suggestions — both close a gap where the *structure itself* would have quietly
pre-decided something ADR-001 or Phase 8A deliberately left open:

**Correction 1 — transaction ownership moved out of `apps/api`.** The original
proposal had `apps/api` "own transaction boundaries." `architect` identified this as
the single sharpest problem in the plan: spike 2 found the recurring-series
all-or-nothing-vs-partial-success question is an *application transaction-boundary
choice*, and Phase 8A explicitly leaves that choice UNKNOWN. A request-scoped
transaction opened in the HTTP layer makes only one of those two shapes natural.
Fixed: `apps/api` owns request handling and delegates to module-level application
services, which own the transaction boundary; `apps/worker` invokes the same
services (needed for batch materialization); `packages/db` owns only the transaction
*primitive*.

**Correction 2 — added `packages/contracts`.** `planner`'s stated reason for the
Next.js/Fastify pairing (end-to-end TypeScript type sharing) directly contradicts the
boundary rule "`apps/web` must not import `packages/db` or `modules/*`" unless a
types-only package exists for `apps/web` to import instead. `architect` flagged this
as a structural contradiction that "dies on day one" without it. Added.

`architect` also narrowed the "`packages/db` is the sole gateway" claim: sole
constructor of the pool and sole owner of the tenant-context seam, **yes** (this
directly supports spike 1's validated pattern); sole owner of every module's actual
query-writing, **no** — that would create a horizontal layer cutting across every
module's data, straining exactly the vertical extraction seams ADR-001 designed A/B1
to preserve. Modules own their own repositories and SQL, importing the seam rather
than delegating query construction to a central package. This is reflected in §2
above.

## 4. Routing / evidence record

**Agents actually invoked, in order:**

1. **`planner`** — invoked first because this is, on its face, a planning task
   ("prepare the implementation plan"), and `planner.md`'s own description names
   exactly this responsibility ("Use PROACTIVELY for any feature request,
   architectural question, or refactor"). Purpose: produce the initial work-package
   breakdown, repository shape, risk classification, and blocker list.
2. **`architect`** — invoked second, after reading `planner`'s output, because that
   output leaned heavily on module-boundary and architecture-consistency judgments —
   `architect.md`'s specific, distinct remit ("applies Calquartz's approved
   architecture... flags when a proposed change would strain module boundaries") —
   and because adversarial-minimization discipline (Phase 8B §12) argues against
   letting one generalist agent's structural claims go unchecked when a narrower
   specialist exists for exactly that check. Purpose: independently verify the
   proposed repository/module structure against ADR-001 and flag anything that
   quietly decides an open question.

**Actual specialist output for each** (verbatim, not paraphrased, reproduced in full
in the two preceding conversation turns' task notifications; summarized here with
direct quotation of the load-bearing claims only, per this document's own space
constraints — the full text is preserved in this session's transcript):

- **`planner`**, in full, proposed: the repository shape in §2 (before `architect`'s
  two corrections); nine work packages (WP0–WP9) each with an explicit risk tier,
  routing chain, and human-gate determination; a verification-evidence section
  naming five specific negative proofs the foundation phase should produce (boundary
  lint firing on a deliberate violation, fail-closed enrolment firing on an
  unenrolled table, the boot gate refusing three specific bad configs, migration
  up/down/up reversibility, and the job protocol reproducing spike 4's result inside
  real application code); and five numbered blockers (quoted in §5 below,
  attributed).
- **`architect`**, in full, confirmed the overall shape stays within A/B1 and meets
  none of ADR-001's four reopen triggers; identified the transaction-boundary and
  contracts-package issues (§3); narrowed the `packages/db` sole-gateway claim;
  flagged that `packages/time` risks silently resolving the still-open
  ambiguous/nonexistent-local-time policy unless its return types make both cases
  explicit; flagged that the worker's database credential must stay narrower than
  the web application's per spike 1 criterion 3 (not visible in the original
  proposal); and stated explicitly "No escalation required... none of ADR-001's four
  reopen triggers" is met by anything proposed.

**Which specialist recommendations/findings were incorporated into the final plan:**
both of `architect`'s two "correction" items (§3, in full); `architect`'s narrowing
of the `packages/db` claim (§2, module ownership of module tables); `architect`'s
`packages/time` explicit-surfacing requirement (§2); `planner`'s full work-package
structure, risk classifications, and verification-evidence list (§2, and referenced
above — not fully restated in this document per "one fact, one home," since the
complete WP0–WP9 breakdown exists verbatim in the session transcript and restating it
here would duplicate rather than cite it — **this is itself a limitation of this
document flagged honestly**, see §6).

**Which were rejected or left unresolved, and why:** none of either specialist's
findings were rejected. `architect`'s recommendation to record, once a repository
exists, that A/B1's "web + worker" is realized as three processes from one image was
**not yet acted on** — there is no repository to record it in yet; carried forward as
a to-do for WP0. The worker-credential-narrower-than-web requirement and the
catalog-driven fail-closed enrolment check were both already present in spike 1's own
evidence and in `planner`'s WP4/WP5 descriptions; `architect`'s contribution was
confirming they need a structural home rather than being left implicit, which is
noted but does not change the work-package boundaries `planner` proposed.

**Claims in this document that come from my own reasoning, not a specialist:** the
overall document structure and section ordering (§1–§7); the framing sentence in §3
characterizing the two corrections as corrections rather than suggestions; the
decision to consult `architect` at all, and the stated reason for that choice (§4,
item 2's "Purpose" line is my synthesis of why I invoked it, not architect's own
self-description); §6 and §7 in full.

**Evidence actually inspected or used:** the two specialists' own live outputs (the
only "evidence" this document rests on beyond citation); I did not independently
re-read ADR-001, Phase 8A, the Security Floor, or the spike evidence myself in this
pass — both specialists report having read them directly (their outputs name exact
absolute file paths they consulted), and I am relying on their citations rather than
independently re-verifying each one in this synthesis pass. **This reliance is stated
explicitly, not concealed**, and is itself a candidate limitation (§6).

**Important UNKNOWNs that prevent implementation:** see §5 (blockers) in full — these
are `planner`'s own numbered list, reproduced with attribution, not restated as if
newly discovered by this synthesis.

## 5. Blockers — owner decisions required before WP0 begins

Reproduced from `planner`'s output, attributed, not modified:

1. **The tenant-isolation mechanism** (ADR-001 explicit non-decision; RLS is the
   leading, spike-1-validated candidate, not yet selected). The scaffolding for it
   (schema convention, fail-closed enrolment test, the mechanism-agnostic
   `withTenant` seam) is buildable now; the mechanism itself is not Claude's to
   select. Sequencing consequence `planner` names explicitly: deciding this after
   several migrations exist is materially more expensive than deciding it now.
2. **The durable-job table schema** (concrete columns for the job/effect-log tables)
   — requires a human gate per the routing table's "touches a shape a prior Second
   Brain document assumed" rule.
3. **Repository hosting and CI provider** — where the repo lives, private vs. public,
   which CI runs it, where its secrets live.
4. **Licensing clearance for two named reuse candidates** (Cal.diy's `csp.ts`,
   SnagTime's `assertProductionRuntimeSecurity` pattern) before either is copied
   verbatim rather than re-derived. Default assumption in the plan if unresolved:
   re-derive, copy nothing.
5. **The canonical product name** — still an ADR-001 non-decision; bakes into
   package scope, image name, database name, and role names. Default if unresolved:
   working slug `calquartz`, accept a rename cost later.

Two smaller items `planner` marked as resolvable by Claude autonomously if the owner
declines to weigh in: where Verification Evidence Records live (default: an in-repo
`verification/` directory), and whether the workspace-root `.claude/agents/` are also
installed into the new repository once it exists.

**Not blockers, deliberately not resolved here** (per both specialists and per
Phase 8A's own explicit non-decisions): the concrete auth library, the
nonexistent/ambiguous local-time policy, the recurring-series atomicity model, seats,
round-robin. None is needed to lay the foundation; the foundation must not encode an
assumption about any of them.

## 6. Limitations of this synthesis (stated honestly, not smoothed over)

- This document does not reproduce `planner`'s full WP0–WP9 breakdown verbatim,
  citing "one fact, one home" — but that means a reader of *only* this document does
  not have the complete work-package detail (risk tier, exact routing chain, and
  human-gate determination per package) that `planner` actually produced. That detail
  exists only in this session's transcript at present, which is a real access gap
  for anyone reading this document later without that transcript. Recorded as a
  candidate follow-up: transcribe the full WP0–WP9 table into this document or a
  companion file before this plan is acted on.
- I did not independently re-verify either specialist's citations against the source
  documents in this synthesis pass — I am relying on their reported readings. Both
  specialists' outputs are internally consistent with each other and with what this
  session has independently confirmed in earlier passes this conversation (ADR-001's
  content, Phase 8A's DECISION status, the Security Floor's 16-item structure), which
  is corroborating but not equivalent to a fresh independent re-check of this specific
  synthesis's claims.
- Neither specialist was asked to produce a formal Verification Evidence Record for
  this planning pass, and none was produced — correctly, since planning is not
  verification and nothing here claims otherwise.

## 7. Status

**RECOMMENDATION.** Not a decision. No owner approval sought or recorded in this
pass. WP0 (repository genesis) cannot begin until blockers 1–5 in §5 are resolved by
the owner; work packages `planner` classified LOW may proceed under the project's
already-granted engineering autonomy (project `CLAUDE.md` §3) once the blockers
clear, without requiring further owner sign-off on each one individually.

## Adversarial Correction Pass — 2026-09-09

Scope note: this pass is a **review-and-correction pass, not a restart**, following the
same discipline as [[Phase 8A — Implementation Stack Selection]]'s own "Adversarial
Correction Pass" section (the style model for this one). Nothing in §1–§7 above is
deleted or rewritten. Where this pass concludes something above was overstated,
misclassified, or missing, that is corrected here, additively, with a pointer back.

This pass was triggered by re-reading ADR-001, Phase 8A (including its own correction
pass and Owner Approval), Architecture Open Questions, Architecture Requirements and
Constraints, the Security Floor, the Booking Correctness — Operational Review Process,
the Authority/Risk/Routing Model, the Verification Evidence Format, and the Phase 7 spike
evidence directly — not merely trusting §4's account of what `planner`/`architect`
reported reading — and by independently re-deriving judgment on
[[Calquartz — Application Foundation Implementation Plan — Adversarial Review|the
existing Adversarial Review]]'s twelve findings rather than accepting them uncritically.
Where this pass's own re-reading agrees with that review, the review is cited as
corroborating INFERENCE, not treated as itself authoritative — a review is a
RECOMMENDATION-tier document, same rank as the plan it reviews.

**Where an authoritative source and a lower-rank document (this plan, or the Review)
disagree, the authoritative source is preserved as ground truth and the discrepancy is
recorded, not silently resolved by picking whichever reads better.** No discrepancy of
that shape was found in this pass beyond what is already named below (the §7/§5
self-contradiction and the naming/topology points) — ADR-001, Phase 8A, the Security
Floor, and the spike evidence are internally consistent with each other on every point
this pass checked.

### (1) Blocker sequencing — corrected

**FACT**, re-verified directly against ADR-001 and Phase 8A: none of the four named
implementation gates (security, correctness, tenant isolation, DST, durable-job,
licensing/provenance, verification — ADR-001 "Implementation gates") is stated as a
precondition for *all* foundation work; each gates the specific slice of work it concerns.
§7's sentence — "WP0 (repository genesis) cannot begin until blockers 1–5 in §5 are
resolved" — is **not supported by this document's own §5 text**, which already scopes
blockers 1 and 3 narrowly and supplies accepted defaults for blockers 4 and 5. The
existing Adversarial Review's Review 2 reached the same conclusion independently; this
pass re-derived it from ADR-001/Phase 8A directly rather than only from that review, and
confirms it.

**Corrected framing (RECOMMENDATION, replacing §7's blanket statement)**:

- **Tenant-isolation mechanism** (blocker 1): blocks only tenant-scoped schema/data-access
  work that depends on the final mechanism — the first migration creating a
  tenant-scoped table, and any code that assumes a specific mechanism's shape. Does
  **not** block neutral repository scaffolding, local project structure, non-tenant
  tooling, documentation, or other reversible foundation work.
- **Repository hosting/CI** (blocker 3): blocks only remote-repository creation/push and
  CI activation. Does **not** block local scaffolding — a local, unpushed git repository
  can absorb most of WP0's work.
- **Local-time policy** (Phase 8A §8/§20 UNKNOWN — not previously named as a blocker at
  all in this plan; see correction 6 below): blocks only the first booking-domain
  implementation slice that constructs or interprets local wall-clock booking times. Does
  **not** block unrelated foundation work.

None of these becomes, or was ever correctly, a universal "nothing can start" gate. This
correction narrows §7's own sequencing claim; it does not change which decisions remain
owner-only (§5 is otherwise preserved).

### (2) Durable-job schema classification — corrected

**FACT** (spike 4, `4. Calquartz Spikes/evidence/spike-4-durable-jobs-evidence.md`, cited
via ADR-001/Phase 8A §9): the claim/lease/fence/`SKIP LOCKED` protocol, retry
exhaustion, stale-token detection, and incremental checkpointing for recurring/batch work
are evidence-backed, not merely proposed. §5 blocker 2 ("the durable-job table
schema... requires a human gate") **overclassifies** this: spike 4's evidence already
specifies the required columns and behavior (claim, lease, fencing token, effect log,
checkpoint state) closely enough that writing the actual migration is a
**confirm-not-decide step**, not a fresh design requiring a human gate under the
"touches a shape a prior Second Brain document assumed" rule (Authority/Risk/Routing
Model §3.2) — the prior document (spike 4) already specifies the shape being confirmed,
which is the case that rule's own routing table treats as a citation, not a new decision
(§3.2's "Architecture change — retrieval" row states the same principle for a different
task class, and the same reasoning applies here by analogy: citing an already-answered
question is not a new decision event).

**RECOMMENDATION**: draft the job-table migration directly from spike 4's evidence and
route it through the Authority/Risk/Routing Model's "Database schema/data migration" row
as a **confirmation**, requiring owner sign-off before the migration is applied (this
remains a HIGH-risk-domain row per §2.1 item 5, so the human gate itself is not removed —
only its character, from "resolve an open design question" to "confirm an
already-specified one," changes). **UNLESS** implementation reveals the schema cannot
satisfy the protocol, or surfaces a genuinely new architectural question (e.g. a column
shape that only makes sense under one candidate tenant-isolation mechanism) — in that
case, escalate as a new decision, not preemptively.

### (3) Licensing/provenance blocker — corrected

**FACT** (Security Floor item 16 evidence; Reuse Audit): dependency-license composition
for both reference repositories is UNKNOWN, and Cal.diy carries an unresolved
MIT-vs-`UNLICENSED` contradiction inside `apps/api/v2` — neither named reuse candidate
(`csp.ts`, `assertProductionRuntimeSecurity`) is stated to live inside that path.

The plan's existing default rule — **"re-derive, copy nothing unless licensing/provenance
is explicitly cleared"** — is correct and is **preserved unchanged** as the standing
default. **Correction**: this is not itself a foundation blocker, because neither named
candidate is required by any element of §2's foundation structure — `packages/config`'s
boot-gate validation and any CSP/security-header middleware are foundation-appropriate
work regardless of whether either upstream pattern is ever consulted, and the plan's own
stated default (re-derive) means no clearance is needed to proceed. **Reclassified**:
from "blocker" to **reuse-specific gate, not currently triggered** — it activates only
if verbatim copying of either specific file is ever proposed, at which point licensing
clearance is required *before* that specific act, not before foundation work generally.
§5 blocker 4 should read as **closed by the plan's own accepted default**, not open.

### (4) Canonical product name — corrected

**RECOMMENDATION** (unchanged in substance from §5 blocker 5, reclassified): "calquartz"
is accepted as a **provisional working identifier** for scaffolding (package scope,
image name, database name, role names) — not a hard genesis blocker. ADR-001's own
"Explicit non-decisions" section already states the canonical product name remains
unresolved and non-blocking. Final naming/branding remains explicitly open; a rename
later touches package.json, image tags, database name, and role names — mechanical, not
architectural, cost, as the plan itself already states. §5 blocker 5 should read as
**closed by the plan's own accepted default**, not open — the same correction pattern as
(3) above.

### (5) Tenant-isolation scaffolding naming — corrected

Direct assessment (not deferred to the existing Review, though it reaches the same
conclusion): the plan's `withTenant`/`withSystem` naming for `packages/db`'s seam is
assessed against the actual test proposed — **does it functionally commit to a
mechanism, or only read as if it does?**

**INFERENCE**: the naming does **not** functionally commit to RLS — a
`withTenant(tenantId, callback)` signature that opens a transaction and hands the
callback a connection can be implemented under RLS (`SET LOCAL` + policy) or under an
application-layer scoping mechanism (query rewriting/filtering) without changing the
call-site signature. **However**: the specific two-mode split (`withTenant` /
`withSystem`, an ordinary-scoped mode plus an explicit bypass mode) maps closely onto
RLS's own two-role shape (an ordinary role subject to policies, plus a `BYPASSRLS` role —
spike 1's own tested shape) in a way an application-layer-scoping alternative would not
naturally need (that alternative more naturally needs a query-rewriting seam, not a
transaction-context/bypass seam). This is a **RISK**, not a functional commitment: the
naming is *suggestive* of RLS, cheaply renamed, not load-bearing on RLS's actual
mechanics.

**RECOMMENDATION**: rename to mechanism-neutral terms (e.g. `withTenantContext` /
`withElevatedAccess`) before any code is written using them, since renaming costs nothing
now and costs a rename-every-call-site pass later once modules depend on the names. This
does not weaken the tenant-isolation *requirement* — ADR-001 and the Security Floor
(item 12) still require *some* database-level backstop; only the naming/framing changes.
**State explicitly, per ADR-001's own "Explicit non-decisions"**: RLS remains the
spike-1-validated leading *candidate*, not a decision. The integration-test harness
(template-DB-clone-per-file, not transaction-wrapped, per the plan's §2) should validate
tenant isolation as a behavior (a query issued without the seam must fail; a query issued
through the seam must be scoped correctly) in a way that is agnostic to which mechanism
is eventually selected — if the harness's specific shape is chosen *because* of an RLS
assumption (e.g. to test `SET LOCAL` surviving a simulated pool-reuse boundary), that
reason should be stated explicitly rather than left implicit, per the existing Review's
finding (Review 3) — this pass confirms that finding as correct but notes it as
**UNKNOWN, not confirmed**, since the plan's own text does not state its reasoning either
way.

### (6) Local-time policy gate — added (was missing)

**Gap, confirmed by direct re-reading of Phase 8A §8/§20**, not merely by the existing
Review's Review 8/12 findings (which found the same gap): the plan's §5 blocker list
omits the nonexistent/ambiguous-local-time product policy entirely, despite Phase 8A
already naming it as an explicit UNKNOWN and despite `packages/time`'s own existence in
§2 being motivated by exactly this gap. This is an **omission**, not a hidden decision —
but a load-bearing UNKNOWN missing from a document whose purpose is to enumerate
blockers is itself a defect.

**Added as an explicit gate (RECOMMENDATION)**: nonexistent local times, ambiguous local
times, timezone-transition handling, and recurring-occurrence-across-DST notification
timing are **UNKNOWN, owner-level, unless already established** — none is established by
any authoritative document read in this pass. This gate applies specifically to the
booking-domain implementation slice that constructs or interprets local wall-clock
booking times (per correction 1's scoping principle) — it does **not** block unrelated
foundation work (repository scaffolding, `packages/db`, `packages/jobs` protocol, CI).

A **proposed default may be recorded** (Phase 8A §8 already names one plausible shape):
reject nonexistent local start times at booking-creation time with a user-facing error;
require explicit UTC-offset disambiguation for ambiguous local times. This is stated here
as a **RECOMMENDATION**, explicitly **not a DECISION** — Phase 8A itself only surfaced,
did not resolve, this policy, and this pass does not resolve it either; only the owner
can convert it to a DECISION.

### (7) Premature standing packages — re-evaluated

Re-derived independently against each package's actual justifying evidence (not merely
adopting the existing Review's Review 1/10 table, though this pass's conclusions
substantially agree with it):

| Package | Required by an approved decision? | Concrete evidence? | Reversible local convention? | Premature abstraction? | Verdict |
|---|---|---|---|---|---|
| `packages/kernel` | No — no spike, ADR-001 clause, or Phase 8A selection names it | None | Yes | **Yes** | **Downgrade to a local file** (e.g. `lib/kernel.ts` in the first module that needs it); promote to a package only once a second module duplicates it. Strong presumption against standing status per correction instructions — "looks tidy" is not evidence of necessity. |
| `packages/obs` | No | None | Yes | **Yes, clearest case** | **Downgrade to a local file** (`logger.ts`); promote once `apps/worker` needs the identical thing `apps/api` already has. |
| `packages/config` | Partially — Security Floor item 15 (boot-gate validation) requires *some* mechanism | Security Floor item 15 | Yes | Mild | **Downgrade to a documented convention** (one validated config module per entrypoint) until duplication across `apps/api`/`apps/web`/`apps/worker` is actually measured, not assumed. The boot-gate *requirement* is real and foundation-appropriate; the *package* boundary is not yet earned. |
| `packages/time` | Partially — spike 3 requires wall-clock-correct construction and explicit ambiguous/nonexistent-time surfacing | Spike 3 evidence | Yes | **Yes, and risk-bearing** — see correction 6: a package-level API shape fixes a return-type contract before the product policy is decided | **Downgrade to a local module** with the explicit `{ok, value} \| {ok: false, reason}` result-type discipline; the *discipline* is foundation-appropriate now, the *package* boundary is not. |
| `packages/contracts` | Indirectly — `architect`'s correction (§3) is real (the boundary rule + Next.js/Fastify type-sharing goal do create the contradiction it fixes) but both the boundary rule and the type-sharing shape are themselves choices, not requirements; Phase 8A's own §19 leaves the API paradigm (REST/tRPC/GraphQL) explicitly UNKNOWN | Phase 8A §18 (type sharing is a **stated, approved reason** for the Next.js/Fastify pairing — see correction (7a) immediately below) | Moderate | Mild-to-moderate | **Keep as a package, but keep it minimal** — see (7a). |
| `packages/db` | Yes — spike 1 requires a pool + tenant-context seam | Spike 1 evidence | N/A — sole pool constructor is itself the point | No | **Keep as a package.** Rename the seam per correction 5. |
| `packages/jobs` | Yes — spike 4 requires the protocol | Spike 4 evidence | N/A | No, if scoped to protocol only | **Keep as a package**, scoped strictly to protocol (claim/lease/fence/`SKIP LOCKED`), per correction (7b) below for the caveat this must carry. |

**(7a) `packages/contracts` — important distinction, per instruction**: **Phase 8A §18
explicitly approved shared TypeScript types end-to-end as a stated reason for the
Next.js+Fastify pairing** — this is Phase-8A-approved (DECISION, per the Phase 8B Owner
Approval scope), not merely a plan-author preference. **This pass does not eliminate
`packages/contracts` or replace it with a codegen architecture** — doing so would
contradict an already-approved element of the stack decision, which is out of this
pass's scope to alter (correction pass instructions explicitly forbid converting a
recommendation into a decision, and equally forbid silently un-deciding one). What
**is** still open: the *exact implementation shape* of that type-sharing (a
hand-maintained package vs. a codegen/OpenAPI-generated client) is an implementation
detail Phase 8A did not fix — the plan may keep `packages/contracts` as currently scoped,
provided it stays a thin, types-only package with no runtime logic, since a public API
(Phase 8A §19, still UNKNOWN) would need a different, externally-versioned contract
shape if that ever materializes — an unstated assumption this pass now states explicitly
(also see correction 12).

**(7b) `packages/jobs` — caveat propagation**: Phase 8A's own Adversarial Correction Pass
§4 explicitly downgrades "the application permanently owns the full job-framework
implementation" from settled design to "a RECOMMENDATION with a stated re-evaluation
point." §2 of this plan states "claim/lease/fence protocol only, zero domain handlers"
without carrying that caveat forward. **Corrected**: `packages/jobs` at foundation time
should be described as implementing the spike-4-validated **protocol** (FACT-backed,
foundation-appropriate now); whether the application permanently owns that
implementation versus eventually adopting a library (`pg-boss`, `graphile-worker`)
remains **RECOMMENDATION, not settled**, exactly as Phase 8A states — this document
should carry that caveat wherever `packages/jobs` is described, not merely at its
original source.

**E2E Playwright scaffolding**: not a package-boundary question, but the same
"undemonstrated abstraction" test applies — standing up Playwright infrastructure before
a single page exists front-loads infrastructure a first vertical slice may not yet
exercise. **RECOMMENDATION**: defer to the work package producing the first real page;
the integration-test harness (real-Postgres, template-clone) is genuinely
foundation-level per Phase 8A §12 and should stay.

### (8) Transaction ownership — verified, not silently deciding atomicity

Direct re-check of `architect`'s correction (§3, correction 1) against spike 2's actual
finding: spike 2 established that recurring-series atomicity (all-or-nothing vs. explicit
partial success) is **an application transaction-boundary choice, not a
constraint-mechanism property** — this is **FACT**, cited correctly by `architect`.

**Verification performed by this pass**: does "module-level application services own
transaction boundaries; `packages/db` provides only the transaction primitive" itself
decide anything beyond what spike 2 established? **Answer: it decides *where in the call
stack* the transaction boundary sits (service-layer, not HTTP-layer) — a real,
previously-underspecified structural choice — but it does not decide *which atomicity
model* a module-level service implements.** A module-level service can express either
all-or-nothing or explicit-partial-success internally; the correction removes one hidden
decision (an HTTP-request-scoped transaction, which structurally favors all-or-nothing)
without introducing a comparably-sized new one. Stated explicitly, in writing, as three
distinct layers that must not be conflated:

1. **Transaction primitive** (packages/db: begin/commit/rollback, `SET LOCAL`) — mechanism only, no policy.
2. **Application-service transaction boundary** (which call owns opening/closing a transaction) — a structural choice, now settled by `architect`'s correction, reversible at moderate cost.
3. **Recurring-series atomicity policy** (all-or-nothing vs. explicit partial success) — **remains fully open**, an explicit ADR-001/Phase 8A non-decision, not touched by either of the above.

This correction **survives** the check: it is acceptably neutral because reshaping it
later touches only the calling convention inside affected modules, not the atomicity
policy itself or every module's own logic.

### (9) Package/boundary claims — precision pass

**`packages/db`**: may own the PostgreSQL pool, connection handling, transaction
primitives, and the tenant-context primitive; must **not** automatically become a
universal query gateway if modules own their own SQL (already correctly stated in the
plan, per `architect`'s narrowing). **Risk named explicitly, not previously named in this
plan**: the boundary-lint rule ("only `packages/db` may construct a pg pool or issue a
raw query") stops a module from *bypassing* the seam to get a raw connection; it does
**not** stop a module that legitimately has a connection via the seam from issuing a
query that never actually calls the tenant-context primitive first. **This is a known
gap, not a new one** — Phase 8A §21 already names the structurally identical gap for
Kysely generally ("a careless call site could still issue a query outside the transaction
that set `SET LOCAL`... the tool does not prevent this by construction"). **Mitigation
requirement, stated here as a verification/enforcement obligation, not a package-boundary
guarantee**: this must be caught by a specific verification proof (tenant-context
leakage — see correction 11) and/or a lint rule checking that every query inside a module
passes through the seam, not assumed solved merely because the pool is centralized.

**`packages/jobs`**: preserve the spike-4-validated protocol as FACT-backed. **Correction
applied**: strike any language implying the hand-rolled implementation has been "proven
superior" to all libraries — none exists in the plan's own current text (checked
directly; the plan already says "protocol only, zero domain handlers" without an
overclaiming comparative), so no correction to existing text is required here beyond the
caveat-propagation already stated in (7b). Stated affirmatively for clarity: the protocol
is validated; the implementation approach (hand-rolled vs. library-adopting) is a current
direction with a stated re-evaluation point, not a proof of superiority.

**`packages/time`**: must not become, by omission, the place product time semantics get
silently decided. Already substantially addressed by correction 6 (the gate) and
correction 7 (the downgrade to a local module) — restated here for completeness per the
instruction's own itemization: the module's return types must make ambiguous/nonexistent
cases explicit in the type system itself, not merely in a comment, so a future call site
cannot silently ignore the case.

### (10) Three-entrypoint topology — provenance corrected

**FACT**, re-verified directly against both source documents: ADR-001 states "one
deployable (web + worker processes)" — a **two-process** commitment. Phase 8A §18
(DECISION, per Phase 8B Owner Approval) selects Fastify **and** Next.js as paired stack
elements, and Phase 8A's own Adversarial Correction Pass §5 explicitly labels the
Next.js/Fastify **split** (as opposed to a Next.js-only backend) as a
**RECOMMENDATION resting on a real, non-manufactured argument** (a shared
worker/API application-domain layer outside a UI framework's request-lifecycle
assumptions) — not something ADR-001 independently mandates.

**Correction**: the plan's `docker/` section ("one Dockerfile, three entrypoints
(api/web/worker)") should cite **Phase 8A §5 specifically** (the Next.js/Fastify split
rationale) wherever it justifies the three-entrypoint shape, rather than presenting it as
following directly from ADR-001's "web + worker" language alone — ADR-001's own text
would also permit a simpler two-process shape (Next.js serving both UI and API routes).
This is **not** a claim that the plan's three-entrypoint choice is wrong: Phase 8A's
reasoning for the split is real and already-approved as a stack element. It is a
provenance correction — cite the decision that actually produced this shape (ADR-001's
process-count requirement **combined with** Phase 8A's stack selection), not ADR-001
alone.

### (11) Verification requirements — strengthened

The plan's §4 references five negative proofs from `planner`'s output (boundary-lint
firing, fail-closed enrolment firing, boot gate refusing bad configs, migration
up/down/up, job protocol reproducing spike 4). Cross-checked directly against Phase 8A
§22 (which names required Verification Evidence Records explicitly) and the Security
Floor/Booking Correctness process: **the DST/time verification Phase 8A §22 explicitly
requires ("the Luxon wall-clock construction reproducing spike 3's result... inside the
actual availability/slot-generation code") is missing from this plan's list** — a
propagation gap from a higher-authority document (Phase 8A, rank 2/3) into a
lower-authority one (this plan). **Added below**, along with additional proofs this pass
identifies as missing by direct comparison against the Security Floor, the Booking
Correctness process, and the Authority/Risk/Routing Model's own domain list — not merely
adopted from the existing Review, though several coincide with its Review 7 findings:

**Required verification set for foundation-adjacent work, by domain** (named proofs, not
a generic test suite, per instruction):

- **Tenant isolation**: context leakage (a query issued without the tenant-context seam
  must fail, not silently succeed — the single highest-value missing proof, per
  correction 9's named gap); attacker-controlled cross-tenant access (not merely
  schema-enrolment — Reuse Audit §36 distinguishes these explicitly: "asserts data
  separation" is not the same proof as "attacker-controlled cross-tenant access");
  unenrolled-table detection (fail-closed enrolment, already named); worker-role
  behavior (spike 1 criterion 3 — the worker's database credential must be narrower than
  the web application's, `architect`'s own flag, not yet named as a verification item in
  the plan); elevated/system access (the bypass path must be narrow, explicit, logged —
  Security Floor item 10); connection-pool contamination (a tenant-context value must not
  leak across pooled-connection reuse — spike 1's own core finding).
- **Database**: migration safety (up/down/up reversibility, already named); transaction
  rollback (a failed module-level service call must actually roll back, not partially
  commit — directly tests correction 8's transaction-boundary fix); tenant-scoped query
  behavior; the DB-level isolation backstop itself functioning; schema enrollment
  (already named).
- **Jobs**: claim/lease/fence (already named, spike 4); stale worker (lease expiry and
  reclaim); retry exhaustion; recurring checkpoint recovery (incremental, not
  batch-completion-only — spike 4's own "critically dependent" finding); duplicate
  external-effect protection (idempotent effect application).
- **Security**: secret handling (Security Floor item 7 — no secret committed, logged, or
  exposed; cheap to add as a lint/grep-based CI check, not previously named); production
  boot invariants (already named — "boot gate refuses three specific bad configs"; the
  plan should name **which** three, currently unspecified); security headers/config
  (Security Floor item 14 — CSP, `frame-ancestors`/`X-Frame-Options`, HSTS,
  `X-Content-Type-Options`); privileged/admin access (Security Floor item 10);
  auth/authz boundaries (once the auth mechanism exists).
- **Time**: DST forward/backward transitions (Phase 8A §22 requirement, missing from the
  plan — added here); nonexistent local time (per correction 6's gate); ambiguous local
  time (same); recurring occurrences across transitions (Phase 8A's own Adversarial
  Correction Pass §6 names this as a *distinct*, separately-unresolved UNKNOWN from the
  single-construction case — not previously named anywhere in this plan).
- **Booking**: concurrent identical booking; overlap; buffers; cancellation/rebook;
  concurrent reschedule; recurring partial failure; recurring concurrent overlap. (None
  of these apply at foundation time specifically — listed here for completeness per
  instruction, and to be carried into the first booking-domain work package, not
  foundation's own verification set.)

**Stated explicitly, per instruction**: every completed high-risk foundation slice must
produce a real Verification Evidence Record per the [[Verification Evidence
Format|Verification Evidence Format]] — an agent's own report that it "reviewed" a change
is never a substitute for an actual evidence record with named methods and results. No
Verification Evidence Record was proposed for the foundation work itself in the plan's
original text; this is a **gap**, not merely a missing item on a checklist — WP0 touches
several HIGH-risk-adjacent domains (secrets/config, deployment shape, migrations) per the
Authority/Risk/Routing Model §2.1, and should produce its own record.

### (12) Provenance / WP0–WP9 preservation

The plan's own §6 already states honestly that `planner`'s full WP0–WP9 breakdown exists
only in this session's transcript, not durably in the vault — a real, acknowledged gap.
Per this correction's instruction, that gap is now closed: see the new companion file
[[Calquartz — Foundation Work Packages (WP0–WP9)|Calquartz — Foundation Work Packages
(WP0–WP9)]], created in this same pass. **That file states its own provenance plainly**:
it is a reconstruction, not a verbatim transcript recovery — the original live
`planner`-agent transcript is not durably stored anywhere in this vault, and no claim is
made that one exists. The reconstruction draws on this plan's own §2–§5 content (which
summarizes, but does not fully restate, `planner`'s original breakdown) plus this pass's
own re-derivation from ADR-001, Phase 8A, the Security Floor, and the Booking Correctness
process wherever the plan's summary was incomplete. Each work package there is labeled
FACT (spike/ADR/Phase-8A-derived), RECOMMENDATION (this pass's own judgment), gate
(human-required), or UNKNOWN, per instruction — not presented as if it were `planner`'s
original, unrecoverable output.

### (13) Evidence preservation honesty

Restated explicitly, extending §6's existing honesty: every load-bearing claim in this
plan sourced from the live `planner`/`architect` agent outputs is **specialist-derived
and summarized, not verbatim**, except where §4 already uses direct quotation (marked as
such there). No full transcript of either specialist invocation is durably stored
anywhere in this vault — this pass confirms that gap exists (checked directly: no such
transcript file was found under `wiki/projects/001-calendar-os/` or
`raw/projects/001-calendar-os/`) rather than assuming a location for it. The new
WP0–WP9 companion file (correction 12) is the closest durable proxy now available, and
is labeled as a reconstruction, not a recovered transcript.

### (14) Final complexity audit

Applying the charter's own standard (project `CLAUDE.md` §44, "optimize for maximum
useful capability per unit of complexity") directly to §2's foundation structure:

**Verdict**: the plan is not the giant universal framework the charter warns against —
nothing proposed is microservices-before-need or speculative multi-tenancy-of-multi-
tenancy. But it does carry more standing `packages/*` infrastructure than its own cited
evidence individually justifies, per correction 7's table: two packages
(`kernel`, `obs`) have **no cited requirement at all** and are generic-framework
instincts, not Calquartz-derived necessities; three more (`config`, `time`, `contracts`)
have a real underlying requirement but a premature *package-level* commitment ahead of a
second real consumer. **Each flagged abstraction with no concrete first-feature consumer
is challenged/deferred, not built now** — per correction 7's per-package table. The
preferred order, restated explicitly per instruction: **thin generic foundation →
Calquartz-specific implementation → extract proven patterns later**, not a
universal framework built before the first real feature exists. `apps/*`, `migrations/`,
the integration-test harness, `packages/db`, and `packages/jobs`'s protocol scope all
survive this audit because each has a named, cited, spike- or ADR-derived requirement —
they are not "built because it looks tidy."

### (15) Kept explicitly open — restated, not resolved

The following remain genuinely UNKNOWN and this pass does **not** resolve any of them,
consistent with ADR-001's and Phase 8A's own non-decision lists: RLS vs. an alternative
database-level tenant-isolation backstop; the recurring-series atomicity model; the
ambiguous/nonexistent local-time final product policy (a default may be
**recommended**, per correction 6, never decided here); the auth library; whether a job
library (`pg-boss`/`graphile-worker`) eventually replaces the application-owned protocol;
the calendar provider; the notification provider; seats/group bookings; round-robin; a
public developer API; embeds; the canonical final product/brand name (beyond the accepted
working identifier, correction 4). Scaffolding must not be built in a way that hides any
of these behind an implicit choice — correction 5's naming fix and correction 7's
package-vs-file downgrades exist specifically to keep that true.

### (16) WP0 authorization scope — restated precisely

Not a blanket "approve WP0." Stated exactly, per instruction:

**Potentially allowed now, before the tenant-isolation mechanism is finalized**:
repository-local structure (local git init, directory layout); non-tenant tooling
(boundary-lint/dependency-cruiser configuration, `packages/kernel`-as-file,
`packages/obs`-as-file, `packages/config`-as-convention); reversible configuration;
test-harness foundations that do not encode a specific isolation mechanism (the
integration-test harness's *existence* — real-Postgres, template-clone — is
foundation-appropriate; its specific reliance, if any, on RLS-specific behavior is
UNKNOWN per correction 5 and should be either confirmed mechanism-neutral or corrected
before being treated as settled); documentation/provenance (this correction pass and its
companion file); boundary checks (`packages/db`'s pool + tenant-context primitive under
mechanism-neutral naming per correction 5 — spike-1-justified regardless of which
mechanism is eventually selected); `packages/jobs`'s protocol scaffolding (spike-4-
justified, caveated per correction 7b); the durable-job migration, confirmed (not
decided) against spike 4's specification per correction 2.

**Must wait**:
- **Tenant-scoped schema** depending on the final isolation mechanism — any migration
  creating a real tenant-scoped table, until blocker 1 (tenant-isolation mechanism) is
  resolved by the owner.
- **Booking-time implementation** — any code constructing or interpreting local
  wall-clock booking times, until the local-time policy (correction 6) is resolved or an
  explicit owner-accepted default is recorded.
- **Remote repository/CI activation** — any push to a hosted remote or any CI run, until
  blocker 3 (repository hosting/CI provider) is resolved by the owner.

Blockers 4 (licensing) and 5 (product name) are **closed** by the plan's own already-
accepted defaults (corrections 3–4) and gate nothing. No other item gates any WP0 work
under this restated scope.

### Related

- [[Calquartz — Application Foundation Implementation Plan — Adversarial Review|the
  existing Adversarial Review]] — reviewed directly in this pass; its findings are
  substantially corroborated by this pass's own independent re-reading of the
  authoritative sources, not merely adopted
- [[Calquartz — Foundation Work Packages (WP0–WP9)|Calquartz — Foundation Work Packages
  (WP0–WP9)]] — the new companion file created per correction 12

## Post-Audit Correction Pass — 2026-09-09

Scope note: this is a **second, later same-day, review-and-correction pass**, following
the [[Calquartz — WP0 Pre-Authorization Evidence Audit|WP0 Pre-Authorization Evidence
Audit]]'s independent re-verification of the Adversarial Correction Pass above. Same
discipline as that pass: nothing in §1–§7 or the Adversarial Correction Pass is deleted
or rewritten; this section applies, additively, the specific corrections the audit's
§9(1) identified as still outstanding — recommendations that existed only on paper, not
yet reflected in this document's own prescriptive text (§2, and the WP0–WP9 companion
file). The audit itself (and the existing Adversarial Review) remain unmodified,
append-only historical artifacts, per this vault's own schema.

**(A) DB tenant-context seam — naming applied.** §2's repository tree above now names
the seam `withTenantContext`/`withElevatedAccess` directly (not merely as a
recommendation in correction 5) and states explicitly that WP0 must not implement `SET
LOCAL`, RLS-specific policies/roles, `BYPASSRLS`, or any other mechanism-specific
tenant-isolation implementation. RLS remains a POSSIBLE, spike-1-validated *candidate*,
not a decision — unchanged from correction 5/ADR-001's explicit non-decision.

**(B) Boot-gate refusal conditions — named explicitly.** Wherever this plan references
the boot gate generically (§4, correction 11), the three refusal conditions are now
named, per the audit's §9(1) requirement and Security Floor item 15's own evidence base:
(a) a missing, placeholder, or insufficiently-random signing secret; (b) an
unverified-TLS database URL; (c) an active demo/local provider fallback. Naming these
conditions here is documentation only — it does not itself build the boot gate. The
gate's actual implementation stays inside its own security-gated work package (WP8 in
the companion WP0–WP9 file), not WP0's local-scaffolding scope.

**(C) `packages/contracts` — transport-neutrality constraint stated in §2.** §2 now
states directly, not only in correction 7a's discussion, that the package must contain
domain-shaped, transport-neutral types only, and must not encode REST, tRPC, GraphQL,
HTTP-status, router, or other transport-specific assumptions. The API paradigm remains
UNKNOWN (Phase 8A §19) — this correction constrains the package's shape; it does not
select among REST/tRPC/GraphQL.

**(D) Integration-test harness — UNKNOWN stated in §2, not assumed proven.** §2 now
states directly that the harness's exact implementation shape is UNKNOWN until harness
code exists and is checked — consistent with the audit's §4/§8 finding that no harness
code exists yet and mechanism-neutrality is unconfirmed. The real-Postgres/template-
clone requirement (Phase 8A §12) is preserved. The plan must not silently introduce
RLS-specific test mechanics anywhere in the harness's description — none is present
after this correction.

**(E) `packages/jobs` — permanent-ownership caveat stated in §2.** §2 now carries the
correction-7b caveat directly at its point of definition, not only in the correction-pass
discussion: the claim/lease/fence/`SKIP LOCKED`/checkpoint protocol is spike-4-validated
FACT; whether the application permanently owns a hand-rolled job framework, versus later
adopting a library (`pg-boss`, `graphile-worker`), is NOT decided (Phase 8A's own
Correction Pass). No language anywhere in this plan or the WP0–WP9 file claims spike 4
proved the hand-rolled approach superior to a library — checked directly in this pass
against both files' full text; none was found (consistent with correction 9's own prior
finding).

**(F) Playwright — relocated out of WP0.** §2's `test/e2e/` entry now states explicitly
that Playwright's *implementation/scaffolding* is deferred out of WP0 (no real page
exists yet to exercise it — the same "undemonstrated abstraction" test correction 7
already applied to `packages/kernel`/`packages/obs`) and relocated to the work package
that delivers the first real page. **This does not delete the eventual Playwright
requirement** — Phase 8A §18's DECISION that Playwright is part of the approved stack is
unaffected; only *when* its scaffolding is built moves.

### Minimality re-check — WP0 tree, re-derived independently

Per this pass's own instruction, every proposed WP0 path is independently reassessed
against the authoritative sources (ADR-001, Phase 8A, the Security Floor, the four
executed spikes), not deferred to any prior table's conclusion, though the result
substantially agrees with both the Adversarial Correction Pass's own table (correction 7)
and the Evidence Audit's §6 table:

| Path | Independent re-assessment | Classification |
|---|---|---|
| `packages/kernel` | No spike, ADR-001 clause, or Phase 8A selection names it anywhere; no concrete first consumer identified | **PREMATURE** — downgrade to a local file, promote only once a second module duplicates it |
| `packages/obs` | Same — no citation found anywhere in the authoritative sources | **PREMATURE** — downgrade to a local file, clearest case |
| `packages/config` | Security Floor item 15 requires *some* boot-gate mechanism, but not a standing package boundary ahead of measured duplication across `apps/api`/`apps/web`/`apps/worker` | **FIRST-FEATURE-REQUIRED** — keep as a documented per-entrypoint convention now; the boot-gate *requirement* is foundation-appropriate, the *package* boundary is not yet earned |
| `packages/time` | Spike 3 requires wall-clock-correct construction and explicit ambiguous/nonexistent-time surfacing, but a package-level API shape would fix a return-type contract before the local-time product policy is decided | **FIRST-FEATURE-REQUIRED** — local module with explicit `{ok, value} \| {ok: false, reason}` result types now; package boundary deferred to the first booking-domain slice (WP6) |
| `packages/contracts` | Phase 8A §18 approves end-to-end type sharing as a stated reason for the Next.js/Fastify pairing (DECISION); the package boundary itself has moderate justification, and correction (C) above now fixes its transport-neutral shape | **FOUNDATION-JUSTIFIED-BUT-REVERSIBLE** — keep as a package, kept minimal (types-only, no runtime logic) |
| `packages/db` | Spike 1 requires a pool + tenant-context seam; sole pool constructor is the point of the component | **FOUNDATION-REQUIRED** — keep as a package, mechanism-neutral naming per correction (A) |
| `packages/jobs` | Spike 4 requires the protocol; scope strictly limited to protocol only, permanent-ownership question explicitly not decided (correction E) | **FOUNDATION-REQUIRED**, scope-limited to protocol — not a domain-handler framework |
| Playwright | No real page exists yet to exercise; front-loads infrastructure ahead of anything it would test | **PREMATURE** for WP0 specifically — relocated per correction (F) to the first-real-page work package (a FIRST-FEATURE-REQUIRED item there, not eliminated) |
| Docker (three entrypoints, local build/config only) | Required eventually for deployment (ADR-001 process count + Phase 8A §5 split rationale), but WP0's local-only scope does not need it to run anything | **FOUNDATION-JUSTIFIED-BUT-REVERSIBLE** |
| Integration-test harness (real-Postgres, template-clone) | Phase 8A §12 names this as required testing methodology; removing it would mean the first vertical slice's correctness claims rest on mocks, contrary to why spikes 1/2/4 used real Postgres — but its exact implementation shape is UNKNOWN until built (correction D) | **FOUNDATION-REQUIRED** for the harness's *existence*; its specific shape is UNKNOWN, not yet FACT, until checked against real code |

No package is preserved merely because a prior pass proposed it — each row above was
independently checked against its own citable requirement (or lack of one), consistent
with this pass's own instruction to not be deferential to the existing audit while being
informed by it.

### Related (post-audit pass)

- [[Calquartz — WP0 Pre-Authorization Evidence Audit|WP0 Pre-Authorization Evidence
  Audit]] — the audit whose §9(1) findings this pass corrects for; left unmodified, per
  this pass's own scope boundary
- [[Calquartz — Foundation Work Packages (WP0–WP9)|Calquartz — Foundation Work Packages
  (WP0–WP9)]] — updated in the same pass to carry these corrections into its own WP0
  scope text

## Related

- [[../decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]]
- [[Phase 8A — Implementation Stack Selection]]
- [[Authority, Risk, and Routing Model]] · [[Verification Evidence Format]]
- [[../architecture/Security Floor|Security Floor]]
- [[../reuse/Reuse Audit Summary|Reuse Audit Summary]]
- [[Calquartz — Application Foundation Implementation Plan — Adversarial Review|Adversarial Review]]
- [[Calquartz — Foundation Work Packages (WP0–WP9)]]
