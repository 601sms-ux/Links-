---
type: synthesis
created: 2026-09-08
updated: 2026-09-08 (owner approval recorded, Phase 8B, same day)
sources: ["wiki/projects/001-calendar-os/decisions/ADR-001 — A-B1 Modular Monolith Architecture.md", "wiki/projects/001-calendar-os/Calquartz — Architecture Decision Brief.md", "wiki/projects/001-calendar-os/architecture/Architecture Recommendation.md", "wiki/projects/001-calendar-os/architecture/Architecture Requirements and Constraints.md", "wiki/projects/001-calendar-os/questions/Architecture Open Questions.md", "wiki/projects/001-calendar-os/Calquartz — Pre-Approval Technical Spikes.md", "wiki/projects/001-calendar-os/architecture/Phase 6 — Independent Adversarial Architecture Review.md", "wiki/projects/001-calendar-os/engineering/Phase 5B — Factual and Technical Prerequisites.md", "4. Calquartz Spikes/evidence/00-environment-setup.md", "4. Calquartz Spikes/evidence/spike-1-tenant-isolation-evidence.md", "4. Calquartz Spikes/evidence/spike-2-booking-invariant-evidence.md", "4. Calquartz Spikes/evidence/spike-3-dst-evidence.md", "4. Calquartz Spikes/evidence/spike-4-durable-jobs-evidence.md", "wiki/projects/001-calendar-os/architecture/Database-Level Tenant Isolation — RLS and Alternatives.md", "wiki/projects/001-calendar-os/architecture/Failure-Mode Analysis.md", "wiki/projects/001-calendar-os/architecture/Threat Model.md", "wiki/projects/001-calendar-os/architecture/Multi-Tenancy Trust Model.md", "wiki/projects/001-calendar-os/architecture/Security Floor.md", "wiki/projects/001-calendar-os/architecture/Infrastructure Constraint Analysis.md", "wiki/projects/001-calendar-os/architecture/Calquartz — VPS Infrastructure Inventory.md", "wiki/projects/001-calendar-os/architecture/Redis Justification Analysis.md", "wiki/projects/001-calendar-os/engineering/Durable Decision Capture Policy.md", "wiki/projects/001-calendar-os/engineering/Verification Evidence Format.md"]
tags: [engineering, stack-selection, phase-8a, calendar-os, calquartz, decision]
confidence: medium
provenance: hand-authored, Phase 8A, following the same FACT/OBSERVATION/INFERENCE/RECOMMENDATION/UNKNOWN discipline as the rest of the project; owner-approval note appended Phase 8B, 2026-09-08
classification: DECISION — stack selection only (§18 element list), owner-approved 2026-09-08. Every unresolved item in §19/§20 remains explicitly UNKNOWN, not upgraded by this approval.
scope: calquartz-specific
status: ACTIVE — stack approved by owner 2026-09-08 (Phase 8B). Implementation-subdetail UNKNOWNs in §19–20 remain OPEN.
---

# Phase 8A — Implementation Stack Selection

## 1. Phase Objective

Select the implementation stack for Calquartz v1: backend/runtime, PostgreSQL access
layer, date/time library, durable-job implementation approach, frontend, auth/authz
approach, testing stack, and deployment shape — evaluated against actual Calquartz
requirements and Phase 7 spike evidence, not popularity. This is a **stack-selection and
engineering-foundation planning phase only**. No Calquartz code is written, no repository
is created, no infrastructure is touched. The output is a RECOMMENDATION requiring owner
approval, exactly as ADR-001 required for the architecture itself.

## 2. Inputs and Authoritative Sources

Read in full before this document was drafted: `CHATGPT_HANDOFF_CONTEXT.md`; the vault
root `CLAUDE.md`; the project's own `CLAUDE.md` (§46 Durable Decision Capture pointer,
§47 current-state pointer); [[../decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]];
[[../Calquartz — Architecture Decision Brief|Architecture Decision Brief]];
[[../architecture/Architecture Recommendation|Architecture Recommendation]];
[[../architecture/Architecture Requirements and Constraints|Architecture Requirements and Constraints]];
[[../questions/Architecture Open Questions|Architecture Open Questions]];
[[../Calquartz — Pre-Approval Technical Spikes|Pre-Approval Technical Spikes]] (all five
Result subsections); [[../architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6]];
[[Phase 5B — Factual and Technical Prerequisites|Phase 5B]]; the five spike evidence files
under `4. Calquartz Spikes/evidence/`; [[../architecture/Database-Level Tenant Isolation — RLS and Alternatives|Database-Level Tenant Isolation]];
[[../architecture/Failure-Mode Analysis|Failure-Mode Analysis]];
[[../architecture/Threat Model|Threat Model]];
[[../architecture/Multi-Tenancy Trust Model|Multi-Tenancy Trust Model]];
[[../architecture/Security Floor|Security Floor]];
[[Calquartz Booking Correctness — Operational Review Process|Booking Correctness — Operational Review Process]];
[[Calquartz Security Floor — Operational Review Process|Security Floor — Operational Review Process]];
[[Authority, Risk, and Routing Model|Authority, Risk, and Routing Model]];
[[Durable Decision Capture Policy|Durable Decision Capture Policy]];
[[../README|project README]]; `log.md` tail (Phases 5A–7); [[../architecture/Infrastructure Constraint Analysis|Infrastructure Constraint Analysis]]
and [[../architecture/Calquartz — VPS Infrastructure Inventory|VPS Infrastructure Inventory]];
[[../architecture/Redis Justification Analysis|Redis Justification Analysis]];
[[Verification Evidence Format|Verification Evidence Format]] (for classification
convention). No listed source was found missing.

## 3. Phase 7 Closure

Bounded closure check against the required criteria:

- **All four spikes recorded as executed.** FACT — [[../Calquartz — Pre-Approval Technical Spikes|Pre-Approval Technical Spikes]] carries a dated "Result (2026-09-08, Phase 7)" subsection for spikes 1–4, each citing its evidence file under `4. Calquartz Spikes/evidence/`. All four evidence files exist and were read.
- **Hypotheses/results correctly classified.** FACT. Spike 1 result: "RLS remains the recommended candidate — not selected as a decision." Spike 2 result: "Recommended: the exclusion constraint... Recommendation, not a decision." Spike 3: states a hypothesis "CONFIRMED empirically" (an empirical result, correctly left as OBSERVATION/FACT, not escalated to a library selection — explicitly: "Subsumed by spike 3, which tests the construction, not the library"). Spike 4: describes what "was tested and requires no new mechanism" (empirical), without naming a job-framework product. No spike improperly promotes its finding to a stack decision.
- **No spike recommendation has silently become a DECISION.** Confirmed — none of the four Result subsections, nor ADR-001's "Update (2026-09-08, Phase 7)" paragraph, uses the word "DECISION" for any spike outcome; ADR-001 itself explicitly states spikes 1–4 are "implementation constraints and recommendations, not architecture or production-stack decisions."
- **ADR-001 unchanged as approved direction.** Confirmed — ADR-001's frontmatter status line and body are unmodified by this phase; this document only reads it.
- **No stale doc claims spikes unexecuted.** Checked [[../Calquartz — Architecture Decision Brief|Architecture Decision Brief]] and [[../architecture/Phase 6 — Independent Adversarial Architecture Review|Phase 6]] — both were already updated in the Phase 7 pass and correctly reflect execution. No contradiction found.
- **Remaining UNKNOWNs genuinely unknown.** Confirmed by spot check: the Security Floor's 16 items remain `UNKNOWN` per ADR-001's own "Implementation gates" section (no Calquartz code exists, so this is correct, not stale).
- **Phase 7 implementation constraints propagated to authoritative homes.** Partially — spike results live in the spikes document and are cross-referenced from ADR-001, but had not yet been folded into [[../architecture/Architecture Requirements and Constraints|Architecture Requirements and Constraints]] as a discrete "implementation constraints" section. This is a genuine, minor gap, not a contradiction. **Correction applied by this phase**: see §C of the accompanying updates (Deliverable C below) — an additive section was appended there rather than reopening spike work.
- **Disposable PostgreSQL environment is gone; no VPS/production changes; no ECC installation.** Confirmed — `4. Calquartz Spikes/evidence/00-environment-setup.md` and the spikes document's "Boundary" section both state the disposable instance used for spikes 1/2/4 was deleted after evidence capture. No evidence anywhere of VPS mutation or ECC installation during Phase 7.

