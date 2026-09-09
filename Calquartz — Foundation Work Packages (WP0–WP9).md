---
type: synthesis
created: 2026-09-09
updated: 2026-09-09 (WP0 scope text corrected same day, per the Foundation Implementation
  Plan's Post-Audit Correction Pass, following the WP0 Pre-Authorization Evidence Audit)
sources: ["wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation Implementation Plan.md", "wiki/projects/001-calendar-os/decisions/ADR-001 — A-B1 Modular Monolith Architecture.md", "wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack Selection.md", "wiki/projects/001-calendar-os/architecture/Security Floor.md", "wiki/projects/001-calendar-os/engineering/Authority, Risk, and Routing Model.md", "wiki/projects/001-calendar-os/engineering/Verification Evidence Format.md", "wiki/projects/001-calendar-os/engineering/Calquartz Booking Correctness — Operational Review Process.md", "wiki/projects/001-calendar-os/Calquartz — Pre-Approval Technical Spikes.md", "wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation Implementation Plan — Adversarial Review.md"]
tags: [engineering, planning, foundation, work-packages, calendar-os, calquartz]
confidence: low-medium — see Provenance section
provenance: RECONSTRUCTION, not a recovered transcript — see "Provenance" section below for exactly what this is and is not
classification: mixed — every work package below carries its own FACT/RECOMMENDATION/gate/UNKNOWN labels; this document as a whole is RECOMMENDATION, not DECISION
scope: calquartz-specific
status: DRAFT — companion to the Foundation Implementation Plan's Adversarial Correction Pass (2026-09-09); no application code, repository, or migration created by this document
---

# Calquartz — Foundation Work Packages (WP0–WP9)

## Provenance — read this before treating anything below as historical record

**Correction note (2026-09-09, same day)**: WP0's scope text below was corrected
after the [[Calquartz — WP0 Pre-Authorization Evidence Audit|WP0 Pre-Authorization
Evidence Audit]] found several of the Foundation Implementation Plan's own recommended
corrections existed only in that plan's narrative, not yet reflected in this file's
prescriptive WP0 text. See the plan's own "Post-Audit Correction Pass — 2026-09-09"
section for the full record of what changed and why. This provenance section's own
account of the file's origin (below) is otherwise unchanged.

**What this is not**: a verbatim transcript of the live `planner`-agent invocation that
originally produced a WP0–WP9 breakdown during this session's Foundation Implementation
Plan pass (2026-09-09). That transcript is **not durably stored anywhere in this
vault** — checked directly in this pass: no transcript file exists under
`wiki/projects/001-calendar-os/` or `raw/projects/001-calendar-os/`. The
[[Calquartz — Application Foundation Implementation Plan|Foundation Implementation
Plan]] itself already states this gap honestly in its §6 ("Limitations of this
synthesis"), and its own Adversarial Review (Review 11) independently confirmed the
practical consequence: no reviewer, then or since, could verify or attack the actual
per-work-package risk tier, routing chain, or human-gate detail, because it existed only
in ephemeral session context.

**What this is**: a reconstruction, assembled from (a) what the Foundation
Implementation Plan's §2–§5 already states about the proposed structure and the five
named blockers, (b) the two fragments of `planner`'s original WP numbering that survive
in the plan's own text (§4 states that the worker-credential-narrower-than-web
requirement and the catalog-driven fail-closed enrolment check were both "already
present in spike 1's own evidence and in `planner`'s WP4/WP5 descriptions" — the only
concrete WP-number-to-content mapping that exists anywhere in the vault), and (c) this
pass's own re-derivation of a plausible, evidence-grounded work-package sequence from
ADR-001, Phase 8A, the Security Floor, the Booking Correctness process, and the
Authority/Risk/Routing Model, applying the Adversarial Correction Pass's corrections
(particularly 1, 6, 7, and 16) directly to the sequencing and scope of each package.

**What this means practically**: WP4 and WP5's subject matter (tenant/membership schema
with enrolment check; worker-role/credential separation and job-table confirmation) is
the one part of this document with a direct textual anchor in a prior specialist output.
Every other package's exact number, name, and boundary is this pass's own synthesis, not
`planner`'s original wording — **do not cite this document as evidence of what
`planner` actually said** about any work package other than WP4/WP5's general subject
matter. It is offered as a durable, evidence-grounded planning artifact in its own right,
correctly labeled as such.

