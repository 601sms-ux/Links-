---
type: synthesis
created: 2026-09-09
updated: 2026-09-09
sources: ["wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation Implementation Plan.md", "wiki/projects/001-calendar-os/decisions/ADR-001 — A-B1 Modular Monolith Architecture.md", "wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack Selection.md", "wiki/projects/001-calendar-os/architecture/Security Floor.md", "wiki/projects/001-calendar-os/engineering/Authority, Risk, and Routing Model.md", "wiki/projects/001-calendar-os/engineering/Verification Evidence Format.md", "wiki/projects/001-calendar-os/reuse/Reuse Audit Summary.md", "wiki/projects/001-calendar-os/Calquartz — Pre-Approval Technical Spikes.md", "wiki/projects/001-calendar-os/questions/Architecture Open Questions.md"]
tags: [engineering, review, adversarial, foundation, calendar-os, calquartz]
confidence: medium
provenance: hand-authored adversarial review by a reviewing Claude Code session, 2026-09-09; NOT a specialist-agent output (not planner, not architect); reviews the joint planner+architect synthesis document adversarially, including architect's own correction to planner's original proposal
classification: RECOMMENDATION — a review verdict, not itself an architecture, stack, or implementation decision. Nothing in this document upgrades any UNKNOWN to DECISION.
scope: calquartz-specific
status: DRAFT — review only, no implementation authorized
---

# Calquartz — Application Foundation Implementation Plan — Adversarial Review

**Role of this document.** This is an adversarial second-pass review of
[[Calquartz — Application Foundation Implementation Plan|the Foundation Implementation
Plan]] (itself a synthesis of live `planner` and `architect` agent output, 2026-09-09).
It does not modify that plan, ADR-001, Phase 8A, any governance document, or any agent
file. It creates no application code, repository, migration, or configuration. Where this
review disagrees with the plan, the plan is not silently fixed — the disagreement is
recorded here for the owner to weigh.