**Conclusion**: Phase 7 closure is clean. One minor propagation gap was found (spike
findings not yet cross-referenced into Architecture Requirements and Constraints) and
corrected additively (Deliverable C). No spike work was reopened. No contradictions
required resolution beyond that one addition.

## 4. Calquartz Stack Requirements

Derived from ADR-001, the Architecture Requirements and Constraints, and the four spike
Results — not from convention or popularity:

- One VPS: 2 CPU / ~7.6 GiB RAM / ~99 GB disk (FACT, [[../architecture/Calquartz — VPS Infrastructure Inventory|VPS Infrastructure Inventory]]), currently idle with large headroom, no additional infrastructure purchase in v1.
- One deployable process family: web + worker, per A/B1 (ADR-001). No second machine, no Kubernetes.
- PostgreSQL is the sole authoritative datastore. No Redis in v1 (deferred, [[../architecture/Redis Justification Analysis|Redis Justification Analysis]]).
- Must support: RLS-driven transaction-local tenant context surviving pool reuse under transaction-mode pooling (spike 1); a `tstzrange` exclusion constraint over booking intervals, atomic all-or-nothing or explicit-partial-success recurring-series writes as an *application transaction-boundary* choice (spike 2); wall-clock-correct slot construction across DST transitions with explicit handling of nonexistent/ambiguous local times (spike 3); a Postgres-backed job table with atomic claim, lease, fencing token, `SKIP LOCKED`, idempotent effect application, and incremental checkpointing for a scheduled cross-tenant batch job (spike 4).
- Recurring bookings required in v1 (owner decision) — the data model and job system must both be designed for this from day one, not retrofitted.
- Hosted-only, flat tenancy (owner decisions) — no self-hosting hardening, no org-of-orgs schema needed in v1.
- Solo, AI-assisted operator — favours one coherent language/runtime, strong type safety, minimal moving parts, and libraries an AI agent can reason about and test reliably over maximal ecosystem breadth.
- Payments/subscriptions/billing, Redis, a separate booking-engine service, and self-hosting are all explicitly out of v1 scope; the stack must not be shaped around them prematurely, but should not structurally foreclose them either (per ADR-001's designed-for extraction seams).

## 5. Candidate Stacks

Following the "small number of credible candidates, not a giant framework comparison"
instruction, three backend/runtime candidates and two frontend pairings were considered,
scoped to what plausibly clears the hard-rejection criteria (§7) against the requirements
above:

- **Candidate 1 — TypeScript/Node.js, Fastify or a minimal Express-class server, Postgres via `node-postgres` (`pg`) with raw SQL + a lightweight query builder (Kysely), migrations via a dedicated tool (e.g. `node-pg-migrate` or Kysely's migration support).**
- **Candidate 2 — TypeScript/Node.js, same runtime, but Prisma ORM as the primary data-access layer.**
- **Candidate 3 — Python, FastAPI, SQLAlchemy Core/ORM with Alembic migrations, `asyncpg`/`psycopg` driver.**

Rejected before full evaluation, and why (not full candidates, per the "no giant
framework comparison" instruction): Ruby on Rails / Django full-stack MVC frameworks
(their built-in ORMs — ActiveRecord, Django ORM — historically weak, awkward-to-impossible
support for `tstzrange` exclusion constraints and transaction-local `SET LOCAL` session
variables the way spike 1/2 require without dropping to raw SQL constantly, which would
defeat the point of choosing an ORM-centric framework); Go (materially slower AI-assisted
iteration for a solo operator per the project's own stated optimization order, and no
clear correctness advantage over the TypeScript/Python candidates given PostgreSQL does
the correctness-critical work in every candidate); a serverless/edge-function topology
(actively fights the one-VPS, one-deployable A/B1 shape ADR-001 already approved — would
require reopening the architecture, not merely picking a stack).

Frontend candidates, paired with either backend: **Next.js (React) with a shared
TypeScript type boundary to a TS backend**, or **a separate SPA (React/Vite) calling a
versioned API**, evaluated in §10.

## 6. Backend Evaluation

| | Candidate 1 (TS/Fastify + Kysely/raw SQL) | Candidate 2 (TS/Fastify + Prisma) | Candidate 3 (Python/FastAPI + SQLAlchemy) |
|---|---|---|---|
| Raw-SQL/PG-feature escape hatch | Native — Kysely is a thin typed query builder over SQL; raw SQL is a first-class, unceremonious path | Present but historically friction-prone for PG-specific DDL (exclusion constraints, `SET LOCAL`) — routinely requires `$queryRaw`/`$executeRaw` and migration workarounds | Native — SQLAlchemy Core executes near-raw SQL comfortably; Alembic migrations support raw DDL directly |
| Transaction-local session var (`SET LOCAL app.tenant_id`) for RLS | Straightforward — the query builder does not own transaction lifecycle, so wrapping a `pg` client transaction and issuing `SET LOCAL` first is direct application code | Documented but awkward: Prisma's connection/transaction abstraction (`$transaction`) has known friction with `SET LOCAL` correctly landing in the same transaction/connection under a pool — a genuine, recorded weak point for RLS-style patterns, not a rumour | Straightforward — SQLAlchemy `Connection`/session-scoped transactions map directly onto issuing `SET LOCAL` first |
| `tstzrange` exclusion constraint | Fully expressible via raw SQL DDL in any migration tool | Expressible only via a raw-SQL migration (Prisma schema DSL has no native exclusion-constraint syntax) — works, but the ORM's own schema-as-source-of-truth model is undermined for this one table | Fully expressible via Alembic raw DDL |
| Type sharing with frontend | Excellent if paired with a TS frontend — one language, shared types end-to-end | Excellent, same reason, plus Prisma's generated client types | None natively — needs an OpenAPI/schema-generation step (FastAPI does this well) to share types with a TS frontend |
| AI-agent maintainability | High — TypeScript's static types plus a thin, explicit query layer are easy for an AI agent to reason about without hidden ORM magic | Medium-high day-to-day, but the RLS/exclusion-constraint friction above means an AI agent will repeatedly need raw-SQL escape hatches inside an ORM-centric codebase, a worse fit for consistency | High — Python is well-represented in AI training data and FastAPI's explicitness is comparable to Fastify's |
| Ecosystem maturity | Mature; Kysely is younger than Prisma but stable and widely used for exactly this "thin layer over PG-specific SQL" niche | Very mature, most popular TS ORM | Mature; SQLAlchemy is the dominant Python data layer |

**FACT**: Prisma's transaction/connection-pooling interaction with PostgreSQL
session-local settings (`SET LOCAL`) is a widely-documented friction point in the Prisma
ecosystem, not evidence collected by this project's own spikes (spike 1 used raw `pg`
directly, per its evidence file — it did not test Prisma). This is flagged as
**INFERENCE, not spike-tested**: spike 1 proves RLS-plus-`SET LOCAL` works under real
pooling *at the raw-driver level*; it does not prove or disprove any specific ORM's
compatibility with that pattern. This gap is named explicitly in §21 (adversarial
self-check) and §22 (verification requirements).

## 7. Database/Data-Access Evaluation

Both TypeScript candidates and the Python candidate can express every spike-tested
mechanism (RLS, transaction-local context, exclusion constraint, advisory/row locks,
`SKIP LOCKED`) via raw SQL or a thin query-builder layer. Candidate 2 (Prisma) is
downgraded, not rejected, specifically because its ORM abstraction actively works against
three of the four spike-validated mechanisms (RLS context propagation, exclusion-constraint
DDL, and comfortable raw-SQL escape for job-table `SKIP LOCKED` queries), forcing an
AI-assisted developer to fight the tool on exactly the subsystems this project has already
spent four spikes validating. Candidates 1 and 3 both treat raw SQL as first-class, which
directly satisfies question 3 in §8 of the task spec (exclusion constraints, RLS,
transaction-local context, explicit transactions, row locks, raw SQL — all present without
awkward workarounds in both).

Connection pooling and tenant context (question 4): in both surviving candidates, the
recommended pattern is application-level pool management (e.g. `pg.Pool` /
`asyncpg.Pool`) with the tenant-context `SET LOCAL` issued as the first statement inside
every transaction that touches tenant-scoped tables — exactly the pattern spike 1
empirically validated. This is a **RECOMMENDATION carried forward from spike 1's
mechanism**, not a new claim.

## 8. Date/Time Evaluation

Spike 3 tested the *construction*, not a specific library (by design — the spikes
document explicitly excludes "date-library selection" as a separate spike, subsumed into
spike 3). Two constructions were demonstrated correct: a direct wall-clock build, and a
Cal.diy-style offset correction. For a TypeScript backend, **Luxon** (already the library
spike 3 tested against, with both correct and incorrect construction patterns
demonstrated in the same ecosystem) or the newer, IANA-native **Temporal** proposal
(via a polyfill, since native runtime support is not yet universal) are the credible
candidates; for Python, **`zoneinfo` + a small wall-clock-construction helper**, or
`pendulum`. **RECOMMENDATION**: Luxon for a TypeScript backend, specifically *because*
spike 3's evidence base already demonstrates both the hazard and the fix in Luxon
directly — an AI-assisted developer inherits a validated, in-project example of the
correct pattern rather than needing to re-derive it against an unfamiliar library's
semantics. This is explicitly **not** a claim that Luxon is superior in the abstract; it
is a claim that reusing the already-validated construction pattern lowers correctness
risk relative to introducing an untested library.

Nonexistent/ambiguous local times (task spec §5, §9): spike 3 found Luxon silently
resolves both cases with no warning. **RECOMMENDATION**: Calquartz must not rely on
library defaults for these cases — an explicit application-layer policy (e.g. reject
nonexistent local start times at booking-creation time with a user-facing error, and
require an explicit UTC-offset disambiguation for ambiguous local times) must be a named,
tested requirement, not left to whatever the library happens to do. This is a genuine
**UNKNOWN / open design question** this document does not resolve, consistent with §9 of
the task spec (do not prematurely resolve product questions) — it is surfaced, not
answered, here.

## 9. Durable-Job Evaluation

Spike 4 validated the claim/lease/fence/`SKIP LOCKED` protocol directly against a
hand-built PostgreSQL job table, not against any named job-framework library. **No
existing job framework was spike-tested.** Two implementation paths are credible:

1. **Hand-rolled job table**, implementing exactly the protocol spike 4 validated (claim
   via `UPDATE ... WHERE status='pending' ... RETURNING`, lease expiry, fencing token,
   `SKIP LOCKED`, idempotent effect log, incremental batch checkpointing). Highest fidelity
   to spike-tested evidence; more code to write and maintain.
2. **A thin durable-job library built on the same primitives** (e.g. `pg-boss` for
   Node/Postgres, or `graphile-worker`) — both use `SKIP LOCKED`-based Postgres claiming
   internally and no Redis, compatible with the A/B1/no-Redis constraint. Neither has been
   spike-tested against the *specific* fencing-token and incremental-batch-checkpoint
   requirements spike 4 validated as necessary for the recurring-materialization batch
   shape.

**RECOMMENDATION**: start from the hand-rolled protocol (path 1), because it is the only
option with direct spike evidence behind every required property (fencing token, graceful
shutdown without retry-budget consumption, mid-batch checkpoint-and-resume). Evaluate a
library (path 2) only after path 1's hand-rolled version is running and its operational
cost is measured — adopting an unvalidated library's claim semantics on faith would
re-introduce exactly the risk the spikes were run to retire. This directly answers task
spec question 7 (§8): the protocol is implemented as application-layer Postgres SQL
against a dedicated job table, following spike 4's validated shape, not delegated to an
untested third-party queue product.

## 10. Frontend Evaluation

Both frontend candidates (Next.js/React monolith-adjacent, or a separate React/Vite SPA)
are compatible with either surviving backend. **RECOMMENDATION**: Next.js (React), paired
with the TypeScript backend candidate (Candidate 1), for end-to-end TypeScript type
sharing without a separate schema-generation step, and because Next.js's App
Router/server-component model fits cleanly inside a single deployable web process — no
separate frontend server or CDN-origin split is required at v1 scale, keeping the
deployment shape as simple as A/B1 already commits to. If the backend candidate were
Python (Candidate 3), the frontend would need FastAPI's OpenAPI-schema-generation path to
approximate the same type-sharing benefit — a real but secondary reason favouring the
TypeScript pairing overall.

## 11. Authentication/Authorization Evaluation

Requirement: identity, tenant membership, and authorization kept conceptually separate
(project charter §14–15; Multi-Tenancy Trust Model). **RECOMMENDATION**: a session-based
auth mechanism (signed, HTTP-only, secure cookies) for the primary web application, backed
by a dedicated `users` table (identity) and a separate `memberships` table (tenant + role,
distinct from identity) — matching the charter's own explicit separation. A mature,
actively maintained auth library appropriate to the chosen backend (for TypeScript:
something in the Lucia/Auth.js family, evaluated at implementation time, not selected
here) is preferable to hand-rolling session/cookie security primitives, given the Security
Floor's session-security and CSRF/XSS items. **This document does not select a specific
library** — task spec §9 lists authentication/authorization only as a category to evaluate
capability against, and no spike tested any specific library; naming one here would be
exactly the kind of premature resolution §9 of the task spec warns against for product
questions, and the same caution applies to unvalidated security-critical libraries.
Capability check: both backend candidates can express session cookies, CSRF tokens, and a
role-check middleware/decorator cleanly: **PASS (capability only, not implementation
verification)**.

## 12. Testing Evaluation

Requirement: unit, integration against real PostgreSQL (not mocks — spikes 1/2/4 all ran
against real PostgreSQL specifically because mocked behaviour would not have caught the
pooling/DST/concurrency defects found), concurrency, DST/timezone, recurring-series,
security/tenant-isolation, E2E, CI practicality. **RECOMMENDATION**: for the TypeScript
candidate, Vitest or Node's built-in test runner for unit tests, with integration tests
run against a real, disposable PostgreSQL instance (Testcontainers-style, or a CI-managed
Postgres service container) — mirroring exactly the spike methodology already validated
in Phase 7, not mocking PostgreSQL. Playwright for E2E (the charter's own required
register→...→cancel flow). Concurrency and tenant-isolation tests should be written as
direct descendants of the spike 1/2/4 test harnesses already produced as evidence, not
reinvented — this is the single largest concrete testing-quality advantage the executed
spikes hand to implementation: working, validated test patterns already exist.

## 13. Deployment Evaluation

A/B1 already fixes the shape: one deployable image, a web process and a worker process,
one PostgreSQL instance, no Redis, no Kubernetes. **RECOMMENDATION**: a single Docker
image built from the chosen backend, run as two processes (web, worker) via a process
manager or two container instances from the same image on the existing VPS, PostgreSQL
either containerized alongside or (preferable for backup/restore simplicity) run as a
host-level service. Health/readiness endpoints, migrations run as an explicit deploy step
(not implicit at boot), a documented rollback path (previous image tag + a reversible
migration discipline), and automated PostgreSQL backups with a tested restore procedure —
all named requirements from the charter (§37–38) and the task spec (§5), none yet
implemented, none decided beyond "this shape" here.

## 14. Security Implications

**No claim that the Security Floor has passed — no Calquartz code exists.** Capability
assessment only, per task spec §10: both surviving backend candidates support parameterized
queries (SQL-injection resistance), HTTP-only/secure cookies, CSRF-token middleware, CORS
configuration, rate-limiting middleware/libraries, structured logging with redaction
control, and standard security-header middleware (Helmet-class for Node, equivalent for
Python) without structural obstruction. Worker-role privilege separation is directly
supported by the same mechanism spike 1 validated (a narrower database role) — this is
spike-tested, not merely asserted. Dependency/security maintenance: both ecosystems have
mature automated-audit tooling (`npm audit`/Dependabot; `pip-audit`). **Risk not
hand-waved**: the Prisma-vs-raw-SQL friction noted in §6 is itself a security-adjacent
concern — an ORM that pushes developers toward raw-SQL escape hatches for RLS-critical
paths increases the chance of a hand-written query missing tenant-context setup, which is
exactly the SnagTime failure mode (hand-maintained enrollment list, fails open) ADR-001
and spike 1 were designed to avoid reproducing. This is a concrete reason, not a vague
preference, behind recommending Candidates 1/3 over Candidate 2.

## 15. Complexity/Solo-Operator Analysis

One language end-to-end (TypeScript, Candidate 1 + Next.js) minimizes context-switching
for a solo AI-assisted operator and lets test/type infrastructure span frontend and
backend uniformly. The thin-query-builder-over-raw-SQL approach (Kysely) trades some
developer convenience (less auto-generated boilerplate than Prisma) for materially lower
risk on the exact subsystems (RLS, exclusion constraints, job-table SQL) this project has
already spent real engineering evidence validating — judged the correct trade for this
project's stated priority order (correctness and tenant isolation ahead of maintainability
convenience, per task spec §3).

## 16. Reversibility Analysis

- **Backend language/runtime**: expensive to change once written — effectively a rewrite. Highest-stakes choice in this document.
- **Query builder (Kysely) vs. hand-written SQL**: cheap to change; Kysely is a thin layer, not a schema-owning framework — migrating off it later touches call sites, not data.
- **Date/time library**: moderate cost — isolated to a date-utilities module if designed as one from the start (a stated implementation precondition, §21).
- **Job implementation (hand-rolled vs. library)**: moderate — the job table's schema is the expensive part to change; the claim-loop code around it is more replaceable.
- **Frontend framework**: moderate-to-high, but decoupled from backend correctness — a frontend rewrite does not touch booking/tenant-isolation correctness.
- **Auth library**: moderate — session/cookie mechanics are swappable behind the identity/membership/authorization boundary the charter already requires be kept separate.
- **Deployment shape**: low cost to adjust within A/B1's fixed process count; high cost only if it required reopening ADR-001 itself, which nothing here proposes.

## 17. Candidate Rejection Reasons

- **Prisma-centric stack (Candidate 2)**: not hard-rejected — no requirement is
  structurally impossible with Prisma — but **downgraded below Candidates 1 and 3**
  because its ORM abstraction fights three of the four spike-validated mechanisms
  (documented Prisma/RLS transaction friction, no native exclusion-constraint DSL, raw-SQL
  escape hatch friction for job-table queries), increasing the risk of exactly the
  "developer forgets the tenant-scoping discipline" failure this project has spent four
  spikes specifically trying to avoid.
- **Rails/Django-class full-stack MVC frameworks**: rejected before full evaluation — their
  bundled ORMs share Prisma's weaknesses on PG-specific DDL and transaction-local settings,
  without offering a comparable AI-assisted-development or type-sharing advantage to offset
  it.
- **Serverless/edge-function topology**: rejected — directly conflicts with A/B1's approved
  one-deployable, web+worker process shape; adopting it would require reopening ADR-001,
  which is out of this phase's scope.
- **Go**: rejected before full evaluation — no correctness advantage given PostgreSQL
  carries the correctness-critical work in every candidate, and materially slower
  AI-assisted iteration for a solo operator, contrary to the task spec's stated
  optimization order.

## 18. Recommended Stack

**RECOMMENDATION — OWNER APPROVAL REQUIRED, NOT A DECISION:**

- **Backend**: TypeScript, Node.js, Fastify (or an equivalently minimal HTTP framework).
- **Database access**: `pg` (node-postgres) as the driver, Kysely as a thin typed query
  builder, raw SQL for exclusion constraints, RLS policies, `SET LOCAL` tenant context, and
  job-table `SKIP LOCKED` queries. Migrations via a dedicated SQL-first migration tool
  (e.g. `node-pg-migrate`).
- **Date/time**: Luxon, using the wall-clock-correct construction pattern spike 3 already
  demonstrated, with an explicit application-layer policy for nonexistent/ambiguous local
  times (design pending — named UNKNOWN, §20).
- **Durable jobs**: a hand-rolled Postgres job table implementing the exact claim/lease/
  fence/`SKIP LOCKED`/idempotent-effect-log protocol spike 4 validated, including
  incremental checkpointing for the recurring-occurrence-materialization batch shape.
- **Frontend**: Next.js (React), same TypeScript codebase family, sharing types with the
  backend directly.
- **Auth**: session-based, HTTP-only secure cookies, `users`/`memberships` tables kept
  conceptually and schematically separate; specific library selection deferred to
  implementation planning (not this document).
- **Testing**: Vitest/Node test runner for unit tests; integration tests against a real,
  disposable PostgreSQL instance (mirroring the spike methodology); Playwright for E2E;
  concurrency/tenant-isolation/DST tests built as direct descendants of the spike 1/2/3
  test harnesses.
- **Deployment**: single Docker image, web + worker processes, PostgreSQL as a host-level
  or co-located service, explicit migration step at deploy time, documented rollback, and
  automated backups with a tested restore procedure — on the existing single VPS, no
  additional infrastructure purchase.

## 19. Explicit Non-Decisions

This document does **not** resolve, and explicitly leaves open: the specific auth library;
the exact nonexistent/ambiguous-local-time product policy; the recurring-series atomicity
model (all-or-nothing vs. explicit partial success — spike 2 confirmed this is an
application transaction-boundary choice, not fixed by this stack selection); seats/
group-booking and round-robin scope; the canonical product name; the final calendar and
notification providers; public developer API scope; embeds; whether a job library
(`pg-boss`/`graphile-worker`) eventually replaces the hand-rolled protocol. The selected
stack preserves optionality on all of these: none requires a stack change to resolve later.

## 20. Remaining Unknowns

- Whether Prisma's documented RLS/transaction friction would actually manifest as a
  correctness problem in Calquartz's specific query patterns — untested by this project;
  only inferred from general Prisma-ecosystem reports, not from a Calquartz-specific spike.
- Whether a job library (`pg-boss`, `graphile-worker`) can be adapted to the exact
  fencing-token and incremental-checkpoint requirements spike 4 validated as necessary —
  not spike-tested; the hand-rolled recommendation exists specifically because this is
  unknown.
- The nonexistent/ambiguous local-time product policy (spike 3 surfaced, did not resolve).
- Auth library selection and its specific Security Floor compliance — capability only was
  assessed, not a specific library's implementation.
- Real production performance/load characteristics of any candidate on the actual 2 CPU/
  7.6 GiB VPS — no load testing has occurred; Phase 5B's headroom measurement is idle-state
  only.

## 21. Final Adversarial Self-Check

**Strongest argument against this recommendation**: it optimizes hard for fidelity to
already-validated spike patterns (raw SQL, hand-rolled job table) at some cost to
development velocity a more batteries-included stack (Prisma, a job library) might offer
a solo operator — the task spec's own priority order places maintainability/velocity
below correctness, but a reasonable reviewer could argue the velocity cost is
underestimated here, since it was not itself measured.

**Requirement this stack handles worst**: "maintainability for a solo AI-assisted
operator" specifically for the job table — a hand-rolled implementation is more code to
keep correct over time than an adopted library would be, and the AI-agent-maintainability
argument favouring it (spike-pattern fidelity) is real but not free.

**Dependency that could become a long-term liability**: Kysely is younger and has a
smaller ecosystem than Prisma; if it stalls or its maintenance slows, the migration cost
back to raw SQL is low (per §16) but the migration-tooling choice paired with it
(`node-pg-migrate`) would need separate evaluation.

**Assumption about the solo AI-assisted workflow**: that an AI agent reasons more
reliably about explicit SQL than about ORM-generated queries it cannot directly inspect —
plausible given this project's own SnagTime forensic finding (hand-maintained enrollment
list failing open) but not independently tested against Claude-generated code specifically
for this stack.

