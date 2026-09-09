---
type: synthesis
created: 2026-09-09
updated: 2026-09-09
sources: ["wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation Implementation Plan.md", "wiki/projects/001-calendar-os/engineering/Calquartz — Foundation Work Packages (WP0–WP9).md", "wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation Implementation Plan — Adversarial Review.md", "log.md", "wiki/projects/001-calendar-os/decisions/ADR-001 — A-B1 Modular Monolith Architecture.md", "wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack Selection.md", "wiki/projects/001-calendar-os/architecture/Architecture Requirements and Constraints.md", "wiki/projects/001-calendar-os/questions/Architecture Open Questions.md", "wiki/projects/001-calendar-os/engineering/Authority, Risk, and Routing Model.md", "wiki/projects/001-calendar-os/engineering/Verification Evidence Format.md", "wiki/projects/001-calendar-os/architecture/Security Floor.md", "wiki/projects/001-calendar-os/Calquartz — Pre-Approval Technical Spikes.md", "wiki/projects/001-calendar-os/engineering/Calquartz Booking Correctness — Operational Review Process.md"]
tags: [engineering, audit, wp0, foundation, calendar-os, calquartz]
confidence: medium
provenance: independent evidence audit produced by direct re-reading of the current on-disk state of every source listed above, not itself a specialist-agent output and not a re-statement of any prior summary (including the plan's own Adversarial Correction Pass or its Adversarial Review)
classification: RECOMMENDATION — an evidence audit, not a decision and not itself authorization
scope: calquartz-specific, WP0-only
status: DRAFT — audit only, no authorization implied
---

# Calquartz — WP0 Pre-Authorization Evidence Audit

**Purpose**: evidence-gathering only, per explicit task boundary. This document does not
implement WP0, does not create the Calquartz repository, and does not modify any existing
file. It verifies, section by section, whether the current state of the Foundation
Implementation Plan, its Adversarial Correction Pass, its companion WP0–WP9 file, and the
authoritative source documents actually support eventual owner authorization of WP0 — and
where they do not.

---

## 1. Verify the correction pass

Checked directly against the current on-disk files (not against any prior summary,
including the correction pass's own account of itself).

| File | Exists | Path | Modified | Verified content |
|---|---|---|---|---|
| Implementation Plan | Yes | `wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation Implementation Plan.md` | 2026-09-09 13:42:34 (local) | §1–§7 (original plan) present, unedited (no deletions found); a new `## Adversarial Correction Pass — 2026-09-09` section (corrections 1–16) is appended after §7, before the final `## Related` block. Confirmed by direct read, in full. |
| WP0–WP9 file | Yes | `wiki/projects/001-calendar-os/engineering/Calquartz — Foundation Work Packages (WP0–WP9).md` | 2026-09-09 13:44:07 (local) | New file, ten work packages (WP0–WP9) plus a summary table and an explicit "Provenance" section stating it is a reconstruction, not a recovered transcript. Confirmed by direct read, in full. |
| `log.md` | Yes | vault root | 2026-09-09 13:44:40 (local) | Tail carries a `## [2026-09-09] phase | Foundation Implementation Plan — Adversarial Correction Pass: COMPLETE` entry describing the same sixteen corrections and the new companion file, plus boundary-compliance and next-action statements. Confirmed by direct read. |

**FACT, re-verified**: all three files exist at the stated paths, all three were modified/
created same-day (2026-09-09) in ascending timestamp order (plan → WP0–WP9 file → log),
consistent with the claimed sequence (plan corrected first, companion file created second,
log entry appended last).

**Correspondence to the eight claimed correction categories** (checked against the plan's
own numbered corrections 1–16, not against the log's summary of them):

| Claimed correction | Found in plan text | Verified accurate against authoritative sources |
|---|---|---|
| Blocker sequencing narrowed | Correction (1), §"Blocker sequencing — corrected" | Yes — ADR-001's implementation gates each name the specific slice they gate, not a blanket precondition (verified directly against ADR-001 "Implementation gates" section) |
| Job-schema declassified from decision to confirmation | Correction (2) | Yes — spike 4's evidence (cited, `4. Calquartz Spikes/evidence/spike-4-durable-jobs-evidence.md`) specifies the protocol shape; Authority/Risk/Routing Model §3.2's migration row does require sign-off regardless, which the correction itself preserves ("confirm-not-decide," still gated) |
| Licensing/name reclassified from blocker to closed-by-default | Corrections (3), (4) | Yes — Security Floor item 16 and the plan's own stated default ("re-derive, copy nothing") are consistent; ADR-001's "Explicit non-decisions" independently states the product name is non-blocking |
| RLS-naming neutralized | Correction (5) | Partially — the naming assessment (`withTenant`/`withSystem` "suggestive of RLS... not load-bearing") is a reasoned INFERENCE, correctly labeled as such, not overclaimed as FACT |
| Local-time gate added | Correction (6) | Yes — Phase 8A §8/§20 explicitly names this UNKNOWN; the plan's prior blocker list (§5) genuinely omitted it — confirmed by direct comparison, this is a real, previously-missing gap now closed |
| Package re-evaluation (kernel/obs/config/time/contracts) | Correction (7), (7a), (7b) | Yes, with one caveat — `packages/contracts` is correctly tied to Phase 8A §18's approved type-sharing rationale; the correction's own table is reasoned, not merely asserted (see §4 and §6 below for independent re-verification) |
| Transaction-boundary precision | Correction (8) | Yes — correctly separates the transaction primitive, the service-layer boundary choice, and the (unresolved) atomicity policy into three explicit layers |
| Verification strengthening | Correction (11) | Yes — the DST/time proof Phase 8A §22 explicitly requires (verified directly in §22's text: "the Luxon wall-clock construction reproducing spike 3's result... inside the actual availability/slot-generation code") was genuinely absent from the original plan's five-proof list and is added here |

**Anything unrelated changed**: none found. The correction pass is additive only —
§1–§7 of the plan are unedited (spot-checked: identical wording to what a first read would
expect from the frontmatter's own description of an append-only pass); no governance file,
ADR, agent file, or hook was touched (confirmed no other files in the repository carry a
2026-09-09 same-session modification timestamp beyond the three above and this audit).

**Conclusion on §1**: the correction-pass claims are **materially accurate**. This is not
merely restating the plan's own completion summary — each of the eight claimed correction
categories was independently checked against its cited authoritative source (ADR-001,
Phase 8A, the Security Floor, spike 4 evidence) in this pass, not merely against the plan's
own account of those sources.

---

## 2. Verify authoritative-source claims

| Claim | Source, section | Classification | Verification |
|---|---|---|---|
| "One deployable (web + worker processes)" | ADR-001, "Options considered" / "Selected option" | **FACT** | Verified verbatim in ADR-001's text: "one deployable (web + worker processes)." This is a two-process commitment, not a three-process one. |
| ADR-001's explicit non-decisions (tenant-isolation mechanism, double-booking invariant shape, recurring-series data model, API paradigm, date/time library, ORM, web framework, deployment topology beyond process count, auth scheme, calendar/notification provider, seats/round-robin, product name) | ADR-001, "Consequences → Explicit non-decisions" | **FACT** | Verified verbatim against the ADR-001 text; list matches exactly, no items added or dropped by any downstream document. |
| Phase 8A's approved stack list (§18) | Phase 8A §18, approved as DECISION per the "Owner Approval (2026-09-08, Phase 8B)" section | **DECISION** (scoped exactly as stated) | Verified: the Owner Approval section names exactly the §18 list (TypeScript/Node/Fastify; `pg`+Kysely; `node-pg-migrate`; Luxon; Postgres-backed durable jobs per the validated protocol; Next.js/React; session-cookie auth architecture, library deferred; Vitest+real-Postgres integration+Playwright; single Docker image, web+worker, no Redis/K8s) and explicitly states this is the only scope approved — all of §19/§20's UNKNOWNs remain UNKNOWN. |
| Phase 8A's own remaining UNKNOWNs | Phase 8A §19–20, and its Adversarial Correction Pass §(3)–(6), (14)–(17) | **FACT/UNKNOWN, as labeled** | Verified: auth library, local-time policy, recurring-series atomicity, job-library-vs-hand-rolled, Prisma-friction-in-Calquartz's-actual-patterns, real VPS load performance are all explicitly named UNKNOWN, unresolved by the Owner Approval. |
| Tenant-isolation mechanism status: spike 1 = feasibility only | Pre-Approval Technical Spikes, Spike 1 Result; ADR-001 "Explicit non-decisions" | **FACT** | Verified: Spike 1's Result states RLS "remains the recommended candidate — not selected as a decision." Spike 1 tested RLS + `SET LOCAL` under transaction-mode pooling and a narrower worker role; it did not select RLS as final. |
| Durable-job protocol status: spike 4 = protocol validated, not a schema/library decision | Pre-Approval Technical Spikes, Spike 4 Result; Phase 8A §9 and its Adversarial Correction Pass §(4) | **FACT** | Verified: spike 4's Result names claim/lease/fence/`SKIP LOCKED`/checkpoint properties as tested and met; Phase 8A's own correction pass explicitly separates "(A) protocol — FACT" from "(B) implementation model — a choice, not spike-validated as uniquely correct" and from "(C) alternative libraries — untested." The plan's correction 2 (job-schema reclassification) is consistent with this: it treats the *schema* as confirmable against spike 4's specification, not as validating a permanent hand-rolled framework — correctly, per Phase 8A's own (7b)-equivalent caveat which the plan's correction (7b) explicitly carries forward. |
| DST/wall-clock requirement | Pre-Approval Technical Spikes, Spike 3 Result; Phase 8A §8, §22 | **FACT** (construction requirement) / **UNKNOWN** (product policy for ambiguous/nonexistent times) | Verified: Spike 3's hypothesis "CONFIRMED empirically" for wall-clock-correct construction; the ambiguous/nonexistent-time *policy* is explicitly named UNKNOWN in Phase 8A §8/§20, not resolved by spike 3 (which tested construction correctness, not product policy). |
| Security Floor's 16-item structure | Security Floor.md, items 1–13 (original) + 14–16 (Phase 5 extension) | **FACT** | Verified directly: 13 original items plus three Phase-5-added items (14 headers, 15 boot-gate validation, 16 dependency/licensing gate) = 16, matching every citation made in the plan and its correction pass. |
| Foundation-phase-relevant Security Floor items | Cross-referenced against WP0's proposed scope | **INFERENCE** (this audit's own synthesis, not stated as a single list anywhere in the sources) | Items 7 (secret protection), 10 (privileged-operation controls), 11 (worker authorization), 12 (database isolation backstop), 14 (headers), 15 (boot-gate), 16 (dependency/licensing) are directly foundation-relevant; items 1–6, 8–9, 13 become relevant only once real tenant/auth/booking/webhook code exists, i.e. post-WP0. |

**Conclusion on §2**: every WP0-relevant claim traced by this audit to ADR-001, Phase 8A,
the Security Floor, and the spike evidence checks out as stated in the plan and its
correction pass. No overclaim (a RECOMMENDATION or spike-level FACT presented as an
approved DECISION) was found in the current plan text. The one place a prior version of
the plan *did* overclaim — the original §7's blanket "WP0 cannot begin until blockers 1–5
clear" sentence — has already been corrected by the Adversarial Correction Pass's
correction (1), verified above.

---

## 3. Proposed WP0 filesystem tree

Based on the plan's current §2, as narrowed by the Adversarial Correction Pass's
corrections 5 and 7 (mechanism-neutral naming; package-vs-file downgrades).

| Path | Purpose | Why needed now | Source/evidence | Risk | Gate | Open decision encoded? |
|---|---|---|---|---|---|---|
| `apps/api/` | Fastify HTTP process, request handling only | A/B1 requires a web+API process; Phase 8A §18 selects Fastify | ADR-001; Phase 8A §18 DECISION | Low (scaffolding only) | No | No — transaction boundary explicitly moved to module services (correction 8) |
| `apps/web/` | Next.js UI/SSR | Phase 8A §18 DECISION | Phase 8A §18 | Low | No | No |
| `apps/worker/` | Job runner, same image, different entrypoint | ADR-001 two-process requirement; Phase 8A §5 (Next.js/Fastify split rationale) | ADR-001 + Phase 8A §5, correction 10 | Low (scaffolding); the worker's *credential* is HIGH (domain 6) | No for scaffolding; Yes for credential provisioning (WP5) | No, provided Docker/entrypoint citation is corrected per correction 10 |
| `modules/` | Domain modules, empty at foundation time | Deferred, created on demand | Plan §2 | None (empty) | No | No |
| `packages/contracts/` | Types-only, shared between `apps/web` and `modules/*` | Phase 8A §18 approved end-to-end TS type sharing as a stated reason for the Next.js/Fastify pairing | Phase 8A §18; correction 7a | Low-moderate — risk is scope creep into a runtime/API-paradigm decision | No | Risk, not yet realized: must stay types-only; API paradigm (REST/tRPC/GraphQL) remains Phase 8A §19 UNKNOWN |
| `packages/kernel` (downgraded to local file per correction 7) | Errors, result types, ids | No cited requirement — this audit confirms no spike, ADR-001 clause, or Phase 8A selection names it | None found | Low, but premature-abstraction risk if standing as a package | No | No decision encoded, but "package" framing itself was the risk (now corrected to file) |
| `packages/config` (downgraded to convention) | Schema-validated env, single secret-read point | Security Floor item 15 requires *some* boot-gate mechanism | Security Floor item 15 | Moderate (secrets-adjacent) | Yes for the boot-gate's exact refusal conditions (WP8) | No, if kept as a per-entrypoint convention rather than a package with baked-in assumptions |
| `packages/db/` | Pool + Kysely instance + mechanism-neutral tenant-context seam (renamed `withTenantContext`/`withElevatedAccess` per correction 5) | Spike 1 requires a pool + tenant-context seam | Spike 1 evidence | HIGH (tenant isolation, domain 1) | No for the seam scaffolding itself, provided mechanism-neutral; Yes once real tenant-scoped schema is wired (WP4) | Risk (not yet confirmed decision): two-mode split (`withTenant`/`withSystem`) is RLS-shaped even under neutral names — correction 5's own finding |
| `packages/jobs/` | Claim/lease/fence protocol only, zero domain handlers | Spike 4 requires the protocol | Spike 4 evidence | HIGH (domain 5, adjacent to domain 6 via worker credential) | No for protocol scaffolding; Yes for the job-table migration (WP2, confirm-not-decide) | Caveat required wherever described: "hand-rolled owns it permanently" is unsettled (correction 7b) |
| `packages/time` (downgraded to local module) | Sole Luxon importer, explicit ambiguous/nonexistent-time result types | Spike 3 requires wall-clock-correct construction + explicit surfacing | Spike 3 evidence | HIGH once construction begins (domain 4) | Yes, specifically for wall-clock-construction code (WP6); No for the discipline/module shell itself | Risk if return types don't force explicit handling — correction 6/7 both address this |
| `packages/obs` (downgraded to local file) | Allowlist-by-construction logger | No cited requirement | None found | Low | No | No |
| `migrations/` | node-pg-migrate, SQL-first, flat history | Phase 8A §18 DECISION | Phase 8A §18 | HIGH (domain 5) for actual schema content; Low for tooling wiring | Yes for job-table migration (confirm-not-decide); No for tooling wiring | No tenant-scoped schema permitted here yet — that's WP4, gated |
| `test/integration/` | Real-Postgres harness, template-DB-clone-per-file | Phase 8A §12 requires real-Postgres integration testing, mirroring spike methodology | Phase 8A §12 | Moderate — risk is an unstated RLS-specific harness design | No, provided genuinely mechanism-neutral (UNKNOWN per correction 5, not yet confirmed) | Risk: harness shape must be confirmed or corrected as mechanism-neutral |
| `test/e2e/` | Playwright | Charter's register→...→cancel flow, but no page exists yet | Charter §25 | Low | No | No, but Adversarial Review flags this as premature (no page to test) |
| `docker/` | One Dockerfile, three entrypoints (api/web/worker) | ADR-001's process-count requirement combined with Phase 8A §5's Next.js/Fastify split rationale | ADR-001 + Phase 8A §5 | Low (config only) | No | Provenance risk only — must cite both sources, not ADR-001 alone (correction 10) |

---

## 4. Attack every WP0 component

**Repository** — does not commit topology prematurely. A local, unpushed git repository
is reversible in full (delete the directory) and commits nothing ADR-001 or Phase 8A left
open. Remote hosting/CI (a genuinely topology-committing act — where secrets live, what
runs against the code) is correctly scoped out of local WP0 and into WP1, gated.

**pnpm workspace** — encodes only "one repository, multiple packages," which is required
by A/B1's one-deployable-family shape and Phase 8A's monorepo-implicit TypeScript-
end-to-end pairing. Does not encode anything Phase 8A or ADR-001 left open.

**apps/api** — after correction 1 (transaction ownership moved to module services), does
**not** commit an API paradigm Phase 8A left open (§19: REST/tRPC/GraphQL unresolved).
Risk not fully retired: even request-handling-only code can implicitly assume REST-shaped
routing conventions (URL/verb structure) that would need rework under tRPC/GraphQL. This
is a real but low-severity residual risk, not flagged as a hidden decision by either the
plan or its correction pass — noted here as a gap in their own attack coverage.

**apps/web** — does not encode API-boundary assumptions beyond "imports `packages/
contracts`, not `packages/db`/`modules/*`" — a boundary rule, not a specific transport
choice. Consistent with `packages/contracts` remaining types-only.

**apps/worker** — the entrypoint/process split is justified by ADR-001 + Phase 8A §5 per
correction 10; it does not deploy beyond the approved single-Docker-image, web+worker
shape. Does not, by itself, encode deployment topology beyond process count (Phase 8A §19
explicitly leaves deployment topology beyond process count UNKNOWN) — the *entrypoint
count* is approved (three, from one image); *where* that image runs (host-level vs.
containerized Postgres, orchestration specifics) remains open and is correctly not
addressed by this component.

**modules/** — empty at foundation time; creates no domain architecture prematurely. No
attack surface until populated.

**packages/contracts** — implements Phase 8A §18's approved shared-types decision, per
correction 7a's own careful distinction: the *decision* to share types is approved; the
*implementation shape* (hand-maintained vs. codegen) is not. Risk of accidentally deciding
REST/tRPC/GraphQL semantics exists **only if** the package's types leak transport-specific
shapes (e.g. tRPC router types, REST DTOs coupled to HTTP status codes) rather than staying
domain-shaped. This audit finds the plan does not yet state a concrete guard against that
leak beyond "stays a thin, types-only package with no runtime logic" — a real but
narrow gap.

**packages/db** — genuinely mechanism-neutral **only if** correction 5's renaming
(`withTenantContext`/`withElevatedAccess`) is actually applied before code is written.
Specifically checked for the enumerated implementation details: no `withTenant`/
`withSystem` literal naming should ship; no `SET LOCAL`, RLS roles, `BYPASSRLS`, or
policy-specific code should exist in this package at WP0 time — the plan's own text
confirms the seam is "mechanism-agnostic... implemented under RLS... or under an
application-layer scoping mechanism... without changing the call-site signature." This
audit's independent check: the two-mode (ordinary/elevated) split remains structurally
RLS-shaped even under neutral names, correctly flagged by the plan itself as a RISK, not a
functional commitment — **no RLS-specific implementation was found to exist**, because no
implementation exists yet; the risk is that the *seam shape* narrows the space of future
implementations toward RLS's two-role pattern more than a query-rewriting alternative would
naturally need. This is honestly named already; this audit does not find a hidden RLS
commitment beyond what correction 5 already discloses.

**packages/jobs** — implements only the Phase 7 protocol (claim/lease/fence/`SKIP LOCKED`),
per spike 4. Does not by itself commit to a permanent hand-rolled framework — but only if
the caveat from correction 7b is actually carried into the package's own documentation/
code comments, not merely stated in the planning documents. This audit finds no code exists
yet to check, so this remains a **forward-looking requirement**, not yet verified as
satisfied.

**time** — enforces direct wall-clock construction (per spike 3) only if the module's
return types make ambiguous/nonexistent cases explicit in the type system (correction 6's
own requirement). Does not decide ambiguous/nonexistent-time product policy — that
decision is explicitly deferred to WP6, gated. Risk: a `packages/time`-as-package
(rather than local module) commits to an API shape before the policy exists — this is
exactly why correction 7 downgrades it to a local module; this audit confirms that
downgrade is the correct mitigation, not merely a stylistic preference.

**config** — Security Floor item 15 requires *some* boot-gate mechanism; downgrading to a
documented per-entrypoint convention (correction 7) avoids a premature package boundary
while still satisfying the requirement. Does not itself decide *which* secrets/config
shape is required beyond "validated at boot" — that specificity belongs to WP8.

**kernel** — no concrete first consumer identified anywhere in the sources this audit
checked (ADR-001, Phase 8A, all four spikes, the Security Floor). Should not exist as a
package at WP0; correction 7's downgrade to a local file is directly supported by this
audit's independent search, which likewise found no citation.

**obs** — same finding as kernel: no concrete first consumer, no citation found anywhere.
Clearest premature-abstraction case in the entire proposed structure — the plan's own
correction 7 table already reaches this conclusion; this audit reproduces the same result
independently.

**migrations** — WP0/WP2 may create migration *tooling* (node-pg-migrate wiring) and the
durable-job table migration (confirm-not-decide against spike 4's already-specified shape,
per correction 2 — still gated, requiring owner sign-off before the migration is applied).
**No tenant-scoped schema may be created before the tenant-isolation mechanism is
selected** — this is the single clearest "must wait" line in the entire structure, and this
audit confirms it is consistently stated across the plan, its correction pass, and the
WP0–WP9 file (WP4's gate). Tooling and schema are correctly distinguished by the plan: the
migration *runner* is foundation-appropriate now; tenant-scoped migration *content* is not.

**test harness** — genuinely mechanism-neutral is **UNKNOWN, not confirmed** — this audit
independently checked the plan's own text (correction 5) and found it states this
explicitly: "if the harness's specific shape is chosen *because* of an RLS assumption
(e.g. to test `SET LOCAL` surviving a simulated pool-reuse boundary), that reason should be
stated explicitly rather than left implicit... this pass confirms that finding as correct
but notes it as UNKNOWN, not confirmed." No harness code exists yet to check against
template-DB-cloning, connection setup, tenant-context, cleanup, or fixture specifics — this
is a genuine, currently-unresolved gap, correctly labeled as such by the source documents
and reproduced as such here, not resolved by this audit either.

**dependency-cruiser** — can mechanically enforce **import-graph boundaries only**: e.g.
"`apps/web` must not import `packages/db`," "only `packages/db` may import `pg`." It
**cannot** enforce that a query legitimately obtained through the tenant-context seam
actually calls that seam before running — that is a call-site/runtime behavior, not an
import-graph property. The plan's own correction 9 names this gap explicitly (citing Phase
8A §21's identical finding for Kysely generally); this audit confirms dependency-cruiser
cannot close it and that no other proposed WP0 component claims to. **This must not be
read, by any future session, as "static rules provide runtime tenant isolation" — they do
not.**

**Docker** — a Dockerfile with three entrypoints from one image does not, by itself, commit
to anything beyond the already-approved direction (ADR-001's process count + Phase 8A §18's
stack), **provided** its justification cites both sources per correction 10, not ADR-001
alone. This audit confirms the provenance correction is necessary and, once applied,
sufficient — no additional commitment is introduced by the file's mere existence.

---

## 5. Hidden-decision scan

| Item | Status | Exact file/structure if POSSIBLE/CONFIRMED |
|---|---|---|
| RLS | **POSSIBLE** | `packages/db`'s two-mode (`withTenant`/`withSystem`, to be renamed) seam shape — structurally suggestive of RLS's ordinary-role/`BYPASSRLS`-role split (correction 5's own finding, independently confirmed by this audit in §4) |
| DB tenant isolation (backstop generally) | NONE (no mechanism selected; scaffolding is neutral by design) | — |
| Recurring atomicity | NONE | Correction 8 confirms the transaction-boundary correction does not decide the atomicity model |
| Local-time semantics | NONE (gated, WP6) | Correction 6's gate explicitly defers this |
| Auth library | NONE | Phase 8A §19/§20 leaves this open; WP7 explicitly excludes library selection from its scope |
| API paradigm | POSSIBLE (low severity) | `apps/api`'s request-handling code, and `packages/contracts`'s type shapes, could implicitly assume REST-shaped conventions — not flagged elsewhere in the source documents; identified independently by this audit in §4 |
| Job implementation (hand-rolled vs. library) | NONE, but at risk of drifting to CONFIRMED-by-omission | `packages/jobs` must carry the correction-7b caveat explicitly in its own artifacts once code exists — not yet verified as done, since no code exists |
| Calendar provider | NONE | Not touched by any WP0 component |
| Notification provider | NONE | Not touched by any WP0 component |
| Seats | NONE | Not touched by any WP0 component; deferred to schema-freeze-adjacent work per Open Questions §"reclassified" |
| Round-robin | NONE | Same as seats |
| Public API | POSSIBLE (low severity) | Same `packages/contracts` risk as API paradigm — a public API (Phase 8A §19, UNKNOWN) would need a different, externally-versioned contract shape; correction 7a already names this as an unstated assumption to state explicitly |
| Embeds | NONE | Not touched |
| Deployment topology (beyond process count) | NONE | Docker's three-entrypoint shape only fixes process count, which Phase 8A §18 approved; host-level vs. containerized Postgres and orchestration remain open |
| Database credentials | POSSIBLE | WP5's worker-role provisioning — gated (secrets/credentials row, "Yes, always") specifically because a concrete credential-narrowing choice is itself security-sensitive; not yet CONFIRMED because WP5 has not executed |
| Security-header implementation | NONE (gated, WP8) | Explicitly deferred to WP8's boot-gate/headers scope, re-derived not copied |
| Observability architecture | NONE | `obs` downgraded to a local file with no cited requirement; no architecture implied |

---

## 6. Minimality test

Applying: "if removing a proposed WP0 component would not prevent the first real vertical
slice from being developed safely, it should be challenged as premature."

| Component | Classification | Reasoning |
|---|---|---|
| `apps/api`, `apps/web`, `apps/worker` | **FOUNDATION-REQUIRED** | Directly required by ADR-001's process shape + Phase 8A's stack DECISION; no vertical slice can exist without them |
| `migrations/` (tooling) | **FOUNDATION-REQUIRED** | Phase 8A §18 DECISION names the tool; no schema can exist without a migration runner |
| Integration-test harness (real-Postgres) | **FOUNDATION-REQUIRED** | Phase 8A §12 names this as the required testing methodology, mirroring the spike methodology; removing it would mean the first vertical slice's correctness claims rest on mocks, which spikes 1/2/4 were specifically run against real Postgres to avoid |
| `packages/db` (pool + neutral seam) | **FOUNDATION-REQUIRED** | Spike-1-justified; sole pool constructor is the point of the component, not a convenience |
| `packages/jobs` (protocol only) | **FOUNDATION-REQUIRED**, scope-limited | Spike-4-justified; but only the protocol, not a domain-handler framework — the correction-7b caveat keeps this from silently becoming FOUNDATION-JUSTIFIED-BUT-REVERSIBLE-permanent |
| Boundary lint (dependency-cruiser) | **FOUNDATION-JUSTIFIED-BUT-REVERSIBLE** | Cheap, evidence-motivated (SnagTime's forensic enrolment-list failure) but not strictly required for a first slice to exist — its absence would not block development, only weaken a guardrail |
| `packages/contracts` | **FOUNDATION-JUSTIFIED-BUT-REVERSIBLE** | Phase-8A-approved rationale exists, but the package boundary itself (vs. types living directly in `apps/web`/`apps/api` until a second consumer needs sharing) is a convenience, not a requirement for a first slice |
| `packages/config` (as convention) | **FIRST-FEATURE-REQUIRED** | Security Floor item 15's boot-gate is required before *any* real code with secrets ships — but that is at the first-feature boundary, not strictly at WP0's local-scaffolding boundary, since WP0 itself need not run in production |
| `packages/time` (as local module) | **FIRST-FEATURE-REQUIRED** | Not needed until the first booking-domain slice constructs wall-clock times (WP6) — correctly deferred by the plan's own gating, though the *discipline* (explicit result types) is worth establishing as a convention now |
| `packages/kernel` | **PREMATURE** | No cited requirement, no concrete first consumer, confirmed independently by this audit (§4) |
| `packages/obs` | **PREMATURE** | Same finding, clearest case |
| Playwright e2e scaffolding | **PREMATURE** | No page exists yet; the Adversarial Review's own finding, confirmed here — infrastructure front-loaded ahead of anything to exercise it |
| Docker (three entrypoints) | **FOUNDATION-JUSTIFIED-BUT-REVERSIBLE** | Required eventually for deployment, but WP0's local-only scope does not need it to run anything — reversible to adjust before first real deploy |

---

## 7. WP0 authorization boundary

| Work | Allowed now | Must wait |
|---|---|---|
| Local repository initialization | Yes | — |
| Directory structure (`apps/`, `modules/` empty, `packages/`, `migrations/`, `test/`, `docker/`) | Yes | — |
| Workspace configuration (pnpm) | Yes | — |
| Dependency-cruiser (boundary lint) | Yes | — |
| `packages/contracts` (thin, types-only) | Yes | Any runtime logic or transport-specific type leakage — must wait for API-paradigm resolution |
| DB pool construction (`packages/db`) | Yes | — |
| Tenant context seam (mechanism-neutral naming only) | Yes, under renamed (`withTenantContext`/`withElevatedAccess`) form | Any RLS-specific or alternative-mechanism-specific internals |
| Jobs protocol scaffolding | Yes (protocol only, zero domain handlers) | Domain job handlers (belong to modules, post-foundation) |
| Config (as convention, not package) | Yes for the convention | The exact boot-gate refusal conditions require the Security Floor process (WP8, gated) |
| Time (as local module, explicit result types) | Yes for the module shell/discipline | Any code that actually constructs/interprets local wall-clock booking times (WP6, gated on local-time policy) |
| Migrations (tooling wiring) | Yes | — |
| Tenant-scoped schema | No | Until the owner selects the database-level tenant-isolation backstop (blocker 1) |
| Durable-job schema (migration) | Yes, as a confirm-not-decide draft against spike 4's specification | Applying the migration requires owner sign-off (Authority/Risk/Routing Model's migration row, "Yes, always") |
| Test harness (existence, real-Postgres, template-clone) | Yes | Confirming/correcting its mechanism-neutrality before treating it as settled (currently UNKNOWN, not blocking existence, but blocking the "settled" claim) |
| Docker | Yes (local build/config only) | Any actual deployment or registry push |
| Playwright | Recommended to defer (PREMATURE per §6), not technically blocked | First real page existing |
| CI | No | Blocker 3 (repository hosting/CI provider), owner decision |
| Authentication | Architecture-level scaffolding only (WP7, identity/membership/authorization separation) may proceed under the standing security-sensitive gate | Concrete library selection; any real endpoint before the Security Floor process runs |
| Booking code | No | Tenant-isolation mechanism (blocker 1) and local-time policy (correction 6) both resolved |

---

## 8. Evidence requirements

For each HIGH-risk WP0-adjacent activity, per the Verification Evidence Format:

| Activity | Required evidence | Test/command | Expected result | Runs locally? | Requires tenant mechanism selected? | Requires real repo? | Requires CI? | Status |
|---|---|---|---|---|---|---|---|---|
| Boundary-lint enforcement | Deliberate violation fires the lint rule | dependency-cruiser run against a test violation (e.g. `apps/web` importing `packages/db`) | Lint fails, non-zero exit | Yes | No | No (works in a local repo) | No | **Planned, not yet run** — no repository exists |
| Fail-closed enrolment (mechanism-neutral) | Deliberately unenrolled table causes denial, not silent pass | Integration test against the harness | Denial, not silent success | Yes | No (testable against a synthetic table) | No | No | **Planned, not yet run** |
| Tenant-context leakage | Query issued without the seam fails | Integration test | Query fails/errors | Yes | No (testable generically) | No | No | **Planned, not yet run — named as the single highest-priority missing proof by correction 11** |
| Worker-role narrowness | Worker's DB role cannot perform web-app-only operations | Spike 1 criterion 3, reproduced against real application code | Denied write/read outside worker's scope | Yes | No | Yes, for "real application code" fidelity (spike itself used a disposable instance, not the repo) | No | **Actual evidence exists (spike 1, disposable instance); planned re-verification inside real app code not yet run** |
| Job-table migration up/down/up | Reversibility | `node-pg-migrate` up, down, up | No data loss, schema returns to identical state | Yes | No | Recommended (once repo exists) | No | **Planned, not yet run** |
| Job protocol reproduction | Claim/lease/fence/`SKIP LOCKED`/checkpoint reproduces spike 4 | Concurrency test, ≥2 workers | Exactly-once execution, stale-lease reclaim, incremental checkpoint survives | Yes | No | Recommended | No | **Actual evidence exists (spike 4, disposable instance); planned reproduction inside real app code not yet run** |
| Boot-gate refusal (3 specific bad configs) | Process refuses to start | Boot with missing signing secret / insufficiently-random key / unverified-TLS DB URL | Non-zero exit, no partial start | Yes | No | No | No | **Planned, not yet specified which three — the plan itself flags this as currently unnamed** |
| Secret handling | No secret committed/logged/exposed | grep/lint-based CI check | No match | Yes | No | Recommended (for CI wiring) | Eventually (WP1/WP9) | **Planned, not yet run** |
| DST forward/backward transition (inside real availability code) | Reproduces spike 3 | Construction test across a known DST boundary | Correct wall-clock result | Yes | No | Yes, for "actual availability/slot-generation code" per Phase 8A §22's exact wording | No | **Actual evidence exists (spike 3, disposable instance); Phase-8A-§22-required reproduction inside real code not yet run — this is the proof the original plan dropped and the correction pass restored** |
| CI actually runs the verification loop on push | Positive proof, not assumed | A real push triggers a real CI run | Green/red status visible | No (requires CI) | No | Yes | Yes | **Not yet possible — WP1 not authorized** |

**Distinction maintained throughout**: spikes 1–4's own results (disposable-instance
evidence, already captured in `4. Calquartz Spikes/evidence/`) are **actual evidence that
already exists**. Every item above marked "reproduced... inside real application code" is
**planned verification, not yet run**, because no Calquartz repository or application code
exists. This audit found no place where the plan or its correction pass blurs this
distinction — both consistently label the spike results as already-existing evidence and
the in-repo reproductions as future requirements.

---

## 9. Final judgment

### WP0 PRE-AUTHORIZATION STATUS

## READY WITH SPECIFIC CORRECTIONS REQUIRED

**Reason**: the Foundation Implementation Plan, as corrected by its own 2026-09-09
Adversarial Correction Pass, is substantially accurate against the authoritative sources
this audit independently re-verified (ADR-001, Phase 8A, the Security Floor, the Authority/
Risk/Routing Model, the Verification Evidence Format, Architecture Open Questions, and the
four executed spikes) — no contradiction between the plan and any authoritative source was
found. It is not yet READY FOR OWNER AUTHORIZATION outright because several corrections the
plan itself already recommends (renaming the tenant-context seam, downgrading premature
packages, naming the boot gate's exact three refusal conditions, stating the
`packages/contracts` transport-neutrality guard explicitly) have not yet been **applied to
actual filesystem structure or code**, since no repository exists — they remain
recommendations on paper, correctly labeled as such, not yet enacted.

**(1) Exact corrections required before authorization**:
- Rename `withTenant`/`withSystem` to mechanism-neutral names (`withTenantContext`/
  `withElevatedAccess` or equivalent) at the moment the seam is actually coded — not
  merely recommended in the planning document.
- Name the boot gate's exact three refusal conditions concretely (the plan currently
  defers this to WP8 without naming them; Security Floor item 15's evidence base
  supports naming: missing/placeholder/insufficiently-random secret, unverified-TLS
  database URL, active demo/local provider fallback).
- State explicitly, in `packages/contracts`'s own documentation once it exists, that its
  types must remain domain-shaped and transport-neutral, not REST/tRPC/GraphQL-coupled,
  until Phase 8A §19's API paradigm is resolved.
- Confirm or correct the integration-test harness's mechanism-neutrality (currently
  UNKNOWN, not yet checked against actual template-clone/connection/fixture code, since
  none exists).
- Carry the correction-7b caveat ("application permanently owns the job framework" is
  unsettled) into `packages/jobs`'s own code comments/README once written, not only into
  planning documents.

**(2) Exact components that may be authorized now**: local repository initialization;
directory structure; pnpm workspace; dependency-cruiser configuration; `packages/kernel`
and `packages/obs` as local files; `packages/config` as a per-entrypoint convention;
`packages/db`'s pool + tenant-context seam under mechanism-neutral naming; `packages/jobs`'s
protocol scaffolding (caveated); `packages/time` as a local module with explicit result
types; `packages/contracts` as a thin, types-only package; the integration-test harness's
existence (pending the mechanism-neutrality confirmation above); local Docker
configuration; the durable-job migration as a drafted-for-confirmation artifact (not yet
applied).

**(3) Exact components that must remain prohibited**: any tenant-scoped schema/migration
depending on the final isolation mechanism; any code constructing or interpreting local
wall-clock booking times; any remote repository push or CI activation; any worker-role
credential provisioning without the standing secrets/credentials human gate; applying the
durable-job migration without owner confirmation; any concrete auth library selection or
real authentication endpoint before the Security Floor process runs.

**(4) Unresolved owner decisions**: the tenant-isolation mechanism (RLS vs. an
alternative); repository hosting and CI provider; the ambiguous/nonexistent-local-time
product policy (a default is recommended, not decided); confirmation (not design) of the
durable-job table's exact schema.

**(5) Hidden decisions found**: RLS — **POSSIBLE**, via the `withTenant`/`withSystem`
two-mode seam shape (§5), already disclosed by the plan's own correction 5 and
independently confirmed by this audit, not newly discovered but not yet remediated in
actual code since none exists. API paradigm / public-API contract shape — **POSSIBLE**,
via `packages/contracts`'s potential type-shape leakage, identified independently by this
audit and not previously named in the plan or its reviews.

**(6) Premature abstractions found**: `packages/kernel` and `packages/obs` as standing
packages (no cited requirement at all, confirmed independently); Playwright e2e
scaffolding ahead of any real page.

**(7) Verification gaps**: no code exists yet to run any of the nine required proofs in §8
against; the boot gate's exact three refusal conditions remain unnamed; the test harness's
mechanism-neutrality remains unconfirmed; no Verification Evidence Record has been produced
for any foundation-adjacent work, since no work has begun.

**(8) Whether the corrected plan itself now accurately reflects the evidence gathered in
this audit**: **yes, substantially.** This audit's independent re-verification of every
authoritative-source citation in the plan and its correction pass (ADR-001, Phase 8A, the
Security Floor, the Authority/Risk/Routing Model, the Verification Evidence Format, the
Open Questions, and all four spike results) found no factual contradiction. The plan's own
honesty about its residual limitations (§6 of the original plan; the WP0–WP9 file's own
"Provenance" section) is corroborated, not undermined, by this audit. The gaps this audit
adds beyond what the plan and its existing Adversarial Review already found are narrow: the
`packages/contracts` transport-neutrality risk (§4, §5, §9(1)) and the explicit naming of
the boot gate's three refusal conditions (§9(1)) — both incremental refinements, not
corrections to anything materially wrong in the current plan.

---

## Related

- [[Calquartz — Application Foundation Implementation Plan]]
- [[Calquartz — Foundation Work Packages (WP0–WP9)]]
- [[Calquartz — Application Foundation Implementation Plan — Adversarial Review]]
- [[../decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]]
- [[Phase 8A — Implementation Stack Selection]]
- [[Authority, Risk, and Routing Model]] · [[Verification Evidence Format]]
- [[../architecture/Security Floor|Security Floor]]
- [[../questions/Architecture Open Questions|Architecture Open Questions]]
- [[../Calquartz — Pre-Approval Technical Spikes|Pre-Approval Technical Spikes]]