**Epistemic-status key used throughout**: FACT (demonstrated by a spike or read directly
from a governance document) / INFERENCE (this reviewer's own reasoning from FACTs) /
RECOMMENDATION (a proposal, not binding) / DECISION (owner-approved, binding) / UNKNOWN
(genuinely unresolved). Provenance-category key: (1) demonstrated by a prior spike,
(2) explicitly required by ADR-001, (3) selected in Phase 8A, (4) proposed by `planner`,
(5) proposed/corrected by `architect`, (6) this reviewer's own inference, (7) unresolved
question. A claim's presence in the plan under review is never treated here as proof of
its correctness — `planner` and `architect` are live agent outputs being reviewed, not an
approved blueprint.

---

## Review 1 — Is the foundation actually minimal?

The plan proposes eleven structural elements before any feature exists: `apps/api`,
`apps/web`, `apps/worker`, `modules/`, `packages/contracts`, `packages/kernel`,
`packages/config`, `packages/db`, `packages/jobs`, `packages/time`, `packages/obs`, plus
`migrations/`, two test tiers, and a lint-enforced boundary layer (dependency-cruiser).
Each assessed on the required five questions.

| Package | Concrete requirement | Evidence | Buildable-without? | Constrains an open decision? | Undemonstrated abstraction? | Minimum viable alternative |
|---|---|---|---|---|---|---|
| `apps/api` | ADR-001 requires a web process (2) | ADR-001 | No — A/B1's process split is DECISION | No | No | None smaller — this is the architecture |
| `apps/web` | ADR-001 requires a web process; Phase 8A selects Next.js (2)(3) | ADR-001, Phase 8A §18 | No | No | No | — |
| `apps/worker` | ADR-001 requires a worker process; spike 4 requires durable jobs to run somewhere (1)(2) | ADR-001, spike 4 | No | No | No | — |
| `modules/` (empty) | None yet — no feature exists (6) | — | Yes, trivially — an empty directory is not an abstraction, it is a placeholder | No | **Yes, mildly**: naming it `modules/` presumes a module-per-domain shape before any domain module exists. Harmless as an empty folder, but the plan's own boundary-lint rules (`modules/*` importing rules) treat it as load-bearing structure, which is a real commitment, not a placeholder | Skip until WP1's first real module; create the folder then |
| `packages/contracts` | Structural necessity flagged by `architect` — without it, `apps/web` importing `packages/db`/`modules/*` is unavoidable given the boundary rule (5) | Plan §3, correction 2 | **Arguable.** A single vertical slice could instead let `apps/web` call a versioned HTTP API and generate/share types via an OpenAPI or tRPC-style codegen step rather than a hand-maintained types package — Phase 8A itself left the API paradigm (REST/tRPC/GraphQL) explicitly UNKNOWN (§19) | **Yes, quietly.** A `packages/contracts` package with hand-written shared types is a specific implementation of type-sharing; committing to it before the API paradigm is chosen is a smaller version of the same "structure pre-decides an open question" problem `architect` correctly caught for transaction ownership (Review 4) | Partially — its necessity is asserted as "the only way to avoid the contradiction," but that framing assumes the boundary rule and the type-sharing goal are both fixed; a colocated types file inside `apps/api` reachable only by build-time codegen would satisfy the same boundary rule without a standing package | Skip the package; add a lightweight `openapi`/schema-generation step at WP1, or accept `apps/web` calling a typed HTTP client generated at build time, and only promote to a hand-maintained package if that friction is measured, not assumed |
| `packages/kernel` | None named by any spike, ADR-001, or Phase 8A (6) | — | Yes | No | **Yes** — "errors, result types, ids" is a generic utility layer with no Calquartz-specific requirement behind it | Could live as a single `lib/` file inside the first module that needs it and be extracted once a second module needs the same thing (the standard "extract on second use," not before) |
| `packages/config` | Phase 8A §15 (Security Floor item 15, boot-gate validation) requires *some* boot-time validation mechanism (2)(3) | Security Floor item 15 | Partially — the boot-gate requirement is real, but "the single place any secret is read" as a standing package is a design choice among several that would satisfy item 15 (a single `config.ts` module inside `apps/api` would too) | No | Mild — "single place any secret is read" is a good discipline but packaging it before there is more than one app process consuming it is unproven value | A single validated config module per app entrypoint, promoted to a package only once `apps/worker` and `apps/api` are both real and duplicating validation logic |
| `packages/db` | Spike 1 requires a pool + tenant-context seam (1)(2) | Spike 1 evidence | No — the seam itself is spike-validated | **Yes — see Review 3.** The seam's *existence* is justified; its *shape* (a `withTenant`/`withSystem` API) is not spike-validated, only the underlying RLS+`SET LOCAL` mechanism is | The `withTenant` abstraction itself is `architect`'s and `planner`'s invention, not spike output — spike 1 tested raw `pg` + `SET LOCAL` directly (Phase 8A §6, "spike 1 used raw `pg` directly... it did not test Prisma" — nor did it test any named abstraction layer) | A single exported function wrapping "begin transaction, `SET LOCAL`, run callback, commit" — smaller than a package, but the package boundary itself is defensible given it is the sole pool constructor (ADR-001-adjacent discipline) |
| `packages/jobs` | Spike 4 validates a protocol (1) | Spike 4 evidence | No — protocol needed | No, if scoped to protocol only as the plan states | The plan already separates protocol from implementation reasonably; risk is scope creep beyond "claim/lease/fence" into handler logic | As specified is close to minimal already |
| `packages/time` | Spike 3 requires explicit surfacing of ambiguous/nonexistent time (1) | Spike 3 evidence | Could be a single module, not a package, at foundation time — no second consumer yet | **Yes — flagged correctly by `architect` already; see Review 5.** The real risk is the *policy* being baked into the package's return-type shape before the product policy (Phase 8A §8/§20, still UNKNOWN) is decided | Package boundary is premature; the surfacing discipline is not | A single `time.ts` file with explicit `Result`-shaped returns, not a package, until a second module needs to import it independently |
| `packages/obs` | Not spike-tested, not ADR-001-required, not Phase 8A-selected (6) | — | Yes | No | **Yes, clearly premature.** "Allowlist-by-construction logger" is a good idea (matches Security Floor item 8) but nothing requires it exist as a standing package before a single log line has ever been written in anger | A single `logger.ts` using an off-the-shelf structured logger with a manually-maintained allowlist wrapper function; promote to a package only once `apps/worker` needs the identical thing `apps/api` already has |
| `migrations/` | ADR-001 + Phase 8A require SQL-first migrations (2)(3) | ADR-001, Phase 8A §18 | No | No | No | — |
| Integration + e2e test tiers | Phase 8A §12 requires integration-against-real-Postgres and Playwright e2e (3) | Phase 8A §18 | No, in principle — but **e2e Playwright infrastructure before a single page exists is arguable overreach for WP0 specifically**, as distinct from "the testing strategy names Playwright" | No | Mild — standing up an e2e harness before any UI exists front-loads infrastructure a first vertical slice may not yet exercise | Defer Playwright scaffolding to the work package that produces the first real page; integration-test harness (template-DB-clone) is genuinely foundation-level and should stay |
| Docker (3 entrypoints) | ADR-001 + Phase 8A require single image, three entrypoints (2)(3) | ADR-001, Phase 8A §18 | Arguable — see Review 6, three *entrypoints* is not the same commitment as three *processes running continuously in dev* | Possibly — see Review 6 | No | — |
| dependency-cruiser / lint boundary enforcement | Not spike-tested, not ADR-001-required by name; `architect`'s own correction rests on this being mechanically enforced rather than conventional (5)(6) | Plan §2, "enforced by lint/dependency-cruiser, not convention alone" | This is genuinely valuable given the project's own SnagTime forensic finding (hand-maintained enrolment list failing open, [[../reuse/Reuse Audit Summary]] A.3) — the project has direct evidence that "enforce by convention" fails in exactly this kind of codebase. **Not premature** | No | No | Keep — this is one of the plan's better-justified elements, and it is cheap (one dev dependency, one config file) |

**Overall verdict on Review 1**: `apps/*`, `migrations/`, the integration test harness, and
the boundary-lint tooling are well-justified (categories 1–3 provenance). `packages/db`,
`packages/jobs`, `packages/contracts`, `packages/config`, `packages/time` are
*directionally* justified but each smuggles in a specific implementation shape beyond what
its justifying evidence actually covers (category 6 — this reviewer's inference, echoing
but sharpening `architect`'s own flag on `packages/time`). `packages/kernel` and
`packages/obs` have **no cited requirement at all** — they are generic-framework instincts,
not Calquartz-derived necessities, and are the clearest candidates for deferral. **Yes, the
seven-package `packages/*` layer, taken together with `modules/`, `apps/*`, and two full
test tiers, constitutes more internal framework than a first vertical slice needs.** The
plan is not wildly over-engineered — nothing here is microservices-before-need or
speculative multi-tenancy-of-multi-tenancy — but at least two packages (`kernel`, `obs`)
and one premature package-vs-module boundary each (`contracts`, `config`, `time`) should be
downgraded to single files or deferred, per the "minimum viable alternative" column above.

---

## Review 2 — Attack the five blockers

| # | Blocker | Genuinely required before genesis/schema/first-slice? | Could work proceed unresolved? | Isolable behind a reversible boundary? | Human gate or just a recommendation? | Documented default sufficient? | Self-contradiction? |
|---|---|---|---|---|---|---|---|
| 1 | Tenant-isolation mechanism | Required before **schema freeze**, not before genesis. `withTenant` scaffolding is explicitly buildable now per the plan itself | **Yes** — repository genesis, `packages/kernel`-equivalent work, CI setup, and even `packages/config` do not require this to be resolved | Yes — the plan's own framing ("scaffolding... buildable now; the mechanism itself is not Claude's to select") is exactly a reversible boundary. **This blocker is correctly scoped as blocking schema/WP-with-tables, not blocking WP0 generically** — the plan's own §7 conflates this by saying "WP0 (repository genesis) cannot begin until blockers 1–5... are resolved," which is stricter than blocker 1's own text supports | Genuinely a human gate (2) — ADR-001 leaves this an explicit non-decision, RLS is a candidate not a selection | No — RLS specifically requires "sole selection, not defaulted" per Phase 8A's own restraint | **Yes — see below.** The plan's §7 blanket statement is inconsistent with its own §5 item 1 reasoning |
| 2 | Durable-job table schema | Required before the **jobs package's own migration**, not before genesis or before `packages/time`/`packages/kernel`/CI work | Yes | Yes | Yes, per the routing table's own "touches a shape a prior Second Brain document assumed" rule (Authority/Risk/Routing §3.2) | Spike 4 evidence already specifies most of the required columns (claim, lease, fencing token, effect log, checkpoint) — arguably this is closer to "write it down and confirm" than a genuine open design question. **This blocker may be overclassified**: the *protocol* is FACT (spike 4); only the exact column names/types are undecided, which is closer to an implementation detail than a human-gate-worthy decision | Not contradictory, but likely overclassified as HIGH when spike 4's specificity suggests a much narrower confirm-not-decide step |
| 3 | Repo hosting / CI provider | Required before **first commit exists somewhere durable and before CI runs**, not before local scaffolding work, dependency selection, or writing `packages/kernel`/`packages/time`/`packages/config` locally | Yes, substantially — a local git repo with no remote can absorb a large amount of WP0 work before hosting is chosen | Yes | This is an operational/deployment decision (routing table: "Deployment... Human gate: yes, always"), correctly a human gate, but its scope is narrower than "all of WP0" | A working local default (init a repo locally, defer remote/CI choice) is sufficient for most of WP0 | The plan's §7 treats this identically to blocker 1, which is a category error: repo hosting affects *where code lives*, not *what the code is* |
| 4 | Licensing clearance (Cal.diy `csp.ts`, SnagTime `assertProductionRuntimeSecurity`) | **Not a foundation blocker at all.** Neither pattern is cited anywhere in §2's package list as required for WP0. This is a **reuse-specific gate**, scoped to the moment either file would be copied verbatim, not to repository genesis | Yes, entirely — foundation work (kernel, config skeleton, contracts, db pool, jobs protocol, time module) touches neither named candidate | Trivially — the plan's own stated default ("if unresolved: re-derive, copy nothing") **is** the reversible boundary, already documented | Arguably not a human gate at all if the default is accepted — "re-derive" requires no owner input | The plan already supplies a safe default and states it explicitly | **Yes, direct self-contradiction.** §5 item 4 states a working default that fully resolves the practical question ("copy nothing"), then §7 still lists it among blockers that stop WP0. If the default is accepted, this is not a blocker; if the default is not accepted, it blocks only the two specific files' authorship, not WP0 as a whole |
| 5 | Canonical product name | **Not a genesis blocker.** The plan's own §5 item 5 supplies a working default (`calquartz`) and accepts rename cost later | Yes, trivially | Yes — a slug rename touches package.json, image tags, DB name, role names; annoying but mechanical, explicitly acknowledged by the plan itself as an accepted cost | Not really a human gate if the default is accepted — this is closer to the "smaller items Claude can resolve autonomously" category the plan itself names two paragraphs later for evidence-record location | Yes, and the plan states it | **Yes — same pattern as blocker 4.** The plan supplies its own working default, then still lists the item among things that "cannot begin until... resolved" |

**Conclusion**: of five named blockers, only **two (1: tenant-isolation mechanism scoped to
schema-touching work; 3: repo hosting/CI scoped to remote-commit/CI-touching work)** are
genuine human gates that block *some* WP0 work. Blocker 2 is likely overclassified as a
full human-decision gate when spike 4 already supplies most of the answer. **Blockers 4 and
5 are not blockers at all as the plan itself has written them** — each carries an
explicitly documented, already-accepted default that fully resolves the practical question
for foundation purposes. The plan's §7 sentence ("WP0... cannot begin until blockers 1–5...
are resolved") **contradicts** its own §5, which supplies working defaults for two of the
five and scopes a third narrowly. This is the review's single clearest finding of a
genuinely misclassified blocker set: **the "all five block WP0" framing in §7 is not
supported by the plan's own §5 text**, and should be corrected to something like "blockers
1 and 3 gate specific later steps within WP0 (schema-touching work, remote/CI setup);
blocker 2 is a narrow confirm-not-decide step; blockers 4 and 5 are already resolved by
documented default and should not gate anything unless the owner explicitly rejects the
stated default."

---

## Review 3 — Attack the tenant-isolation decision

RLS is spike-1-validated as a *candidate*, not selected. The plan proposes
`packages/db` "owns... the tenant-context (`withTenant`/`withSystem`) seam." This needs
close inspection for architecture-by-scaffolding.

**What is genuinely mechanism-agnostic**: a function signature like
`withTenant(tenantId, callback)` that opens a transaction and hands the callback a
connection is not RLS-specific on its face — an application-layer-scoping mechanism could
implement the same signature by filtering queries instead of setting a session variable.

**What quietly narrows toward RLS anyway**: the plan names the seam `withTenant`/
`withSystem` — a two-mode split (tenant-scoped vs. system/bypass) that maps unusually
precisely onto RLS's own two-role model (an ordinary role subject to policies, plus an
explicit `BYPASSRLS` role spike 1 tested). An application-layer-scoping alternative would
more naturally need a *query-rewriting* seam (inject `WHERE tenant_id = $1` universally),
not a *transaction-context* seam — the two mechanisms are not interchangeable behind an
identical-looking wrapper. **This is architecture-by-scaffolding, mild but real**: naming
the seam after RLS's own two-role shape, before RLS is selected, makes the *other*
candidate (documented application-layer scoping with compensating controls, named in spike
1's own "Architecture decision affected" framing) harder to retrofit later without a
rename/reshape, not merely a swap of implementation inside the same interface.

**Table conventions and test harness**: the plan's integration-test harness is described as
"template-DB-clone-per-file, not transaction-wrapped — see §4, planner finding." This
detail is *itself* a quiet RLS-leaning choice: a transaction-wrapped test harness (rollback
after each test) is the natural choice under most Postgres testing patterns, and choosing
template-clone-per-file instead is worth asking why — the likely reason (unstated in the
plan) is that RLS policies interact with `SET LOCAL` inside the same transaction, and a
transaction-wrapped test harness would make it harder to test "does the policy actually
apply across a *simulated* new connection/pool-reuse boundary" the way spike 1 needed to.
If that is the actual reason, it is a second, unstated instance of the mechanism leaking
into the scaffolding. **INFERENCE, not confirmed** — the plan does not state this
reasoning, so this reviewer cannot confirm it, only flag it as the kind of decision that
deserves an explicit "we chose this test harness shape for reason X, independent of which
isolation mechanism is eventually selected" justification, which is currently missing.

**Migrations and "table conventions"**: the plan does not specify column-naming or
`tenant_id`-placement conventions in detail (good — that is correctly left open), but the
catalog-driven fail-closed enrolment check spike 1 validated ("derive the enrolled-table
list from the schema at build time") is itself schema-shape-agnostic and does not
pre-decide RLS. No finding here.

**Verdict**: the `withTenant`/`withSystem` naming and the template-clone test-harness
choice are the two concrete places this reviewer finds RLS being architecturally
pre-selected through scaffolding, contrary to ADR-001's and Phase 8A's explicit UNKNOWN
status for this mechanism. Neither is a severe violation — both are cheaply renamed/reshaped
before code exists — but both should be corrected *before* WP0, specifically because Review
2 above already flags foundation-vs-schema-touching work as separable: if `packages/db`'s
seam is going to be built at foundation time regardless, it should be built under
mechanism-neutral naming (e.g. `withTenantContext`/`withElevatedAccess` rather than
`withTenant`/`withSystem`) so that selecting an alternative to RLS later is a
reason-about-the-implementation problem, not also a rename-every-call-site problem.

---

## Review 4 — Attack the transaction-boundary correction

`architect`'s correction (moving transaction ownership from `apps/api` to module-level
application services) is presented as *un*-deciding the recurring-series atomicity
question. Attacking that claim on its own terms:

**Does "module-level services own transaction boundaries" itself pre-decide something?**
Partially, yes. It decides that **the transaction boundary is drawn at the
application-service call, not at the HTTP request** — which is a smaller decision than
"all-or-nothing vs. partial success," but it is still a decision, and it is still a
decision ADR-001/Phase 8A did not make explicitly. Spike 2's own finding ("both atomicity
models... turn out to be an application-transaction-boundary choice, not a
constraint-mechanism property") establishes that the exclusion constraint is agnostic to
*which* atomicity model is chosen, but says nothing about *where in the call stack* the
transaction boundary should sit. `architect`'s fix answers a real structural question
(HTTP-layer vs. service-layer transaction ownership) that genuinely was underspecified, and
that fix is defensible on its own terms — it does not, in fact, foreclose either the
all-or-nothing or partial-success model, since a module-level service can implement either
shape internally. **This correction survives the attack**: it removes one hidden decision
(HTTP-layer transaction scoping making partial-success awkward to express) without
introducing a comparably-sized new one.

**Is a genuinely mechanism-agnostic foundation possible at all, or is some commitment
unavoidable?** No foundation is perfectly neutral — every line of scaffolding embeds some
assumption. The honest question is whether the *specific* assumptions embedded are (a)
reversible and (b) narrower than the decision they're adjacent to. By that standard,
`architect`'s correction is a good example of minimizing embedded assumption (it commits to
"who calls the transaction API," not "what the transaction does"), while the
`withTenant`/`withSystem` naming in Review 3 is a worse example (it commits to a shape that
leaks the *specific candidate mechanism*, not just "who calls the API"). **The
distinguishing test this review proposes**: a foundation decision is acceptably neutral if
renaming/reshaping it later touches only the package that made it; it is not neutral if
renaming it later would require touching every call site across every future module. By
that test, `architect`'s transaction-boundary fix passes; the tenant-context naming in
Review 3 is closer to failing (every module that ever calls `withTenant` would need
updating if the seam's actual shape changes when a non-RLS mechanism is selected).

---

## Review 5 — Attack the package boundaries

**`packages/db`**: "pool+Kysely+tenant-seam owned centrally, modules own their own SQL."
Coherent in principle — centralizing the pool and the transaction/tenant-context primitive
while decentralizing query-writing avoids the horizontal-layer problem `architect`
correctly named (a central query layer would cut across every module's data, straining the
extraction seams ADR-001 wants preserved). The genuine risk is enforcement: nothing in the
plan's own boundary-lint description ("only `packages/db` may construct a pg pool or issue
a raw query") actually prevents a module from writing tenant-unsafe SQL *using* the shared
pool once it has it — the lint rule stops modules from bypassing `packages/db` to get a raw
connection, but does not stop a module from getting a connection via the seam and then
issuing a query that never calls `withTenant`. This gap is structurally identical to the
one Phase 8A's own §21 flagged for Kysely generally ("a careless call site could still
issue a query outside the transaction that set `SET LOCAL` tenant context — the tool does
not prevent this by construction"). **Not a new problem this plan introduces, but a known
one this plan does not name or plan a mitigation for**, and should.

**`packages/contracts`**: see Review 1 — necessary given the stated boundary rule and
type-sharing goal, but both of those are choices, not requirements; a codegen-based
alternative exists and was not evaluated in the plan.

**`packages/time`**: what's actually required by spike 3 is narrower than a package —
spike 3 requires (a) using wall-clock-correct construction and (b) surfacing
ambiguous/nonexistent local times explicitly rather than resolving them silently. Neither
requires a *package* boundary; both require a *discipline*. The plan's framing ("sole
Luxon importer") is the part doing extra, unjustified work: forcing every date operation
through one package is a reasonable idea, but it is also the mechanism by which a
not-yet-decided *product policy* (what happens when a booking is requested at a nonexistent
local time) gets a fixed API shape before the policy is chosen. `architect`'s flag on this
point (noted in the plan's §4) is correct and this review agrees with it — but the plan's
§2 structure still lists `packages/time` as a standing package rather than acting on that
flag by deferring the package boundary itself, only the accompanying policy. **A smaller
boundary — a single module whose functions return an explicit `{ok: true, value} |
{ok: false, reason: 'ambiguous'|'nonexistent'}` result shape, without being a versioned,
multi-consumer package yet — would satisfy spike 3's actual requirement without also fixing
a package-level API surface before the product policy exists.**

**`packages/jobs`**: three genuinely separate things the plan does not fully separate, per
Phase 8A's own Adversarial Correction Pass §4 distinction (protocol vs. implementation
model vs. library-vs-hand-roll):
1. **The spike-4-validated protocol** (claim/lease/fence/`SKIP LOCKED`) — FACT, foundation-appropriate.
2. **The decision that the application permanently owns the full implementation** rather
   than adopting a library — Phase 8A's own correction pass explicitly downgrades this to
   "a RECOMMENDATION with a stated re-evaluation point, not a settled permanent
   architecture." The Foundation Plan does not carry this caveat forward — it states
   flatly "claim/lease/fence protocol only, zero domain handlers," which is a
   protocol-scoping statement, but does not remind the reader that Phase 8A itself
   considers "hand-rolled forever" unsettled. **A propagation gap, not a contradiction** —
   worth naming since Review 11 already finds propagation-fidelity to be a systemic issue.
3. **The package boundary itself** — reasonable at foundation time given #1 is
   spike-validated and needs a stable home regardless of #2's eventual resolution.

**`packages/obs`**: not required before the first vertical slice by any spike, ADR-001
clause, or Phase 8A selection. This is the single clearest premature-infrastructure
candidate in the whole package list (see Review 1's table) — a logger with allowlist
discipline is good practice, not foundation-blocking architecture, and does not need a
package boundary before two consumers (`apps/api`, `apps/worker`) exist to justify one.

---

## Review 6 — Attack the three-process/single-image decision

ADR-001 fixes "one deployable (web + worker processes)... No second deployable." This is a
**process-count** commitment (2), not a **build/image-topology** commitment. The
Foundation Plan's `docker/` section ("one Dockerfile, three entrypoints (api/web/worker)")
introduces a *third* process (splitting `apps/api` from `apps/web`) beyond ADR-001's
explicit two (web + worker), and does so via the Next.js/Fastify split Phase 8A's own
Adversarial Correction Pass §5 already flagged as a **RECOMMENDATION, not settled fact**
("Next.js+Fastify remains the strongest candidate evaluated... the original document did
not make this trade-off explicit"). The Foundation Plan inherits Phase 8A's stack selection
(which does include Fastify) correctly as a DECISION per Phase 8A's Owner Approval — so the
three-process shape is not, on inspection, an unauthorized addition; it is Phase 8A's
already-approved stack (Fastify + Next.js) combined with ADR-001's process-count
requirement, which together do imply three processes in practice (Next.js web, Fastify api,
worker) even though ADR-001's own text names only "web + worker."

**Where the plan does over-specify beyond what A/B1 strictly requires**: A/B1 requires "one
deployable image... web + worker processes" — it does not require that `api` and `web` run
as *separate, continuously-running processes* at foundation/dev time. A simpler initial
deployment shape (Next.js serving both UI and API routes, deferring the Fastify split until
a real worker-shared-domain-layer need materializes, per Phase 8A's own §5 "(A) Next.js
alone" option) would satisfy A/B1's two-process requirement (web, worker) more literally
than the plan's three-entrypoint image does, while preserving the extraction seam by
discipline (keeping domain logic out of Next.js route handlers) rather than by a second
framework. **This is not a claim that the plan's choice is wrong** — Phase 8A's §5
reasoning for the Fastify split (shared domain layer between worker and API, avoiding
UI-framework request-lifecycle coupling for booking-correctness code) is a real,
non-manufactured argument, explicitly labeled as such by Phase 8A itself. But the
Foundation Plan states the three-entrypoint Docker shape as if it followed necessarily from
A/B1, without citing that it is actually following from Phase 8A's *stack* decision plus
its own acknowledged-as-RECOMMENDATION architectural split, not from ADR-001's process-count
language alone. **Finding: the plan should cite Phase 8A §5 (Next.js/Fastify split
rationale) explicitly wherever it justifies the three-entrypoint image, rather than
presenting it as following directly from ADR-001**, since ADR-001 alone would also permit
the simpler two-process shape.

---

## Review 7 — Attack the verification plan

`planner`'s five negative proofs, classified:

| Proof | Classification |
|---|---|
| Dependency-boundary lint fires on a deliberate violation | Genuinely necessary before feature work — cheap, catches exactly the SnagTime-style silent-coupling failure mode the project has direct forensic evidence of |
| Fail-closed enrolment fires on an unenrolled table | Genuinely necessary before feature work — this is spike 1's own success criterion 2, directly reproducible, and the single highest-value proof given the project's own SnagTime finding (hand-maintained list failing open) |
| Boot gate refuses three specific bad configs | Necessary before production, useful now — Security Floor item 15's whole point is a boot-time gate; verifying it early is low-cost and high-value, though "which three configs" is not specified in the plan and should be (missing) |
| Migration up/down/up reversibility | Useful but premature at foundation time if no migration with real data-shape risk exists yet — worth doing once the first tenant-scoped table is added, not necessarily at WP0 with an empty schema |
| Job protocol reproduces spike 4 inside real application code | Genuinely necessary before feature work — this is explicitly named in Phase 8A §22 as a required Verification Evidence Record, not merely a nice-to-have |

**Missing verification, by the review's own required checklist**:
- **Tenant-context leakage** (a query issued outside `withTenant` succeeding anyway) — not
  named by `planner`, and per Review 5's `packages/db` finding, this is the single highest-
  value missing proof given that the lint boundary does not catch it.
- **Worker credentials** — `architect` flagged the narrower-worker-role requirement (plan
  §4), but no proof is named verifying the worker role is actually narrower once
  provisioned; spike 1's own success criterion 3 is directly reproducible here and is not
  listed.
- **Transaction rollback** — no proof named that a failed module-level service call
  actually rolls back rather than partially committing; directly relevant to Review 4's
  transaction-boundary correction actually working as intended.
- **Cross-tenant access** — partially covered by "unenrolled table" proof, but that is a
  *schema-enrolment* proof, not an *attacker-controlled-input* proof (the project's own
  Reuse Audit §36 explicitly distinguishes these: "SnagTime's e2e asserts data separation,
  not attacker-controlled cross-tenant access"). Missing.
- **Secret handling** — Security Floor item 7 (no secret committed/logged/exposed); not
  named, cheap to add as a lint/grep-based CI check.
- **Time/DST behavior** — Phase 8A §22 names this explicitly as a required Verification
  Evidence Record ("the Luxon wall-clock construction reproducing spike 3's result"); the
  Foundation Plan's own list omits it even though Phase 8A already requires it. **This is a
  propagation gap, not a new gap this reviewer invented** — Phase 8A (rank 2/3 authority)
  already names this requirement and the Foundation Plan (a later, lower-authority
  synthesis per Review 11) drops it.
- **Evidence preservation** — no Verification Evidence Record is proposed to be produced
  *for the foundation work itself*, despite the Verification Evidence Format document
  existing precisely for this purpose and despite WP0 being HIGH-risk-adjacent (touches
  secrets/config, deployment shape, migrations).
- **Migration safety** beyond up/down/up — no proof named for "a migration applied to a
  copy of realistic data does not silently drop or corrupt a column," though this is lower
  priority than the others given no real data will exist at foundation time.

**Highest-risk proof per unit complexity, if forced to pick the smallest useful set**:
tenant-context leakage, worker-credential narrowness (spike 1 criterion 3, trivial to
reproduce), and the DST proof Phase 8A already requires and the plan already dropped. These
three are cheap (each reproduces an already-designed spike test) and each closes a gap this
review found the plan's own listed proofs do not cover.

---

## Review 8 — Attack the "no implementation until blockers clear" sequencing

Reclassifying into an actual dependency order, rather than accepting WP0–WP9 as given (the
full WP0–WP9 breakdown is not reproduced in the plan itself — see Review 11 — so this
ordering works from the plan's §2 structure and §5 blockers, not from unseen WP detail):

- **(A) Must precede any implementation**: none of the five named blockers actually meets
  this bar in full — see Review 2. The closest is blocker 3 (repo hosting/CI) for anything
  requiring a remote or CI run, and even that does not block *local* work.
- **(B) Must precede schema design**: blocker 1 (tenant-isolation mechanism selection) and
  blocker 4's *product-name-in-schema* consequence (database name, role names) — both
  genuinely need to be settled before the first migration with real tenant-scoped tables is
  written, though blocker 4 already has an accepted default (Review 2).
- **(C) Must precede the first vertical slice** (a real feature, not foundation
  scaffolding): blocker 2 (durable-job schema, if the first slice uses jobs) and the
  nonexistent/ambiguous-local-time product policy (Phase 8A §8/§20 UNKNOWN, not resolved
  anywhere in the Foundation Plan and not listed among its blockers at all — **a gap**: if
  the first vertical slice involves booking creation, this UNKNOWN is load-bearing and
  should have been named as a sixth blocker or explicitly deferred with a stated default,
  neither of which the plan does).
- **(D) Can safely remain open behind an explicit seam**: the recurring-series atomicity
  model (Review 4 — the module-level-service correction already provides this seam); the
  job-library-vs-hand-rolled question (Review 5 — the protocol/implementation-model split
  already provides this seam); the specific auth library (Phase 8A already defers this
  correctly).
- **(E) Should remain deliberately unresolved until real usage provides evidence**: whether
  Kysely proves adequate long-term (Phase 8A §21 already names this as a reopen trigger,
  correctly not pre-resolved); whether `packages/obs` and `packages/kernel` need to exist
  as packages at all (Review 1 — best resolved by waiting for a second consumer).

**Ordering conclusion**: repo-local scaffolding (kernel-as-file, config-as-file,
obs-as-file, the boundary-lint tool, `packages/db`'s pool+seam under neutral naming) can
proceed immediately and autonomously. Blocker 1 must resolve before any migration creating
a tenant-scoped table. Blocker 3 must resolve before any remote push or CI run but not
before local work. Blocker 2 should be tightened from "human gate" to "confirm spike 4's
already-specified columns," likely resolvable without a full owner-decision cycle. The
nonexistent/ambiguous-local-time policy is a gap the plan should have named as blocking (C)
and did not.

---

## Review 9 — Attack reuse and provenance

Two named candidates: Cal.diy's `csp.ts` (58 lines, nonce-based CSP, Security Floor item 14
evidence) and SnagTime's `assertProductionRuntimeSecurity` (~35 lines, Security Floor item
15 evidence). Applying "re-derive unless provenance/licensing/technical suitability is
established":

- **Licensing**: per the Security Floor's own item 16 evidence, dependency/license
  composition is UNKNOWN for both repositories, and Cal.diy specifically carries "an
  unresolved MIT-vs-`UNLICENSED` contradiction inside `apps/api/v2`" (Security Floor item
  16 evidence, sourced from Reuse Security and Licensing Findings). Neither named candidate
  file is stated to live inside `apps/api/v2` — `csp.ts` is at `apps/web/lib/csp.ts` — so
  the specific contradiction may not directly implicate it, but this document did not
  itself verify that boundary; it is relying on the Security Floor's own citation. **The
  plan's stated default ("re-derive, copy nothing") is the correct posture given this
  uncertainty, and should be the operating assumption regardless of whether the specific
  contradiction turns out to implicate these two files or not** — the honest state is
  UNKNOWN, not "probably fine because the contradiction is elsewhere."
- **Technical suitability**: both patterns are short (58 and ~35 lines) and are cited in
  the Security Floor specifically as *evidence that the gap is real and a countermeasure
  exists*, not as code ready to import — the Security Floor's own item 15 language is "the
  demonstrated countermeasure," which describes a *pattern*, not an *artifact* endorsed for
  reuse. **Should they be copied at all?** No — even setting licensing aside, "useful
  pattern" should not become "copy source code" (the task's own instruction, echoing the
  project's charter §6 "do not blindly merge"). Both are short enough that re-deriving a
  Calquartz-specific version (a nonce-based CSP middleware; a boot-time env-validation
  function checking Calquartz's own actual secret list) costs less than establishing clean
  provenance for two ~50-line files would, and re-deriving avoids importing whatever
  licensing risk currently sits unresolved in either upstream repository.
- **Verdict**: the plan's own default (re-derive, copy nothing) is correct and should be
  treated as the operating decision, not merely a fallback "if unresolved" — nothing found
  in this review's reading of the Reuse Audit or Security Floor changes that conclusion.
  This blocker (Review 2, item 4) should be closed by accepting the default explicitly
  rather than left open.

---

## Review 10 — Attack ECC itself

For each questionable foundation element (from Review 1), what it should be instead:

| Element | Current shape in plan | Should instead be |
|---|---|---|
| `packages/kernel` | Standing package | A local file (`lib/kernel.ts`) inside the first module that needs it; promote to a package only after a second module duplicates it |
| `packages/obs` | Standing package | A local `logger.ts`; promote after `apps/worker` needs the identical thing `apps/api` already has |
| `packages/config` | Standing package, "single place any secret is read" | A documented convention (one validated config module per entrypoint) until duplication is measured, not assumed |
| `packages/contracts` | Standing package | A documented rule ("apps/web imports types only, via generated client") backed by codegen, deferred until the API paradigm (still UNKNOWN per Phase 8A §19) is chosen |
| `packages/time` package boundary | Standing package | A local module with the explicit-result-type discipline; package boundary is a future-extraction candidate once a second consumer exists, not a foundation requirement |
| E2E Playwright scaffolding | Foundation-time infrastructure | Deferred to the work package producing the first real page |
| `withTenant`/`withSystem` naming | RLS-shaped naming | Mechanism-neutral naming, same code, cheaper later |

This is the general shape of the finding across Review 1, 5, and this one: **the plan is
building slightly more standing package infrastructure than its own evidence base
(spikes + ADR-001 + Phase 8A) individually justifies**, in a pattern consistent with
default "clean architecture" instinct rather than Calquartz-derived necessity. It is not
the giant universal framework the project's charter (§44, "optimize for maximum useful
capability per unit of complexity") warns against building — most of the excess is one or
two premature package-vs-file boundaries, not new capability — but per that same charter
language, each of the seven items above should be downgraded per the table until a second
real consumer demonstrates the boundary earns its cost.

---

## Review 11 — Attack the plan's evidence preservation

The plan's own §6 admits: `planner`'s full WP0–WP9 breakdown "exists only in a session
transcript... not reproduced in the document"; the synthesis does not preserve full
specialist outputs; the synthesis relies on specialist-reported readings of ADR-001, Phase
8A, the Security Floor, and spike evidence rather than independently re-reading them in
this pass. **This review independently confirms the practical consequence**: Review 8
above could not verify or attack the actual WP0–WP9 risk-tier/routing/human-gate detail,
because that detail does not exist in any durable Second Brain page — only a summary of its
existence does. A session six months from now, asked "was WP3 classified HIGH or LOW, and
why," has no durable page to consult; it has only this plan's assertion that such a
breakdown existed and was incorporated.

**Is this acceptable for institutional memory?** No, per the vault's own root `CLAUDE.md`
("a decision made in conversation is not institutionally captured until it is recorded in
the authoritative Second Brain... an unrecorded decision is, from that session's point of
view, indistinguishable from a decision never made" — project `CLAUDE.md` §46, echoing the
root schema's provenance discipline). The plan's own §6 already states this as a limitation
rather than concealing it, which is the correct honesty move — but stating a gap is not the
same as closing it, and nothing in the plan proposes closing it before WP0 begins.

**Minimum durable provenance structure proposed** (not a call for duplicating everything —
the plan already correctly declines that, citing "one fact, one home"):
1. A companion file (e.g. `Calquartz — Foundation Work Packages (WP0–WP9).md`) transcribing
   `planner`'s actual WP0–WP9 table — risk tier, routing chain, human-gate determination —
   verbatim, once, as its authoritative home. The Foundation Plan then links to it rather
   than summarizing it.
2. Each specialist invocation's identity, prompt/purpose, and load-bearing claims (already
   present in the plan's §4, reasonably well) — kept as-is.
3. Explicit citation of which source documents each specialist actually consulted (the plan
   already states both specialists "name exact absolute file paths they consulted" but does
   not list those paths) — should be listed, not merely asserted to exist.
4. The synthesis reasoning (already present, §3).
5. Corrections made (already present, §3, well done — this is the strongest-provenance part
   of the document).
6. Unresolved questions (already present, §5, though see Review 2 for corrections needed to
   its blocker framing).
7. Final owner decision, when one exists (correctly not yet present — the plan correctly
   does not fabricate one).

Item 1 is the single concrete gap worth closing before WP0 begins; items 2–7 are already
adequately handled by the plan's own structure.

---

## Review 12 — Search for hidden decisions

Scanning the plan for anything effectively decided without being labeled a decision:

- **RLS** — see Review 3. Hidden via `withTenant`/`withSystem` naming and the template-clone
  test-harness choice. **Found.**
- **Recurring atomicity** — not hidden; `architect`'s correction explicitly preserves
  neutrality here (Review 4). **Not found** — a genuine success, worth naming as such.
- **Ambiguous/nonexistent local-time policy** — not decided by the plan, but also not named
  as a blocker or open item anywhere in the Foundation Plan despite Phase 8A already
  flagging it as UNKNOWN and despite `packages/time`'s own existence being motivated by it.
  This is an *omission* rather than a *hidden decision* — but an omission of a load-bearing
  UNKNOWN from a document whose whole purpose is to enumerate blockers is itself a finding.
  **Found (as omission, Review 8 category C).**
- **Auth library** — correctly left open (plan §5, "not blockers, deliberately not
  resolved"). **Not found.**
- **Job implementation model (hand-rolled vs. library)** — see Review 5. The plan states
  "protocol only, zero domain handlers" without carrying forward Phase 8A's own explicit
  caveat that "hand-rolled forever" is unsettled. This risks reading, to a future
  implementer, as more settled than Phase 8A itself considers it. **Found (propagation
  gap, borderline hidden decision).**
- **Job library vs. hand-roll** — same as above, not newly hidden, but the caveat
  attenuation is real.
- **Calendar provider, notification provider** — not mentioned anywhere in the Foundation
  Plan; correctly out of scope for a foundation-only document. **Not found; correctly
  absent.**
- **Seats, round-robin** — not mentioned; correctly out of scope. **Not found.**
- **Public API** — the `packages/contracts` package quietly assumes a specific
  type-sharing shape that presumes a non-public, same-codebase API consumer (`apps/web`).
  If a public developer API (Phase 8A §19, still UNKNOWN) is ever added, `packages/contracts`
  as currently scoped (internal types shared between `apps/web` and `modules/*`) would need
  rethinking, since a public API needs a versioned, stable, externally-documented contract,
  not an internal shared-types package. **Found (mild) — `packages/contracts`'s internal
  framing quietly assumes "no public API," without stating that assumption.**
- **Embeds** — not mentioned; correctly out of scope. **Not found.**
- **Deployment topology beyond what A/B1 requires** — see Review 6. **Found** — the
  three-entrypoint single image goes slightly beyond ADR-001's own literal two-process
  text, justified by Phase 8A's stack pick but not cited as such.
- **Repository/CI choice** — correctly named as an open blocker (Review 2), not hidden.
  **Not found as hidden — correctly flagged.**
- **Product name** — correctly named as an open blocker with a stated default (Review 2).
  **Not found as hidden — correctly flagged**, though see Review 2 for the §7
  contradiction.

**Summary of Review 12's findings**: two clear hidden decisions (RLS-shaped tenant-context
naming/test-harness choice; three-entrypoint deployment topology stated as following from
ADR-001 alone rather than from Phase 8A's stack pick), one significant omission (the
ambiguous/nonexistent-local-time policy not named as blocking anything), one propagation
attenuation (job-implementation-model caveat dropped), and one mild unstated assumption
(`packages/contracts`'s internal-only framing).

---

## Required Final Judgment

**APPROVE WITH REQUIRED CORRECTIONS.**

The plan is not badly conceived — `architect`'s two corrections (Review 4, the
`packages/contracts` contradiction-fix) are both genuine, well-reasoned catches, and the
overall shape stays within A/B1 with no reopen trigger implicated. But it should not be
authorized to begin WP0 as currently written, because (a) its own §7 sequencing statement
contradicts its own §5 blocker analysis (Review 2), (b) it embeds two mechanism-specific
naming/design choices that quietly narrow an explicit ADR-001 non-decision (Review 3), (c)
it omits a load-bearing UNKNOWN (the ambiguous/nonexistent-local-time policy) from its own
blocker list (Reviews 8, 12), and (d) it carries more standing package infrastructure than
its cited evidence individually justifies (Reviews 1, 10). None of these is severe enough
to warrant REJECT AND REPLAN — each is a bounded, nameable correction, not a structural
flaw requiring the plan be redone from scratch.

**Minimum corrections required before implementation:**
1. Correct §7's sequencing statement: only blockers 1 (schema-touching work) and 3
   (remote/CI-touching work) genuinely gate any WP0 work; blocker 2 should be tightened to
   "confirm spike 4's already-specified schema" rather than a full human-decision gate;
   blockers 4 and 5 should be marked resolved-by-accepted-default rather than open.
2. Rename `withTenant`/`withSystem` to mechanism-neutral names before any code is written
   using them; document explicitly that the test harness's template-clone-per-file choice
   is independent of which isolation mechanism is eventually selected (or correct the
   harness if it is not actually independent).
3. Add the ambiguous/nonexistent-local-time product policy to the blocker/open-items list
   as a category-C item (must precede the first vertical slice if that slice touches
   booking creation), with an explicit default stated if the owner does not resolve it
   immediately (e.g., reject nonexistent local start times, require explicit UTC-offset for
   ambiguous ones — Phase 8A §8 already names this exact plausible default).
4. Downgrade `packages/kernel`, `packages/obs`, `packages/config`, `packages/time`, and
   `packages/contracts` from standing packages to local files/modules per Review 1's
   "minimum viable alternative" column, promoting each to a package only once a second real
   consumer demonstrates the boundary earns its cost.
5. Cite Phase 8A §5 explicitly wherever the plan justifies the three-entrypoint Docker
   image, rather than presenting it as following directly from ADR-001 alone.
6. Carry forward Phase 8A's own caveat that "application permanently owns the full job
   framework" is an unsettled RECOMMENDATION with a stated re-evaluation point, not a
   settled design, wherever `packages/jobs` is described.
7. Add the missing verification items from Review 7 (tenant-context leakage, worker-role
   narrowness, transaction rollback, cross-tenant attacker-controlled-input testing, secret
   handling, and — critically, since Phase 8A already requires it — the DST/time
   verification the plan currently drops) to the negative-proof list.
8. Produce the companion WP0–WP9 transcription file named in Review 11 before treating the
   work-package breakdown as authoritative, or explicitly accept the provenance gap as a
   standing risk with the owner's sign-off.

### 1. Minimum safe foundation

`apps/api`, `apps/web`, `apps/worker` (per ADR-001 + Phase 8A); `migrations/` (SQL-first,
per Phase 8A); the integration-test harness (real-Postgres, per Phase 8A §12); the
boundary-lint tool (dependency-cruiser or equivalent — cheap, evidence-justified by the
project's own SnagTime forensic finding); `packages/db`'s pool + mechanism-neutral
tenant-context seam (spike-1-justified, renamed per correction 2); `packages/jobs`'s
protocol-only scope (spike-4-justified, caveated per correction 6). Everything else in
Review 1's table should start as a file or convention, not a package.

### 2. Genuine blockers

Blocker 1 (tenant-isolation mechanism), scoped to schema-touching work only. Blocker 3
(repo hosting/CI), scoped to remote/CI-touching work only. The ambiguous/nonexistent-local-
time policy, scoped to booking-creation-touching work (newly surfaced by this review, not
in the plan's own list).

### 3. Non-blockers incorrectly classified as blockers

Blocker 4 (licensing) — already resolved by the plan's own stated default; should be
closed, not left open (Review 9 confirms the default is correct). Blocker 5 (product name)
— already resolved by the plan's own stated default. Blocker 2 (durable-job schema) —
overclassified; should be a confirm-not-decide step given spike 4's specificity.

### 4. Hidden decisions found

RLS pre-selection via `withTenant`/`withSystem` naming and the test-harness shape (Review
3); three-entrypoint deployment topology presented as following from ADR-001 alone rather
than from Phase 8A's stack pick (Reviews 6, 12); `packages/contracts`'s unstated
no-public-API assumption (Review 12).

### 5. Premature abstractions

`packages/kernel`, `packages/obs` (no cited requirement at all); `packages/config`,
`packages/time`, `packages/contracts` (real underlying requirement, premature package-level
commitment); foundation-time Playwright e2e scaffolding (Review 1).

### 6. Missing gates

The ambiguous/nonexistent-local-time policy is not gated as a blocker anywhere despite
being load-bearing for the first booking-touching vertical slice (Reviews 8, 12). No
Verification Evidence Record is proposed for the foundation work itself despite WP0 being
adjacent to several HIGH-risk domains (secrets/config, deployment, migrations) per the
Authority/Risk/Routing Model's own domain list.

### 7. Required provenance improvements

A companion file transcribing `planner`'s full WP0–WP9 breakdown verbatim (Review 11);
explicit listing of the exact file paths each specialist consulted, not merely an assertion
that they did so.

### 8. Recommended sequencing

Local-only scaffolding (kernel-as-file, config-as-file, obs-as-file, boundary-lint tool,
`packages/db` seam under neutral naming, `packages/jobs` protocol) → confirm durable-job
schema against spike 4's already-specified shape → resolve or explicitly default the
ambiguous/nonexistent-local-time policy → resolve tenant-isolation mechanism (owner gate,
required before first tenant-scoped migration) → resolve repo hosting/CI (owner gate,
required before first remote push) → first vertical slice.

### 9. What can proceed autonomously

All local-only scaffolding named in §8 above; adopting the licensing default (re-derive,
copy nothing) as the closed answer to blocker 4; adopting the product-name default
(`calquartz`) as the closed answer to blocker 5; writing the boundary-lint configuration;
writing the spike-4-derived job-table migration for owner confirmation (not decision) per
correction 3 above.

### 10. What requires owner approval

The tenant-isolation mechanism selection (RLS vs. an alternative) — genuinely ADR-001's own
explicit non-decision. Repository hosting and CI provider. Confirmation of the durable-job
table's exact schema (narrow confirm-not-decide, but still per the routing table's own
"touches a shape a prior document assumed" rule). The ambiguous/nonexistent-local-time
product policy, if the owner wants to set it rather than accept Phase 8A's named plausible
default.

### 11. What must remain UNKNOWN

Everything Phase 8A §19–20 and ADR-001's "Explicit non-decisions" already name: the
recurring-series atomicity model (correctly preserved neutral, Review 4); the specific auth
library; whether a job library eventually replaces the hand-rolled protocol; seats/
round-robin; calendar/notification providers; public API scope; embeds; the canonical
product name beyond the accepted working default.

### 12. Whether WP0 should be authorized at all

**Not as currently written.** WP0 should be authorized only after corrections 1–3 above
(sequencing fix, mechanism-neutral naming, the local-time-policy gap) are applied to the
plan, since those three specifically affect what gets built and named during WP0 itself —
building `withTenant`/`withSystem` under RLS-shaped naming, or omitting the local-time
policy from the blocker list, cannot be cheaply undone once modules start depending on the
names or once a booking-creation slice is built against an undecided policy. Corrections
4–8 (package-vs-file downgrades, citation fixes, verification additions, provenance
transcription) are real but lower-urgency — they improve the plan's quality and are
distinctly not sequence-position-1 blockers, and could reasonably be applied incrementally
during WP0 rather than gating its start. **Recommended path: apply corrections 1–3 to the
plan document, then authorize WP0's local-only-scaffolding portion (§9 above) immediately,
while corrections 4–8 are tracked as an explicit WP0 follow-up rather than a precondition.**

---

## Related

- [[Calquartz — Application Foundation Implementation Plan]] — the plan this document reviews
- [[../decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]]
- [[Phase 8A — Implementation Stack Selection]]
- [[Authority, Risk, and Routing Model]] · [[Verification Evidence Format]]
- [[../architecture/Security Floor|Security Floor]]
- [[../reuse/Reuse Audit Summary|Reuse Audit Summary]]
- [[../Calquartz — Pre-Approval Technical Spikes|Pre-Approval Technical Spikes]]
- [[../questions/Architecture Open Questions|Architecture Open Questions]]