**Where the "ORM" (query builder) could hide/weaken PostgreSQL guarantees**: Kysely does
not hide transaction boundaries (a deliberate reason for choosing it), but a careless call
site could still issue a query outside the transaction that set `SET LOCAL` tenant
context — the tool does not prevent this by construction; it must be enforced by
convention/lint rule/code review, same as any raw-SQL approach.

**Where the date library could silently choose semantics**: exactly the nonexistent/
ambiguous-local-time gap named in §8/§20 — Luxon's default behaviour there is silent and
was flagged, not fixed, by spike 3.

**Where the job implementation could accidentally claim stronger guarantees than it
provides**: if the hand-rolled protocol's incremental-checkpoint commit discipline is
implemented sloppily (e.g. checkpointing only at batch completion instead of
incrementally, which spike 4's evidence explicitly says is "critically dependent" on
incremental commits), it would silently regress to the weaker, spike-4-tested-as-failing
shape without any visible signal.

**What would cause this stack to be reopened**: implementation evidence that Kysely (or
raw `pg`) cannot cleanly express a required PG feature discovered only during real schema
design; measured AI-assisted development velocity on the hand-rolled job table proving
materially worse than expected; or any of ADR-001's own four reopen triggers firing during
real implementation (none of which this stack choice can prevent or cause on its own).