**Classification key used throughout**: **FACT** (established directly by ADR-001,
Phase 8A, a spike, or the Security Floor) / **RECOMMENDATION** (this pass's own proposed
scope or sequencing, not binding) / **gate** (requires owner sign-off per the
Authority/Risk/Routing Model before the package's gated portion may proceed) /
**UNKNOWN** (genuinely unresolved, not decided by this document).

---

## Risk-tier and human-gate methodology

Each package below is classified using the two-tier LOW/HIGH-CRITICAL scheme and the
nine named risk domains (plus the tenth catch-all) from
[[Authority, Risk, and Routing Model]] §2.1: tenant isolation, auth/authz, booking
correctness, timezone/DST, database schema/migrations, secrets/credentials, deployment,
destructive operations, security controls generally, and "anything capable of corrupting
durable state." A package touching any of these — or genuinely ambiguous — is
HIGH/CRITICAL; everything else is LOW, by exclusion, not by a separate checklist (per
§2.2 of that document). The routing chain named for each HIGH/CRITICAL package follows
§3.2's table for the matching task/routing class.

---

## WP0 — Repository genesis and local scaffolding

**Risk tier**: Mixed — the package as a whole is not one task; see split below.

**Scope**: local git repository initialization; pnpm workspace layout (`apps/`,
`modules/` empty, `packages/`, `migrations/`, `test/`, `docker/` per the plan's §2);
boundary-lint (dependency-cruiser) configuration; `packages/kernel` and `packages/obs`
as local files, not packages, per Adversarial Correction Pass correction 7;
`packages/config` as a documented per-entrypoint convention, not a standing package, per
the same correction; `packages/db`'s pool + tenant-context primitive under
mechanism-neutral naming (`withTenantContext`/`withElevatedAccess`, per correction 5) —
**WP0 must NOT implement `SET LOCAL`, RLS-specific policies/roles, `BYPASSRLS`, or any
other mechanism-specific tenant-isolation implementation**; RLS remains a POSSIBLE,
spike-1-validated candidate, not a decision; `packages/jobs`'s protocol scaffolding
(claim/lease/fence/`SKIP LOCKED`/checkpoint only, no domain handlers) — the protocol is
spike-4-validated FACT, but whether the application permanently owns a hand-rolled job
framework versus later adopting a library is **NOT decided** (Phase 8A's own Correction
Pass, caveat carried forward per correction 7b); `packages/time` as a local module with
explicit `{ok, value} | {ok: false, reason}` result types, per correction 7 and
correction 6's gate; `packages/contracts` as a thin, types-only package containing
**domain-shaped, transport-neutral types only** — it must NOT encode REST, tRPC,
GraphQL, HTTP-status, router, or other transport-specific assumptions (kept as a
package, per Phase 8A §18's approved type-sharing rationale — correction 7a; the API
paradigm itself remains UNKNOWN, Phase 8A §19, and this scope note is a constraint on
the package's shape, not a paradigm selection). The integration-test harness's
*existence* (real-Postgres, template-DB-clone-per-file, per Phase 8A §12) is
foundation-appropriate; its exact implementation shape is **UNKNOWN until harness code
exists and is checked** — this document does not claim its mechanism-neutrality has
already been proven, and it must not silently introduce RLS-specific test mechanics.
**Excluded from WP0's scope**: Playwright E2E scaffolding — deferred to the work
package that delivers the first real page (no page exists yet to exercise it); the
eventual Playwright requirement itself (Phase 8A §18 DECISION) is not removed from the
broader plan, only relocated out of WP0.

**Human gate**: **No**, for the local-only-scaffolding portion listed above — this is
LOW/reversible work not touching a HIGH domain. **Yes**, for any sub-step that would
create tenant-scoped schema (routes to WP3/WP4 instead, gated) or construct/interpret
local wall-clock booking times (routes to a future booking-domain package, gated per
correction 6).

**Routing chain (LOW baseline, per §3.1)**: implement → self-check against ADR-001/
Phase 8A/Security Floor → verification-loop (build/type/lint where applicable) →
knowledge capture if durable.

**Blockers**: none, per Adversarial Correction Pass correction 1 — none of the five
originally-named blockers gates this package's local-only scope.

**Verification required**: boundary-lint fires on a deliberate violation (named proof,
FACT-required per the original plan §4); no Verification Evidence Record required for
purely LOW sub-steps, but a record should be produced for the package as a whole given
its adjacency to several HIGH-risk-adjacent domains (secrets/config, deployment shape),
per Adversarial Correction Pass correction 11.

---

## WP1 — Repository hosting and CI activation

**Risk tier**: HIGH/CRITICAL — domain 7 (deployment-adjacent: where code lives and what
runs against it) and domain 6 (secrets, once CI secrets are configured).

**Scope**: choosing where the repository is hosted (private vs. public), which CI
provider runs it, where CI secrets live; pushing WP0's local scaffolding to that remote;
activating CI (build/type/lint/test on push).

**Human gate**: **Yes, always** — per §3.2's "Deployment" row and blocker 3's own scope
(Adversarial Correction Pass correction 1). This is an owner decision; Claude does not
select a hosting/CI provider unilaterally.

**Blockers**: blocker 3 (repository hosting/CI provider), scoped exactly to this package
— does not block WP0's local-only work, per correction 1.

**Verification required**: CI actually runs the verification-loop on push (a positive
proof, not merely configured-and-assumed-working); no secret is committed to the
repository history at any point (Security Floor item 7).

---

## WP2 — Migrations tooling and non-tenant-scoped schema baseline

**Risk tier**: HIGH/CRITICAL — domain 5 (database schema/migrations), even though the
content of this package is deliberately non-tenant-scoped.

**Scope**: `node-pg-migrate` (Phase 8A §18, FACT/DECISION) wired into the repository;
migration-runner conventions (explicit deploy-time step, not implicit at boot, per
Phase 8A §13); the durable-job table migration, confirmed (not designed fresh) against
spike 4's already-specified shape, per Adversarial Correction Pass correction 2; any
genuinely non-tenant-scoped baseline tables (e.g. a schema-version/migration-metadata
table, if the chosen tooling needs one beyond its own internal bookkeeping).

**Human gate**: **Yes, for the job-table migration specifically** — routes through the
Authority/Risk/Routing Model's "Database schema/data migration" row as a
**confirm-not-decide** step (correction 2), still requiring owner sign-off before the
migration is applied, per that row's own "touches a shape a prior document assumed"
principle. **No**, for migration-tooling wiring itself (LOW, no schema content yet).

**Blockers**: none for tooling wiring; the job-table migration specifically requires
owner confirmation (not a full design decision) per correction 2.

**Verification required**: migration up/down/up reversibility (named proof, though
Adversarial Review's Review 7 correctly notes this is of most value once a real
data-shape risk exists — still worth exercising once against the job table); the job
protocol reproducing spike 4's result inside real application code, including the
incremental-checkpoint case specifically (Phase 8A §22 explicit requirement).

---

## WP3 — Tenant-isolation mechanism scaffolding (mechanism-agnostic)

**Risk tier**: HIGH/CRITICAL — domain 1 (tenant isolation), the project's single most
scrutinized domain.

**Scope**: the fail-closed, catalog-driven (schema-derived at build time) enrolment
check that spike 1 validated as a requirement — the mechanism this package builds must
work regardless of which database-level backstop is eventually selected; the
tenant-context test harness (real-Postgres, template-DB-clone-per-file per the plan's
§2) built and verified as testing tenant-isolation *behavior*, not a specific
mechanism's internals, per Adversarial Correction Pass correction 5's requirement that
this either be confirmed mechanism-neutral or corrected before being treated as settled.
**Does not include**: selecting RLS or an alternative — that selection is explicitly out
of this package's scope.

**Human gate**: **No**, for the scaffolding/harness work itself, provided it remains
demonstrably mechanism-neutral (this is the point of building it now, per correction
1/16). **Yes**, if at any point building the harness requires committing to a specific
mechanism's internals to proceed — at that point the work has crossed into WP4's
gated territory and should stop and escalate.

**Blockers**: none for mechanism-neutral scaffolding; the mechanism *selection* itself
(blocker 1) is not this package's job.

**Verification required**: fail-closed enrolment fires on a deliberately unenrolled
table (named proof, FACT-required, spike 1's own success criterion 2); tenant-context
leakage proof — a query issued without the seam must fail, not silently succeed
(Adversarial Correction Pass correction 11's highest-priority missing proof).

---

## WP4 — Tenant/membership schema and enrolment (per `planner`'s original WP4, per §4's fragment)

**Risk tier**: HIGH/CRITICAL — domains 1 (tenant isolation) and 5 (database
schema/migrations), simultaneously.

**Scope**: the actual tenant-scoped schema (tenants/workspaces, per the flat-tenancy
DECISION already recorded in [[../architecture/Architecture Requirements and
Constraints|Architecture Requirements and Constraints]] §1 item 8; users/memberships,
kept schematically and conceptually separate per the charter §14–15 and Phase 8A §11);
wiring the fail-closed enrolment check (WP3) against this first real tenant-scoped
table set, proving it fires correctly on real schema, not only a synthetic test table.

**Human gate**: **Yes, always** — this is the first work package that actually requires
the tenant-isolation mechanism (blocker 1) to be resolved, per Adversarial Correction
Pass correction 1's scoping. **This package cannot begin until the owner selects the
database-level tenant-isolation backstop** (RLS or a validated alternative).

**Blockers**: blocker 1 (tenant-isolation mechanism), unscoped exception — this is
exactly the package that blocker genuinely gates, per correction 1.

**Verification required**: fail-closed enrolment fires on the real schema; tenant-context
leakage proof against real tenant-scoped tables; attacker-controlled cross-tenant access
proof (not merely schema-enrolment — Reuse Audit §36's distinction, named in Adversarial
Correction Pass correction 11); connection-pool contamination proof (spike 1's own core
finding, reproduced against real application code).

---

## WP5 — Worker-role separation and job-table schema confirmation (per `planner`'s original WP5, per §4's fragment)

**Risk tier**: HIGH/CRITICAL — domains 1 (tenant isolation, worker-role angle), 5
(database schema, job table specifically), and 6 (secrets/credentials, the worker's own
database credential).

**Scope**: provisioning the worker's database role with credentials narrower than the
web application's, per spike 1 criterion 3 (`architect`'s flag, not previously
structurally homed anywhere in the original plan); confirming the job table's schema
(from WP2) is actually consumed correctly by a real worker process under the narrower
role; the worker-role narrowness proof itself (spike 1 criterion 3, directly
reproducible).

**Human gate**: **Yes** — credential provisioning is domain 6 (secrets/credentials),
which gates unconditionally per §3.2's "Secrets/credentials" row ("Yes, always").

**Blockers**: none beyond the standing secrets/credentials gate; does not require the
tenant-isolation mechanism to be finalized, since worker-role narrowness is validated
against spike 1's already-established criterion regardless of which specific backstop is
eventually selected (a narrower role is compatible with RLS or an application-layer
alternative).

**Verification required**: worker-role narrowness proof (spike 1 criterion 3); secret
handling proof (Security Floor item 7 — no credential committed, logged, or exposed).

---

## WP6 — Booking-domain skeleton (event types, availability) — local-time-policy-gated

**Risk tier**: HIGH/CRITICAL — domains 3 (booking correctness) and 4 (timezone/DST),
simultaneously, once wall-clock construction begins.

**Scope**: event-type and availability schema/module skeleton; the first code path that
constructs or interprets local wall-clock booking times, using the wall-clock-correct
construction pattern spike 3 validated (direct construction, not a day-cursor-plus-
minutes shape — spike 3's own explicit prohibition, cited in Adversarial Correction Pass
correction 6).

**Human gate**: **Yes**, specifically for the local-time-construction portion — per
Adversarial Correction Pass correction 6, this package cannot proceed past
wall-clock-construction work until the ambiguous/nonexistent-local-time product policy
is resolved by the owner, **or** the owner explicitly accepts the stated candidate
default (reject nonexistent local start times; require explicit UTC-offset
disambiguation for ambiguous ones — Phase 8A §8's own named plausible shape). Schema
skeleton work that does not yet construct wall-clock times may proceed without this gate.

**Blockers**: the local-time policy (newly named as a gate in Adversarial Correction
Pass correction 6 — was previously missing from the plan's blocker list entirely).

**Verification required**: DST forward/backward transition proof (Phase 8A §22
explicit requirement, previously missing from the plan — added in Adversarial Correction
Pass correction 11); nonexistent local time proof; ambiguous local time proof; recurring
occurrences across a full transition history (named as a distinct UNKNOWN by Phase 8A's
own Adversarial Correction Pass §6, not the same as the single-construction case).

---

## WP7 — Authentication mechanism scaffolding

**Risk tier**: HIGH/CRITICAL — domain 2 (authentication/authorization).

**Scope**: the session-cookie auth *architecture* (Phase 8A §18, DECISION scope) —
`users`/`memberships` kept schematically separate (already built in WP4); CSRF-token
middleware capability; role-check scaffolding. **Does not include**: selecting a
concrete auth library — Phase 8A §19/§20 and the Phase 8B Owner Approval both explicitly
leave this UNKNOWN, deferred to a future implementation-planning phase, not this
foundation pass.

**Human gate**: **Yes**, before merge/deploy, per §3.2's "New feature,
tenant/auth/security-sensitive" row — routes through the Security Floor — Operational
Review Process and `security-reviewer`.

**Blockers**: none beyond the standing security-sensitive-feature gate; does not require
the auth library to be selected to build the architecture-level scaffolding (identity/
membership/authorization separation), consistent with Phase 8A §11.

**Verification required**: session-cookie mechanics (HTTP-only, secure) capability
check; CSRF-token capability check; auth/authz-boundary proof once real endpoints exist.

---

## WP8 — Observability, security headers, and boot-time configuration gate

**Risk tier**: HIGH/CRITICAL — domain 9 (security controls generally: headers, boot
validation) and domain 6 (secrets, via the boot gate's own subject matter).

**Scope**: the allowlist-by-construction logger (as a local file per correction 7, not a
standing package, until a second consumer justifies extraction); browser-side security
headers (CSP, `frame-ancestors`/`X-Frame-Options`, HSTS, `X-Content-Type-Options` —
Security Floor item 14, re-derived rather than copied from Cal.diy's `csp.ts`, per
Adversarial Correction Pass correction 3's reuse-gate framing); production-configuration
validation at boot (Security Floor item 15 — refuses to start on missing/placeholder/
insufficiently-random secrets, an unverified-TLS database URL, or active demo/local
provider fallbacks; re-derived rather than copied from SnagTime's
`assertProductionRuntimeSecurity`, same reuse-gate framing).

**Human gate**: **Yes**, for the boot-gate's exact refusal conditions (security-sensitive,
routes through the Security Floor process) — **No**, for the logger scaffolding itself
(LOW, no domain triggered).

**Blockers**: none — licensing/provenance is closed by the plan's own accepted default
(re-derive, copy nothing — Adversarial Correction Pass correction 3), so neither named
upstream pattern needs clearance to proceed with a re-derived version.

**Verification required**: the boot gate refuses three specific bad configurations
(named proof from the original plan, **currently unspecified which three** — this
package must name them: e.g. a missing signing secret, an insufficiently-random
encryption key, a database URL without verified TLS — per the Security Floor item 15's
own evidence base); secret handling proof (no secret in logs/source/debug output).

---

## WP9 — Foundation verification harness and evidence-record wiring

**Risk tier**: LOW for the harness-wiring work itself; the *content* it verifies is
whatever risk tier the underlying package carries.

**Scope**: wiring the integration-test harness (real-Postgres, template-clone, per
Phase 8A §12) into CI (once WP1 exists); assembling the full negative-proof set named in
Adversarial Correction Pass correction 11 into actual runnable checks; producing the
first real [[Verification Evidence Format|Verification Evidence Record]] for the
foundation work as a whole, closing the gap the original plan left open (no record was
proposed for WP0–WP8's own work despite touching several HIGH-risk-adjacent domains).

**Human gate**: **No**, for harness wiring itself. Individual records this package
produces inherit whichever package's own gate status applies to what they verify.

**Blockers**: depends on WP1 (CI) existing for the CI-wired portion; local-only
verification (running the harness by hand) has no blocker.

**Verification required**: this package **is** the verification layer — its own
"verification" is that every named proof from correction 11 actually exists as a runnable
check and produces a real, non-illustrative Verification Evidence Record, not an
"agent reviewed it" claim.

---

## Summary table

| WP | Subject | Risk tier | Human gate | Blocked by |
|---|---|---|---|---|
| 0 | Repo genesis, local scaffolding (kernel/obs/config/db/jobs/time as files or mechanism-neutral packages) | Mixed, mostly LOW | No (local-only portion) | None |
| 1 | Repository hosting + CI activation | HIGH | Yes, always | Blocker 3 |
| 2 | Migrations tooling + job-table migration | HIGH (job-table migration only) | Yes (job-table migration, confirm-not-decide) | None (tooling); spike-4 confirmation (job table) |
| 3 | Tenant-isolation scaffolding (mechanism-agnostic) | HIGH | No, if genuinely mechanism-neutral | None |
| 4 | Tenant/membership schema + enrolment | HIGH | Yes, always | Blocker 1 |
| 5 | Worker-role separation + job-table confirmation | HIGH | Yes (secrets) | None (beyond standing gate) |
| 6 | Booking-domain skeleton (local-time-gated) | HIGH | Yes (local-time construction only) | Local-time policy (new gate, correction 6) |
| 7 | Auth mechanism scaffolding | HIGH | Yes, before merge/deploy | None (library selection deferred separately) |
| 8 | Observability, headers, boot gate | HIGH | Yes (boot-gate specifics) | None (licensing closed by default) |
| 9 | Verification harness + evidence records | LOW (wiring) | No | WP1 (CI-wired portion only) |

---

## What this document does not do

It does not select the tenant-isolation mechanism, the auth library, the recurring-series
atomicity model, the local-time product policy, the calendar/notification provider, or
the canonical product name — every UNKNOWN named in ADR-001, Phase 8A, and the
Adversarial Correction Pass remains exactly as open after this document as before it. It
does not authorize any work package to begin — authorization is governed by the
Foundation Implementation Plan's own Adversarial Correction Pass (correction 16) and by
each package's stated human-gate status above. It does not create any repository,
migration, or application code.

## Related

- [[Calquartz — Application Foundation Implementation Plan]] — the plan this file is a
  companion to, per its Adversarial Correction Pass correction 12
- [[Calquartz — Application Foundation Implementation Plan — Adversarial Review]]
- [[../decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]]
- [[Phase 8A — Implementation Stack Selection]]
- [[Authority, Risk, and Routing Model]] · [[Verification Evidence Format]]
- [[../architecture/Security Floor|Security Floor]]
- [[Calquartz Booking Correctness — Operational Review Process]]