**What evidence is still missing**: a Calquartz-specific (not general-ecosystem) test of
Prisma against the RLS/`SET LOCAL` pattern; any load/performance data on the actual VPS
under a realistic booking-and-job workload; and a resolved product policy for ambiguous/
nonexistent local times.

## 22. Verification Requirements

Per [[Verification Evidence Format|the Verification Evidence Format]] and
[[Authority, Risk, and Routing Model|the Authority, Risk, and Routing Model]], the first
implementation work following any owner approval of this recommendation must produce a
Verification Evidence Record for at least: (a) the RLS + `SET LOCAL` + Kysely/raw-`pg`
pattern reproducing spike 1's result inside real application code, not just the spike
harness; (b) the exclusion-constraint migration and booking-creation path reproducing
spike 2's result; (c) the Luxon wall-clock construction reproducing spike 3's result
inside the actual availability/slot-generation code; (d) the hand-rolled job table
reproducing spike 4's result, including the incremental-checkpoint batch case
specifically. No implementation gate in ADR-001 is satisfied merely by this document.

## 23. Decision Status

**RECOMMENDATION — OWNER APPROVAL REQUIRED.**

The stack described in §18, if approved, is approved for **Calquartz v1 only** — not a
permanent commitment beyond that scope. Approval of this stack does **not** mean
implementation is safe or complete: the stack remains subject to the Security Floor,
booking-correctness, tenant-isolation, DST, durable-job, licensing/provenance, and
verification gates named in ADR-001, none of which this document satisfies. If
implementation evidence later shows a chosen stack element cannot satisfy an approved
A/B1 defining property within the stated constraints, that stack element — or, if the
defect is architectural rather than merely a library choice, the architecture itself —
must be reopened rather than silently worked around, exactly as ADR-001 itself requires
for the architecture. This document does not, and cannot, convert its own recommendation
into a decision; only the owner can do that, explicitly, per the Durable Decision Capture
Policy.

## Adversarial Correction Pass (2026-09-08)

Scope note: this pass is a **review and correction of the sections above, not a
restart**. Nothing above this section was deleted or rewritten; §1–23 remain the
historical record of the original Phase 8A reasoning. Where this pass concludes a claim
above was overstated, that claim is corrected here, additively, with a pointer back to the
section it corrects — the original text stays legible as what was originally argued.
Re-read against ADR-001, Architecture Open Questions, Architecture Requirements and
Constraints, the Security Floor, the Multi-Tenancy Trust Model, the Failure-Mode Analysis,
the Threat Model, the Durable Decision Capture Policy, and the four spike evidence files
under `4. Calquartz Spikes/evidence/`. Conclusion up front: the original document already
practiced most of this discipline correctly (it already hedges Prisma's friction as
"widely-documented... not evidence collected by this project's own spikes," already flags
the ambiguous-local-time gap as UNKNOWN, already declines to name an auth library, already
separates "architectural fit" from "capability" for security). The corrections below are
real but narrower than a full rewrite would suggest — this is confirmation-with-tightening,
not rejection.

### (1) Corrections made

- §6/§17: the claim that Prisma's RLS/`SET LOCAL` friction is "not a rumour" is
  rhetorically stronger than the evidence supports. It is accurate that this is
  widely-documented in the general Prisma ecosystem; it is not accurate to imply that
  documentation carries the same evidentiary weight as this project's own spike results.
  Reworded below (§3 of this pass) to keep the two apart explicitly.
- §9: "the only option with direct spike evidence behind every required property" is true
  of the *protocol*, but the surrounding paragraph blurs into recommending the
  *hand-rolled implementation model* as if the same evidence covered that choice too. Spike
  4 validated a protocol; it did not evaluate whether an application-owned job table must
  permanently own that protocol's implementation versus a library implementing the same
  primitives. Corrected in §4 below.
- §10: "no separate frontend server or CDN-origin split is required" is stated as settled
  fact; it is better classified as a design choice among viable alternatives, not the only
  shape that fits A/B1. Corrected in §5 below.
- §18/§453: the recommended stack is restated here as still **RECOMMENDATION — OWNER
  APPROVAL REQUIRED**, unchanged. No element in §18 is converted to a DECISION by this
  pass.

### (2) Claims downgraded from FACT to INFERENCE/RECOMMENDATION/UNKNOWN

Using this project's own FACT/OBSERVATION/INFERENCE/RECOMMENDATION/UNKNOWN discipline
(project `CLAUDE.md` §29):

- "Prisma's transaction/connection-pooling interaction with... `SET LOCAL`... is a widely
  documented friction point" — **remains FACT** (it is a fact that this documentation
  exists), but the document's downstream *use* of that fact to downgrade Candidate 2 is
  correctly labeled **INFERENCE**, not FACT, and is now stated as such explicitly (§3
  below), not left implicit as in the original §6.
- "start from the hand-rolled protocol... because it is the only option with direct spike
  evidence" (§9) — the *protocol* claim is FACT (spike 4 evidence); the *implementation
  model* conclusion drawn from it is **RECOMMENDATION**, weakly supported, now separated
  (§4 below).
- "no separate frontend server... is required at v1 scale" (§10) — downgraded from
  implied-FACT to **RECOMMENDATION**; it is a defensible design choice, not the unique
  shape A/B1 permits.
- Everything in §14 ("Security Implications") remains correctly UNKNOWN/capability-only;
  no change needed — verified against the Security Floor document directly (§8 below).

### (3) Prisma reassessment

Correct classification, replacing the original §6/§17 framing:

- **FACT** — Phase 7 spike 1 validated RLS + transaction-local `SET LOCAL` tenant context
  surviving pool reuse under transaction-mode pooling, using the raw `pg` driver directly
  (`4. Calquartz Spikes/evidence/spike-1-tenant-isolation-evidence.md`).
- **FACT** — Phase 7 did not test Prisma, or any ORM, against this pattern. No spike
  evidence exists either way about Prisma's behavior in Calquartz's specific query shapes.
- **INFERENCE** — a thinner PostgreSQL access layer (raw `pg` + Kysely) plausibly reduces
  abstraction friction around `SET LOCAL`, exclusion-constraint DDL, and `SKIP LOCKED`
  job-table queries, because those mechanisms are expressed directly rather than through an
  ORM's own transaction/schema abstraction. This is a reasonable inference from how the
  mechanisms work, not a measured comparison.
- **RECOMMENDATION** — `pg` + Kysely remains preferred, for the reasons in §6/§15 of the
  original document, but the reasoning rests on general ecosystem friction reports plus
  inference about Calquartz's specific patterns, not on a Calquartz-tested result.
- **UNKNOWN** — whether Prisma could in fact implement Calquartz's exact RLS/`SET
  LOCAL`/exclusion-constraint/`SKIP LOCKED` patterns at an acceptable level of friction.
  Genuinely untested. The original §20 already lists a version of this as a remaining
  unknown; this pass confirms it is correctly classified and should not be read as
  resolved by the general-ecosystem evidence cited in §6.

The final recommendation (pg + Kysely over Prisma) is unchanged by this reassessment — but
the *reason* for it is now explicit: it rests on INFERENCE from mechanism design plus
general-ecosystem documentation, not on Calquartz-specific evidence. A reviewer should not
read §6 as having spike-tested Prisma's unsuitability; it did not.

### (4) Durable-job reassessment

Separating what the original §9 blurred together:

- **(A) Protocol — spike-validated, FACT.** Spike 4
  (`4. Calquartz Spikes/evidence/spike-4-durable-jobs-evidence.md`) validated: durable job
  record, atomic claim (`UPDATE ... WHERE status='pending' ... RETURNING`), lease expiry,
  fencing token, `SKIP LOCKED`, retry/dead-letter state, incremental checkpointing, and
  idempotent effect application for a scheduled cross-tenant batch job. This is a FACT
  about what was tested and found to work.
- **(B) Implementation model — application-owned job table/claim/lease/fence
  logic/worker loop.** This is a **choice**, not something spike 4 validated as uniquely
  correct. Spike 4 tested the protocol using a hand-built table because that was the
  simplest way to test the protocol in isolation — it did not compare that implementation
  path against a library implementing the same primitives.
- **(C) Alternative libraries** (`pg-boss`, `graphile-worker`, etc.) — not spike-tested,
  and this pass does **not** introduce one; per the task's explicit instruction, the
  original §9's inference is also not automatically strengthened into "bespoke is
  definitely superior to every library." Both `pg-boss` and `graphile-worker` use
  `SKIP LOCKED`-based Postgres claiming internally (consistent with A/B1's no-Redis
  constraint), but neither has been evaluated by this project against the specific
  fencing-token and incremental-checkpoint requirements spike 4 found necessary.

Evaluated against Calquartz's actual requirements: A/B1's PostgreSQL-only constraint is
satisfied by all three paths (hand-rolled, `pg-boss`, `graphile-worker`) equally — none
requires Redis. Recurring-booking occurrence-materialization needs the fencing/checkpoint
shape spike 4 validated; whether an adopted library provides equivalent guarantees is
UNKNOWN, not tested. Operational simplicity, dependency burden, testability, and failure
transparency all currently favor a hand-rolled approach only by inference (an AI-assisted
solo operator can read and reason about code it wrote directly, argued in the original
§15), not by measurement — no comparison of actual maintenance cost or defect rate between
the two paths exists. Migration/reversibility: per the original §16, moderate cost either
direction, correctly assessed and unchanged.

**Correction to the classification**: "PostgreSQL-backed durable-job semantics are
recommended" is well-supported (**RECOMMENDATION**, resting on spike 4 FACT evidence).
"Our own implementation should permanently own the entire job framework" is **not**
sufficiently justified by spike 4 alone — it is a **weaker RECOMMENDATION** resting
primarily on the inference that fidelity to the tested protocol outweighs adopting a
library, an inference the original document itself half-acknowledges in §9's own
"evaluate a library only after path 1 is running" language but does not classify
explicitly as an inference rather than a settled conclusion. This pass makes that
classification explicit: **start hand-rolled is a RECOMMENDATION with a stated re-evaluation
point, not a settled permanent architecture for job handling.**

### (5) Next.js/Fastify reassessment

Adversarial comparison, not previously made explicit in the original document:

- **(A) Next.js alone** (API routes/server actions handling all backend responsibility):
  viable for much of Calquartz's request/response surface, and would reduce the stack to
  one framework. What it handles less cleanly: a long-running worker process sharing
  application/domain code with the web process without duplicating it (Next.js is
  structured around request-scoped serverless-shaped functions, not a persistent worker
  loop with its own lifecycle), and a clean separation between "web presentation" and
  "application services" that A/B1's own modular-boundary intent (ADR-001's designed-for
  extraction seams) wants preserved for a future extraction. Not impossible, but requires
  discipline Next.js does not structurally enforce.
- **(B) Next.js frontend + Fastify backend** (the original §10/§18 recommendation): keeps
  the worker process and the web/API process sharing one clearly-bounded application-layer
  codebase (Fastify), with Next.js responsible only for UI/SSR concerns. This is a real
  separation of concerns, not a manufactured one — Fastify owns the durable-job protocol,
  the RLS/tenant-context transaction boundary, and the booking-correctness logic; Next.js
  owns rendering and calls into that layer.
- **(C) Separate frontend (React/Vite SPA) + Fastify backend**: same backend-side
  properties as (B), trading Next.js's SSR/server-component convenience for a simpler,
  fully-decoupled frontend deployable (still one Docker image at v1 scale, so this does not
  by itself cost more against A/B1's shape).

**"What problem does Fastify solve that Next.js alone would not solve adequately for
Calquartz v1?"** — primarily, giving the worker process and the web/API process one shared,
non-framework-coupled application/domain layer, and keeping the booking-correctness and
tenant-isolation code paths outside of a UI framework's request-lifecycle assumptions. This
is a real, non-manufactured justification, not an artifact of A/B1 requiring it — A/B1 does
not itself require a Next.js/Fastify split; a Next.js-only backend is architecturally
compatible with A/B1's process-count shape too.

**"What additional complexity does the combination create?"** — two frameworks, two sets of
conventions and dependency graphs, an internal API boundary between Next.js and Fastify
that must itself be kept simple (same-VPS, likely same Docker image, so this is closer to
"two processes talking over localhost" than "a distributed system," but it is still a
boundary to design and test). For a solo operator, this is a real, non-trivial cost.

**Conclusion**: Next.js+Fastify remains the strongest candidate evaluated, but the original
document did not make this trade-off explicit — it asserted the combination fit A/B1
without asking whether Next.js alone would have sufficed. Corrected classification:
**RECOMMENDATION**, not settled fact, resting on the worker-process/shared-domain-layer
argument above, which is a real fit-for-purpose reason, not a reflexive default.

### (6) Luxon/timezone reassessment

Preserved and re-verified against `4. Calquartz Spikes/evidence/spike-3-dst-evidence.md`:

- **FACT** — the tested day-cursor construction is incorrect across DST transitions.
- **FACT** — direct wall-clock construction is required and was demonstrated correct.
- **FACT** — Luxon was the library used in the spike for both the failing and correct
  constructions.
- **RECOMMENDATION** — Luxon is the preferred library specifically because it is already
  exercised, in this project's own evidence, against the relevant patterns — not because it
  is superior in the abstract. The original §8 already states this correctly ("not a claim
  that Luxon is superior in the abstract"); this pass confirms that framing is accurate and
  should not be read more strongly than written.
- **UNKNOWN** — final product semantics for nonexistent/ambiguous local times, and
  recurring-series behavior across a full DST transition history spanning many occurrences
  (spike 3 tested individual construction correctness, not a full recurring series'
  behavior end-to-end). The original §8/§20 already names the ambiguous/nonexistent-time
  gap; this pass adds that the recurring-series-across-many-transitions case is a distinct,
  also-unresolved UNKNOWN not previously named separately.

Checked for reliance on "SnagTime used Luxon" as justification: **not found**. The original
document's Luxon recommendation rests entirely on spike 3's own evidence, not on precedent
from either reference system. No correction needed on this specific point — the original
text passes this check cleanly.

### (7) Authentication boundary

Confirmed and made explicit (original §11 already substantially correct):

- Auth **mechanism** (session-cookie shape) is addressed here as an implementation
  approach, consistent with the charter's identity/membership/authorization separation
  (project `CLAUDE.md` §14–15).
- Auth **library selection** is deferred, explicitly, and this is intentional — not an
  oversight. This does not mean auth architecture is unaddressed: the
  identity/membership/authorization separation is itself an architectural commitment, made
  here; only the specific library is deferred.
- **Where the deferred piece belongs**: concrete auth-library selection is better scoped to
  a future Phase 8B (implementation planning) than decided inside this stack-selection
  document, because library selection requires evaluating Security Floor compliance
  (session security, CSRF/XSS mitigation specifics) against a specific library's actual
  API — work this document correctly declines to do prematurely. This pass makes that
  scoping explicit; the original §11 gestured at "evaluated at implementation time" without
  naming which phase that implementation-time evaluation belongs to.

### (8) Security Floor status

Verified against [[../architecture/Security Floor|Security Floor]] and ADR-001's
"Implementation gates" section: **no claim anywhere in this document, before or after this
pass, states the Security Floor has passed.** All sixteen items remain `UNKNOWN`, correctly,
because no Calquartz implementation exists. §14 of the original document states this
explicitly and correctly ("No claim that the Security Floor has passed — no Calquartz code
exists"). No correction needed.

Checked for an obvious structural security problem in the recommendation: none found beyond
what is already named. The session-cookie/`users`+`memberships` split (§11), worker-role
privilege separation via a narrower database role (§14, spike-1-tested), parameterized
queries via Kysely/raw `pg` (§6–7), and CSRF/rate-limiting/logging/security-header
capability (§14) are all structurally available in the recommended stack without
obstruction. This is capability, not verification — consistent with the original document's
own framing, unchanged by this pass.

### (9) Recurring-booking sanity check

Verified: the document does not claim recurring bookings are "solved." §19 ("Explicit
Non-Decisions") explicitly leaves the recurring-series atomicity model open; §9 states
spike 4 validated a protocol for "the recurring-occurrence-materialization batch shape,"
not the full recurring-booking subsystem. Cross-checked against the full required chain
(series definition → occurrence generation → wall-clock/DST semantics → conflict detection
→ transaction behavior → partial-failure semantics → cancellation → rescheduling →
calendar sync → notifications → retries → reconciliation): spikes 2, 3, and 4 validated
specific mechanisms within this chain (exclusion-constraint conflict detection, DST-correct
occurrence construction, durable-job claim/lease/fence for batch materialization) — none of
the three, individually or together, validated the full chain. The original document does
not claim otherwise. No correction required; this pass records the check as performed and
passed.

### (10) Deployment/capacity distinction

Verified against [[../architecture/Calquartz — VPS Infrastructure Inventory|VPS
Infrastructure Inventory]] and [[Phase 5B — Factual and Technical Prerequisites|Phase 5B]]:
the original §13/§20 already distinguishes architectural fit from capacity proof — §20
states explicitly "no load testing has occurred; Phase 5B's headroom measurement is
idle-state only." This is the correct distinction and is preserved unchanged. **Confirmed:
the document at no point claims the VPS "has sufficient capacity" as a fact** — headroom is
described only as idle-state measurement, correctly scoped. No correction required.

### (11) Candidate-set assessment

The original candidate set (§5: TS/Fastify+Kysely, TS/Fastify+Prisma, Python/FastAPI+
SQLAlchemy, with Rails/Django, Go, and serverless/edge rejected before full evaluation) is
adequate. Checked whether a materially better candidate was omitted: no realistic candidate
was found that would plausibly change the recommendation — a Deno/Bun runtime variant would
not change the data-access-layer reasoning that drives the recommendation; a GraphQL API
paradigm is an orthogonal choice already correctly left open (§19); a `pg-boss`/
`graphile-worker`-first job architecture is already named and evaluated (§9, and
reassessed in §4 of this pass) without changing the recommendation. No addition made — the
original candidate set is not padded, and this pass does not pad it further.

### (12) Reversibility

Re-verified against the original §16, cross-checked against actual coupling in the
recommended stack; no material correction needed. Restated for completeness with the API
boundary explicitly separated from backend language, since the original §16 did not list it
separately:

- **Backend language/runtime**: expensive — effectively a rewrite.
- **Database**: expensive — PostgreSQL is authoritative and A/B1's defining property; not
  a stack-selection choice at all (ADR-001, not this document, fixes it).
- **Data-access layer (Kysely vs. raw SQL vs. Prisma)**: cheap — thin layer, call-site-only
  migration cost.
- **API boundary (Next.js/Fastify split vs. Next.js-only)**: moderate — separable now while
  no code exists; expensive to restructure after significant application-layer code has
  accumulated inside either framework's own request-lifecycle assumptions.
- **Date/time representation**: moderate, if isolated to a date-utilities module as the
  original §21 already states as a precondition.
- **Authentication model** (mechanism, not library): moderate — the identity/membership/
  authorization separation is architectural and expensive to retrofit if violated early;
  the specific library is cheap to swap behind that boundary.
- **Job model (hand-rolled vs. library)**: moderate — job-table schema is the expensive
  part; claim-loop code is more replaceable, matching the original §16.
- **Frontend framework**: moderate-to-high, decoupled from backend correctness, matching
  the original §16.

Reversibility being moderate-to-cheap for most elements does not, and should not, excuse
weak justification — each choice above is still argued on its own merits in §3–7 of this
pass and §6–15 of the original document, not waved through because it could later be
undone.

### (13) Strongest argument against the current stack

Combining the original §21 with this pass's findings: the recommendation optimizes for
fidelity to already-validated spike patterns (raw SQL, hand-rolled job protocol) at a
development-velocity cost that has not been measured, and it assumes — without
project-specific testing — that Prisma's general-ecosystem friction would manifest in
Calquartz's specific query patterns and that a hand-rolled job implementation is worth its
added maintenance surface over an adopted library. Both assumptions are individually
reasonable inferences but stack on top of each other, compounding an unmeasured velocity
cost. A reviewer could reasonably conclude the recommendation under-weights the possibility
that Prisma or a job library would have been acceptable with modest workarounds.

### (14) Weakest-handled requirement

"Maintainability for a solo AI-assisted operator," specifically for the durable-job
implementation model (§4 of this pass): the hand-rolled protocol is well-evidenced, but the
decision to have the application permanently own the full job-framework implementation,
rather than adopting a library implementing the same primitives, rests on inference, not
comparison. This is the same conclusion the original §21 reached for a related but
narrower question ("a hand-rolled implementation is more code to keep correct over time");
this pass sharpens it into the (A)/(B)/(C) split in §4.

### (15) Biggest long-term liability

The hand-rolled job table's implementation model (§4/§14 of this pass): if the recurring-
occurrence-materialization batch logic or the checkpoint discipline degrades over time
(e.g. checkpointing regresses from incremental to batch-completion-only, a failure mode
spike 4's own evidence names as "critically dependent" on getting right — original §21),
there is no library-level guardrail catching the regression; it is entirely the
solo operator's (and their AI assistant's) discipline to maintain. This is the same
liability the original §21 names for the date-library gap and the job-guarantee gap; this
pass confirms it as the single largest liability across the whole recommendation, not one
among several equally-weighted risks.

### (16) Biggest evidence gap

Whether Prisma's general-ecosystem RLS/`SET LOCAL` friction would actually manifest as a
correctness problem in Calquartz's specific query patterns (§3 of this pass; also named in
the original §20). This is the claim most dependent on inference rather than direct
Calquartz-specific evidence — the entire Candidate-2 downgrade in §6/§17 rests on it, and no
spike tested it directly.

### (17) Reopen triggers

Concrete implementation-time evidence that would require revisiting this stack
recommendation (not the architecture — ADR-001's own reopen triggers are separate and
unaffected):

- Kysely, or raw `pg`, is found unable to cleanly express a required PostgreSQL feature
  discovered during real schema design (already named, original §21).
- Measured AI-assisted development velocity on the hand-rolled job table is materially
  worse than an adopted library would plausibly have been, once implementation is underway
  and can be compared against real effort, not estimated in advance.
- A Calquartz-specific test (not general-ecosystem documentation) of Prisma against the
  RLS/`SET LOCAL` pattern, if ever run, either confirms or refutes the friction this
  document currently only infers — either result should update §6/§17's classification.
- The nonexistent/ambiguous-local-time product policy, once decided, reveals that Luxon
  cannot cleanly express the chosen policy (unlikely given Luxon's flexibility, but
  untested).
- Any of ADR-001's own four architecture-level reopen triggers firing during real
  implementation — this stack selection cannot prevent or cause them, but would need
  re-evaluation if the architecture itself reopened.

### (18) Final recommendation

**RECOMMENDATION — OWNER APPROVAL REQUIRED, NOT A DECISION.** This pass concludes the
evidence supports the stack recommendation in §18 of the original document, with the
reasoning corrections above (Prisma reassessment §3, durable-job implementation-model
narrowing §4, Next.js/Fastify justification made explicit §5). The recommendation itself is
**unchanged** — no stack element is swapped, added, or removed by this pass. What changes is
classification precision: the durable-job **implementation model** (application permanently
owning the full job framework, as opposed to PostgreSQL-backed durable-job semantics in
general) is now explicitly marked as a weaker RECOMMENDATION with a stated re-evaluation
point (original §9's "evaluate a library after path 1 is running," now classified
explicitly as such rather than left as an implicit qualifier), and the Prisma downgrade's
evidentiary basis (general-ecosystem inference, not Calquartz-specific evidence) is now
explicit rather than implied. No element of §18 is, or is treated as, an owner-approved
DECISION. ADR-001 remains the only actual architectural DECISION in this project.

## Owner Approval (2026-09-08, Phase 8B)

Recorded per the [[Durable Decision Capture Policy]]. **The owner has explicitly approved
the stack recommendation in §18 (as corrected by the Adversarial Correction Pass above,
which changed no stack element) as a DECISION**, in chat, during Phase 8B. This approval
is scoped **exactly as stated in the task instruction that recorded it, and no wider**:

- **Approved as DECISION**: TypeScript/Node.js; Fastify; `pg` + Kysely; `node-pg-migrate`;
  Luxon; PostgreSQL-backed durable jobs using the Phase 7-validated protocol; Next.js/React;
  a session-cookie auth *architecture* (concrete library still deferred); Vitest + real
  PostgreSQL integration tests + Playwright; single Docker image, web+worker processes,
  PostgreSQL, no Redis, no Kubernetes. This is the §18 stack-selection list, and only that
  list.
- **This is not owner approval of any downstream implementation detail.** Every item §19
  ("Explicit Non-Decisions") and §20 ("Remaining Unknowns") already named as open **remains
  explicitly UNKNOWN**, unchanged by this approval: the concrete auth library; the
  nonexistent/ambiguous local-time policy; the recurring-series atomicity model; the exact
  tenant-isolation implementation; the exact exclusion-constraint implementation; the exact
  durable-job implementation; whether a job library could later replace the app-owned
  protocol; real VPS capacity/performance under load; seats/group bookings; round-robin;
  the calendar provider; the notification provider; a public developer API; embeds; the
  canonical product name. **None of these is silently upgraded to DECISION by this note.**
  Per the Durable Decision Capture Policy §9–10, only an explicit, separately-obtained
  owner statement resolves each of them, individually, when it happens.
- **ADR-001 is unaffected and remains unchanged.** ADR-001 remains the sole architectural
  DECISION of record; this note approves an implementation stack *for* that already-approved
  architecture, not a new or revised architecture.
- **Authoritative home of this decision**: this note, in this document (the canonical home
  for the stack-selection decision, per the "one fact, one home" rule) — no duplicate copy
  of the approval text is written elsewhere; other documents that reference the stack link
  back here.
- **Propagation**: no other Second Brain document's substantive content assumed the stack
  was still only a RECOMMENDATION in a way that requires correction — [[Architecture Open
  Questions]] already explicitly declined to track the Phase 8A UNKNOWNs (see its own
  Phase 8A note), so a short cross-reference is added there rather than any content change;
  no other document is affected.

## Related

- [[../decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]] — the architecture this stack selection implements
- [[../Calquartz — Pre-Approval Technical Spikes|Pre-Approval Technical Spikes]] — the empirical evidence base for §6–9
- [[../questions/Architecture Open Questions|Architecture Open Questions]] — updated in this same pass to cross-reference this document
- [[../architecture/Architecture Requirements and Constraints|Architecture Requirements and Constraints]] — updated additively in this same pass
- [[Durable Decision Capture Policy|Durable Decision Capture Policy]] — governs how any future owner approval of this recommendation must be recorded
