# Calquartz — Final Adversarial Audit Handoff

**Purpose of this file.** This is a self-contained handoff for an independent reviewer (a
different AI, referred to throughout as "ChatGPT") who has not seen the session that
produced it. It records the outcome of a **final, independent, adversarial
pre-authorization review** of the Calquartz foundation planning artifacts — the
`Calquartz — Application Foundation Implementation Plan.md`, its companion
`Calquartz — Foundation Work Packages (WP0–WP9).md`, and the two prior review passes over
them (an "Adversarial Review" and a "WP0 Pre-Authorization Evidence Audit"), plus a third
prior pass called the **"Final Pre-Authorization Audit"** which concluded 0 Critical / 0
High / 2 Medium / 2 Low findings and verdict **READY WITH REQUIRED CORRECTIONS**. This
handoff's audit treats that prior verdict as **unverified**, re-derives its own judgment
directly from the authoritative source documents (ADR-001, Phase 8A, the Security Floor,
the Authority/Risk/Routing Model, the Booking Correctness process, the four executed
technical spikes, and related material), and states explicitly, for every one of the
prior audit's findings, whether this pass confirms, downgrades, upgrades, or dismisses it.

**Location of the audited vault**: `C:\Users\HP\Desktop\Claude Workspace\1. Shivam's Second
Brain\`. All vault-relative paths below are relative to that root unless stated otherwise.
This handoff file itself lives **outside** the vault, at
`C:\Users\HP\Desktop\Calquartz — Final Adversarial Audit Handoff.md` — it is a handoff
artifact for a human/other-AI reviewer, not vault content.

**How to use this document if you are the independent reviewer.** Read §1–§21 for the
verdict and its full reasoning. Use §A–§H for exactly what to check yourself and what you
can skip. §F gives a tiered reading list if you want to triage what to open first. Every
substantive claim below is tagged **Observed** (I read the exact file/line), **Documented**
(the vault states this as FACT/DECISION and I did not independently re-derive it further),
**Inferred** (my own reasoning from Observed/Documented material), **Reconstructed**
(explicitly labeled as non-recovered synthesis, by the vault's own admission),
**Recommended** (my proposed fix, not binding), or **Unknown** (genuinely unresolved by
anyone). Never read an Inferred or Reconstructed claim below as if it were Observed.

---

## 1. Final verdict

# READY WITH REQUIRED CORRECTIONS

Same top-line verdict as the prior "Final Pre-Authorization Audit." This is **not** a
rubber stamp — see §17 for exactly what was independently re-checked, confirmed, and
added. No Critical or High finding was found by this independent pass either. Three
Medium findings and two Low findings are confirmed or newly identified below (one Medium
finding is new, not present in either prior review). None of them represents a live
security exposure, a silent architectural commitment, or an authorization that would let
unsafe work proceed — every one is a **documentation/completeness gap in the planning
text itself**, not a defect that has (or could yet have) any effect on running code,
since no Calquartz code or repository exists.

---

## 2. Executive rationale

The Calquartz foundation planning stack (Plan → Adversarial Review → Evidence Audit →
Post-Audit Correction Pass → prior Final Pre-Authorization Audit → this pass) is an
unusually self-correcting body of work: each pass independently re-read the authoritative
sources (ADR-001, Phase 8A, the Security Floor, the four executed Pre-Approval spikes)
rather than merely trusting the prior pass's summary, and real corrections were found and
applied at every stage (renamed tenant-context seam, narrowed blocker sequencing, named
boot-gate refusal conditions, added a local-time-policy gate, downgraded premature
packages). This pass independently re-verified that discipline held: **every claim in the
current Plan/WP0–WP9 text that I traced back to ADR-001, Phase 8A, the Security Floor, or
a spike result checked out against the actual source document, with no contradiction
found.**

What keeps the verdict at READY WITH REQUIRED CORRECTIONS rather than READY FOR OWNER
AUTHORIZATION is that several corrections — including one this pass found independently,
not identified by either prior review — exist only as reasoning inside a review document,
not yet carried into the WP0–WP9 file's own prescriptive text the way the Post-Audit
Correction Pass already did for six earlier items. What keeps it from NOT READY FOR
AUTHORIZATION is that none of the gaps found, in this pass or the two prior ones, is a
gap that could let unsafe work proceed: the concrete prohibition list for WP0 (no `SET
LOCAL`, no RLS-specific policies/roles, no `BYPASSRLS`, transport-neutral `contracts`
types, an explicitly-UNKNOWN-labeled test harness) is the thing actually doing the
protective work, and that list is intact, explicit, and independently re-verified present
in the current file text by this pass.

---

## 3. Critical findings

**None.** Independently re-derived, not merely adopted from the prior audit's own "none
identified" conclusion. I looked specifically for: an unresolved architecture/product
decision being silently resolved by WP0–WP9's text; implementation, credentials, or
production access being authorized; HIGH/CRITICAL work being declared complete without a
human gate and an evidence requirement; a security mechanism being implied through
scaffolding shape. None was found. See §13–§14 for the detailed tenant-isolation and
verification analysis behind this conclusion.

---

## 4. High findings

**None.** Two candidates were seriously considered for High and are recorded at Medium
instead (M1, M3 below), in both cases because the current text's own conservative,
explicit prohibitions already prevent the underlying gap from being exploitable — the
defect is in **stated reasoning/completeness**, not in an actually-unsafe authorization.
I deliberately tried to find a reason to upgrade one of these to High (per the task's own
instruction not to preserve the prior verdict's momentum) and could not construct one that
survives contact with the actual WP0–WP9 prohibition text — see §17 for the specific
reasoning on each.

---

## 5. Medium findings

**(M1) WP0/WP3's "No human gate" for the tenant-context-seam scaffolding rests on
reasoning the text states incompletely — and, independently found this pass, the text's
own stated justification is not merely incomplete but factually inconsistent with its own
scope declaration one paragraph above it.**

*Confirms the prior audit's M1, with an added independent finding.* **Observed**
(`Calquartz — Foundation Work Packages (WP0–WP9).md`, WP0 section): the Human Gate
sentence reads *"**No**, for the local-only-scaffolding portion listed above — this is
LOW/reversible work not touching a HIGH domain."* But WP0's own Scope paragraph, two
sentences earlier in the same section, explicitly includes *"`packages/db`'s pool +
tenant-context primitive under mechanism-neutral naming"* — and the very same document's
own WP3 section classifies tenant-context work as *"**Risk tier**: HIGH/CRITICAL — domain
1 (tenant isolation)."* The WP0 Evidence Audit's own filesystem-tree table independently
labels this same primitive *"HIGH (tenant isolation, domain 1)."* **So WP0's own
human-gate rationale sentence — "not touching a HIGH domain" — is not an omission of
context; it is a statement the document's own text elsewhere contradicts.** [Inferred:
this is a sharper, independently-derived restatement of what the prior audit filed as "the
current text never states this reasoning" — that framing undersold the defect as
incompleteness; it is better described as an internal inconsistency between one sentence
and its own document's Scope declaration two lines above it.]

Separately, [Observed, `Authority, Risk, and Routing Model.md` §3.2]: the routing table's
nearest matching row ("New feature, tenant/auth/security-sensitive") gates "**Yes, before
merge/deploy**." Neither that document nor the WP0–WP9 file states anywhere, in so many
words, that local/unmerged/unpushed code is exempt from that gate — the "local-only work
is exempt" reading is an **Inferred** conclusion both prior audits and this one reach, not
a textual fact. [Inferred, independent check]: applying §2.3 of the Authority/Risk/Routing
Model ("default to HIGH whenever genuinely unsure") strictly, a stricter reader could
argue the absence of an explicit "local-only is exempt" row means this case should default
to the HIGH/gated reading rather than the "No" reading the WP0–WP9 file currently states.

**Why this stays Medium, not High** [Recommended reasoning, independently reached]: the
actual operative protection for WP0/WP3 is not the "No gate" sentence — it is the
concrete, explicit prohibition list stated in the same paragraph: *"WP0 must NOT
implement `SET LOCAL`, RLS-specific policies/roles, `BYPASSRLS`, or any other
mechanism-specific tenant-isolation implementation."* That prohibition is unambiguous,
independently checkable against future code, and does not depend on the "not touching a
HIGH domain" framing being correct. A misclassified rationale sentence next to a correct,
concrete prohibition is a documentation defect a future reader could misapply by analogy
(exactly the risk the prior audit named) — but it does not, today, authorize anything
unsafe, because nothing has been built yet to misapply it against.

**Correction required** [Recommended, carrying forward and sharpening the prior audit's
correction 1]: rewrite WP0's and WP3's human-gate sentences to (a) not claim the
tenant-context-primitive sub-scope "does not touch a HIGH domain" — it does, by the
document's own WP3/Evidence-Audit classification — and instead (b) state explicitly that
the "No" gate holds *because* the work remains local, unmerged, and mechanism-free (no
`SET LOCAL`/RLS/`BYPASSRLS`), consistent with the Authority/Risk/Routing Model's
"before merge/deploy" qualifier, and that the gate activates automatically at WP1 (remote
push/CI) or at the first line of mechanism-specific code, whichever comes first.

---

**(M2) No work package in WP0–WP9 explicitly names itself as the home for
recurring-booking series/occurrence scope.**

*Confirms the prior audit's M2, unchanged.* **Observed** (`...WP0–WP9...md`, WP6 section):
WP6 is titled *"Booking-domain skeleton (event types, availability)"* and its Scope names
only event-type/availability schema and the first wall-clock-construction code path — it
does not mention series, occurrence, or recurrence anywhere in its Scope text. **Observed**
(same file, "What this document does not do"): *"It does not select the tenant-isolation
mechanism, the auth library, the recurring-series atomicity model, the local-time product
policy..."* — correctly conservative, but this describes *not selecting a model*, not
*not naming which package eventually owns the schema*. **Documented**
(`ADR-001...md`, "Explicit non-decisions"): *"The recurring-booking series/occurrence data
model's exact design... confirmed as required scope within this architecture; not
designed by this ADR."* Recurring bookings are **Documented** as a DECIDED v1 requirement
(owner decision, 2026-09-08, Phase 5B — `Architecture Open Questions.md`), and
**Documented** (`Phase 6 — Independent Adversarial Architecture Review.md` §5) as fitting
inside A/B1 without requiring a new deployable/datastore/persistence model — a plain
CRUD series/occurrence model.

**Independent assessment** [Inferred]: this is a genuine scope-completeness gap, not an
architectural omission and not a hidden decision — no WP anywhere asserts a schema shape
or an atomicity model for recurring series. The Foundation Implementation Plan's own §1
scopes WP0–WP9 as "the minimum... foundation needed **before** real feature work (booking,
tenancy, auth) begins" — so it is textually defensible that recurring-series *design* is
real feature work, correctly excluded from a foundation-only document. But the document
never says this explicitly; it simply never mentions recurring bookings by name in any
WP's Scope line, which reads, to an unprepared future reader, as an oversight rather than
a deliberate boundary.

**Correction required** [Recommended, unchanged from the prior audit]: either extend WP6's
Scope line to explicitly name series/occurrence schema/module boundary as in-scope
(inheriting WP6's existing gates), or add one explicit sentence stating recurring-series
foundation work is deferred entirely out of WP0–WP9 to a named future phase. Either is
acceptable; silence is not.

---

**(M3) NEW, not identified by either prior review pass: WP7's "Blockers: none" line omits
its own transitive dependency on WP4 — and therefore on blocker 1 (the tenant-isolation
mechanism decision).**

**Observed** (`...WP0–WP9...md`, WP7 section, Scope): *"the session-cookie auth
architecture... `users`/`memberships` kept schematically separate (**already built in
WP4**)."* **Observed**, same WP7 section, Blockers: *"none beyond the standing
security-sensitive-feature gate; does not require the auth library to be selected to
build the architecture-level scaffolding... consistent with Phase 8A §11."* **Observed**,
the document's own Summary table, WP7 row, "Blocked by" column: *"None (library selection
deferred separately)."* **Observed**, WP4 section, Human gate: *"**Yes, always** — this is
the first work package that actually requires the tenant-isolation mechanism (blocker 1)
to be resolved... **This package cannot begin until the owner selects the database-level
tenant-isolation backstop.**"*

**Independent finding** [Inferred, this pass]: WP7's own Scope structurally presupposes
WP4's `users`/`memberships` tables already exist ("already built in WP4"), and WP4 is
itself explicitly and unconditionally gated on blocker 1. WP7's "Blockers: none" line and
the Summary table's "Blocked by: None" cell both omit this transitive dependency. Read
literally, a future implementer could conclude WP7 may start in parallel with, or ahead
of, WP4 — which is not actually possible (there is no table to build against), but the
text does not say so, and a reader building WP7 against an *assumed* membership-table
shape ahead of WP4 actually landing would risk exactly the "structure quietly pre-decides
an open question" failure mode this whole document elsewhere works hard to avoid (see
§3–§16 of the Foundation Plan's own Adversarial Correction Pass, and Review 4 of the
existing Adversarial Review, both of which apply the identical scrutiny to other WPs but
did not catch this one).

**Why Medium, not High**: no unsafe mechanism-specific code follows from this gap — the
practical effect of building WP7 "early" against a nonexistent WP4 schema is that WP7
cannot functionally proceed, not that it proceeds unsafely. It is a planning-completeness
defect of the same kind and severity as M1/M2, not a live risk.

**Correction required** [Recommended]: change WP7's "Blockers" line and the Summary
table's WP7 "Blocked by" cell from "None" to something stating the transitive dependency
explicitly, e.g. *"Depends on WP4's `users`/`memberships` schema existing; transitively
gated on blocker 1 (tenant-isolation mechanism) via WP4, though WP7's own architecture-
level scaffolding does not itself require the mechanism to be selected."*

---

## 6. Low findings

**(L1) `packages/obs`/logger placement stated in two places with mildly inconsistent
framing — confirmed, unchanged from the prior audit.** **Observed**: WP0's scope places
`packages/kernel`/`packages/obs` "as local files, not packages"; WP8's scope separately
describes "the allowlist-by-construction logger (as a local file per correction 7)" as
part of its own scope. Consistent in substance, but neither section states which WP
actually creates the file first. No correction required before authorization; worth a
one-line cross-reference fix.

**(L2) The integration-test harness's foundation-necessity is asserted from a
"Phase 8A §12 requires it" premise, not argued from a stricter minimality-first
premise — confirmed, unchanged from the prior audit, treated as an optional refinement,
not a defect.** A stricter reading could ask whether WP0 needs only a connectivity
smoke-check with fuller harness development deferred to WP3/WP4 (where it is first
actually exercised), rather than a fully-built harness at WP0. This is a judgment call,
not a correction requirement.

---

## 7. Every required correction

In one place, consolidated from §5–§6, in the order a corrector should apply them:

1. (M1) Rewrite WP0's and WP3's human-gate sentences to remove the factually-inconsistent
   "not touching a HIGH domain" claim and state the actual basis for "No" gate: local,
   unmerged, no `SET LOCAL`/RLS/`BYPASSRLS`, gate activates at WP1 or first
   mechanism-specific line of code.
2. (M2) Either extend WP6's Scope to name recurring-series schema/module boundary as
   in-scope, or add an explicit sentence deferring it out of WP0–WP9 entirely.
3. (M3) State WP7's transitive dependency on WP4 (and therefore blocker 1) explicitly in
   WP7's own "Blockers" line and the Summary table's "Blocked by" cell.
4. (L1) State, in either WP0 or WP8, which work package creates the shared
   `logger.ts`/`obs` local file first.
5. (L2, optional) Consider scoping WP0's integration-test harness to a connectivity
   smoke-check only, deferring fuller build-out to WP3/WP4.

None of these five is a precondition for authorizing WP0's already-narrowly-scoped
local-only portion (§9 below) — they are text-quality corrections to the planning
document, not blockers to any specific unit of engineering work.

---

## 8. Every owner decision required

Independently re-derived against the current text, not copied from either prior audit
(though the result substantially agrees with both):

1. **The tenant-isolation mechanism** (RLS vs. a validated alternative) — blocks WP4 only.
   **Documented**: ADR-001 explicit non-decision; RLS is the spike-1-validated leading
   candidate, not selected.
2. **Repository hosting and CI provider** — blocks WP1 only (and is the trigger point at
   which the "before merge/deploy" gate on WP0/WP3's scaffolding, per M1, actually fires).
3. **The ambiguous/nonexistent local-time product policy** — blocks WP6's
   wall-clock-construction portion only. A candidate default is already **Recommended**
   in the text (reject nonexistent local start times; require explicit UTC-offset
   disambiguation for ambiguous ones) — the owner may accept it or replace it.
4. **Confirmation (not design) of the durable-job table migration** against spike 4's
   already-specified shape, before WP2's migration is applied.

**Non-blocking, may remain open indefinitely without affecting WP0–WP9 authorization**:
seats/round-robin scope; the concrete auth library; whether a job library
(`pg-boss`/`graphile-worker`) eventually replaces the hand-rolled protocol; the calendar
provider; the notification provider; a public developer API; embeds; the canonical
product name (working default "calquartz" already accepted).

**Documentation clarification only, not an owner decision**: all five items in §7 above.

**No longer unresolved — already answered by authoritative material, cite where**:
hosted-only vs. self-hosting (**Documented**, `Architecture Open Questions.md`, owner
decision 2026-09-08 — hosted-only); flat vs. nested tenancy (same document, same date —
flat); recurring bookings in v1 scope (same document, same date — yes, required); payments
in v1 (**Documented**, `Architecture Requirements and Constraints.md` §5, owner decision
2026-09-06 — deferred, not v1). None of these needed re-deciding by this pass; none was
found inconsistently stated anywhere in the currently-audited text.

---

## 9. What may safely proceed without those decisions

**Observed**, directly from the WP0–WP9 file's own text, independently re-verified
against every cited authoritative source: local repository initialization; directory
layout (`apps/`, `modules/` empty, `packages/`, `migrations/`, `test/`, `docker/`);
pnpm workspace configuration; dependency-cruiser (boundary-lint) configuration;
`packages/kernel`/`packages/obs` as local files; `packages/config` as a documented
per-entrypoint convention; `packages/db`'s pool + tenant-context primitive under the
renamed, mechanism-neutral naming (`withTenantContext`/`withElevatedAccess`) with the
explicit `SET LOCAL`/RLS/`BYPASSRLS` prohibition intact; `packages/jobs`'s protocol
scaffolding (claim/lease/fence/`SKIP LOCKED`/checkpoint only, permanent-ownership caveat
stated); `packages/time` as a local module with explicit `{ok,...}|{ok:false,...}` result
types; `packages/contracts` as a thin, transport-neutral, types-only package; the
integration-test harness's *existence* (its mechanism-neutrality remains UNKNOWN, not
blocking existence — see §14); local Docker configuration; the durable-job migration
drafted (not applied) as a confirm-not-decide artifact against spike 4's specification.

---

## 10. WP0 item-by-item gate assessment

Independently re-derived, checked against every item's actual current-file classification
(not a prior summary), applying the "does this commit to a security mechanism, tenant
model, DB behavior, worker identity model, job model, schema, package boundary, config
contract, boot behavior, observability architecture, transport choice, test architecture,
or booking semantics?" test named in this audit's own brief:

| WP0 item | Commits to anything? | Current gate | Independently assessed as correct? |
|---|---|---|---|
| Local git init, directory layout, pnpm workspace | Nothing | No | Yes |
| Boundary-lint (dependency-cruiser) config | Module-boundary rules consistent with ADR-001's already-approved module design — not a new commitment | No | Yes |
| `packages/kernel`, `packages/obs` as local files | Nothing — no cited requirement exists anywhere in the sources; correctly downgraded from package to file | No | Yes |
| `packages/config` as convention | Nothing beyond "some boot-gate mechanism will exist" (Security Floor item 15, real but not package-committing) | No for the convention; Yes for actual boot-gate logic (WP8) | Yes |
| `packages/db` pool + tenant-context primitive (mechanism-neutral naming) | **Touches domain 1 (tenant isolation) by the document's own classification** — see M1. Does not commit to RLS specifically (a `withTenantContext(id, cb)` shape is implementable under RLS or an application-layer alternative without a call-site change) but the two-mode (ordinary/elevated) split is structurally suggestive of RLS's own ordinary/`BYPASSRLS` shape — disclosed as a RISK, not hidden | No (rationale internally inconsistent — M1) | Gate outcome correct; **stated rationale is not** |
| `packages/jobs` protocol scaffolding | The protocol (claim/lease/fence/`SKIP LOCKED`) is spike-4-validated FACT; whether the app permanently owns the implementation vs. later adopting a library is explicitly NOT decided, and the caveat is now stated directly in WP0's own text (Post-Audit correction E) | No | Yes |
| `packages/time` local module | Discipline only (explicit ambiguous/nonexistent-time result types); no product policy is baked into a package-level API surface, since it stays a local module until WP6 | No | Yes |
| `packages/contracts` thin package | Types-only, transport-neutral per Post-Audit correction C; does not select REST/tRPC/GraphQL | No | Yes, provided the transport-neutrality constraint is actually enforced once code exists (not yet checkable — no code exists) |
| Integration-test harness existence | Its *existence* is Phase-8A-§12-required; its mechanism-neutrality is explicitly stated UNKNOWN, not assumed | No | Yes — appropriately labeled UNKNOWN, not silently assumed safe |
| Docker (3 entrypoints, local config only) | Process-count from ADR-001 + Phase 8A §5's Next.js/Fastify split rationale, both already-approved; local-only scope does not deploy anything | No | Yes |
| Playwright E2E | Excluded from WP0 entirely (deferred to first real page) | N/A | Yes |

**Conclusion**: every WP0 item's actual gate *outcome* is independently assessed as
correct. The one item where the *outcome* is right but the *stated reason* is wrong is
the tenant-context primitive (M1) — a documentation defect, not a misclassification.

---

## 11. WP0–WP9 sequencing assessment

Independently re-tested, not assumed correct because it already exists (per this audit's
own instruction). The chain WP0 (scaffold) → WP1 (hosting/CI) → WP2 (migrations tooling +
job-table migration) → WP3 (tenant-isolation scaffolding, mechanism-agnostic) → WP4
(tenant schema, gated on blocker 1) → WP5 (worker-role separation) → WP6 (booking
skeleton, gated on local-time policy) → WP7 (auth scaffolding) → WP8 (observability/boot
gate) → WP9 (verification harness) has **no circular dependency** and **no case of a
later architecture decision being smuggled into an earlier WP** — checked WP-by-WP against
the "commits to X?" test in §10's method.

**One genuine hidden-prerequisite gap found, independently, this pass**: WP7 → WP4 →
blocker 1, not stated in WP7's own text (M3, §5 above). No other hidden prerequisite was
found. **No work appears too early** beyond what §10/§16 already flag as premature-if-not-
for-the-file-downgrade (`kernel`/`obs`, already correctly downgraded to files rather than
removed). **No work appears too late**: booking-correctness verification requirements are
named at the WP that actually needs them (WP4, WP6), not deferred wholesale to WP9 — WP9's
own role is *wiring* the runnable-check infrastructure and producing the first rollup
Verification Evidence Record, not being the sole point any verification happens (checked
directly against each HIGH-risk WP's own "Verification required" section, which each
states its own proof requirements independently of WP9). **No security/tenant decision is
required by work that supposedly precedes it**, except the WP7/WP4 gap already named.

**Verdict**: the sequence should **not** be reordered. It should be **annotated** (M3's
correction) so the one real transitive dependency is stated rather than implied.

---

## 12. Booking-correctness assessment

Distinguishing the four categories the task requires, independently derived from
`Calquartz Booking Correctness — Operational Review Process.md`, `Failure-Mode
Analysis.md`, and the four spike Results in `Calquartz — Pre-Approval Technical
Spikes.md`:

**(1) Foundation scaffolding, actually present in WP0–WP9**: the durable-job
claim/lease/fence/`SKIP LOCKED`/checkpoint protocol (WP0/WP2, spike-4-validated FACT); the
tenant-context/enrolment scaffolding (WP0/WP3, spike-1-validated); the wall-clock-correct
time-construction discipline (WP0/WP6, spike-3-validated, gated on the local-time policy).
Nothing else booking-specific exists in WP0–WP9 — correctly, since WP0–WP9 is explicitly
scoped as pre-feature-work foundation.

**(2) Implementation, deferred beyond WP0–WP9 entirely**: the exact booking/occupancy
constraint shape (exclusion constraint recommended by spike 2, not selected); the
idempotency-key mechanism; the recurring-series atomicity model (all-or-nothing vs.
explicit partial success — spike 2 confirmed this is an application-transaction-boundary
choice, still open); cancellation/reschedule exact shape; the occurrence-materialization
batch job's concrete schedule/cadence.

**(3) Unresolved architectural decisions, correctly left open by WP0–WP9**:
tenant-isolation mechanism; booking-constraint shape; recurring-series atomicity;
ambiguous/nonexistent local-time policy (a default is Recommended, not Decided); job
implementation model (hand-rolled permanently vs. library). None of these is designed,
selected, or implied by WP0–WP9's text — independently checked line-by-line against every
package/WP description; no instance found where a WP's scaffolding shape functionally
forecloses one of these decisions, beyond the already-disclosed RLS-shaped-naming RISK
(M1/§10).

**(4) Verification requirements, per booking-correctness category**: double-booking
(concurrent-attempt test, both identical- and overlapping-slot cases — required at the
WP that first builds real constraint code, not WP0–WP9); idempotency (same-key-same/
different-payload test); transactional atomicity (mid-transaction-failure rollback test —
directly tests the Adversarial Correction Pass's own transaction-boundary fix); worker
failure (spike-4 failpoints, reproduced inside real code); DST (spike-3 transition test,
explicitly required by Phase 8A §22 and now present in WP6's "Verification required");
recurring-series-specific proofs (per-occurrence uniqueness, partial-failure isolation,
DST-across-a-full-series). WP0–WP9 correctly does **not** attempt to satisfy any of these
at foundation time — its own text states this explicitly (WP6: "None of these apply at
foundation time specifically... to be carried into the first booking-domain work
package"). No invented design was found or introduced by this audit for any unresolved
item, consistent with the task's own out-of-scope instruction.

---

## 13. Tenant-isolation/security assessment

Checked directly against all 16 Security Floor items, the Multi-Tenancy Trust Model, and
the Database-Level Tenant Isolation document, item by item, for the six named silent-
assumption risks:

- **RLS assumed anywhere?** No functional commitment found. The `withTenantContext`/
  `withElevatedAccess` two-mode shape is structurally *suggestive* of RLS's own
  ordinary/`BYPASSRLS` split — already disclosed as a RISK by the plan's own correction 5
  and independently re-confirmed here, not newly discovered, not hidden.
- **`SET LOCAL` assumed anywhere?** No — explicitly prohibited from WP0's scope by name.
- **Application-only tenant scoping assumed anywhere?** No — the database-level backstop
  remains a named requirement (Security Floor item 12) with the mechanism itself
  unresolved; nothing in WP0–WP9 substitutes an application-only design by default.
- **A specific auth library assumed anywhere?** No — WP7 explicitly excludes library
  selection from its own scope.
- **A specific worker identity model assumed anywhere?** No — WP5 requires "narrower than
  web app" per spike 1 criterion 3, without naming a specific role/grant shape; the exact
  grants remain implementation work at WP5 time, correctly gated on secrets/credentials.
- **A specific database isolation mechanism assumed anywhere?** No — see RLS point above.
- **A specific hosting/deployment architecture assumed beyond ADR-001/Phase 8A's own
  already-approved process-count and stack DECISIONs?** No — Docker's three-entrypoint
  local config is correctly cited (post Post-Audit correction) to both ADR-001's
  process-count requirement and Phase 8A §5's stack rationale, not to ADR-001 alone;
  host-level vs. containerized Postgres and actual orchestration remain open and
  untouched by WP0–WP9.

**Connection-pool reuse**: addressed sufficiently for WP0–WP9's own scope. No tenant-
scoped query runs over the pool until WP4 (gated on the mechanism decision); spike 1
already empirically validated pool-reuse safety for the leading candidate mechanism under
real load. The risk cannot manifest before WP4, and WP4 is correctly gated.

**Worker execution outside the web request path**: addressed sufficiently. WP0 creates
only a worker entrypoint scaffold (no domain handlers, no credential); WP5 gates the
worker's actual database credential behind the standing secrets/credentials rule ("Yes,
always").

**Fail-closed requirements**: WP3's enrolment check is explicitly required to be
catalog-driven (schema-derived at build time), matching spike 1's own fail-closed
requirement and directly responding to the project's own forensic finding (SnagTime's
hand-maintained 27-name list failing open).

**Security behavior before vs. after merge/deploy**: this is exactly M1's subject — see
§5, §10. The distinction exists in the routing model's own text ("before merge/deploy")
but its application to *local, unmerged scaffolding specifically* is an inference the
WP0–WP9 file does not state explicitly. Corrected reasoning is Recommended, not yet
applied to the file.

---

## 14. Verification/proof assessment

For every WP, "what concrete evidence would prove this WP is complete?" — independently
re-checked against each WP's own "Verification required" section:

- **WP0**: boundary-lint fires on a deliberate violation (named, concrete). A
  Verification Evidence Record is required for the package as a whole given its adjacency
  to several HIGH-risk-adjacent domains, per the Adversarial Correction Pass's own
  correction 11 — concrete, appropriately conservative.
- **WP1**: CI actually runs the verification loop on push (a positive, not assumed,
  proof); no secret in repository history — concrete.
- **WP2**: migration up/down/up reversibility; job protocol reproducing spike 4 inside
  real code, including the incremental-checkpoint case specifically (the case spike 4's
  own evidence calls "critically dependent" on getting right) — concrete and correctly
  targeted at the highest-value failure mode.
- **WP3**: fail-closed enrolment fires on a deliberately-unenrolled table (spike 1's own
  success criterion 2, directly reproducible); tenant-context leakage proof (a query
  issued without the seam must fail) — concrete, and correctly identified elsewhere in
  the document chain as the single highest-value missing proof before this correction
  pass added it.
- **WP4**: fail-closed enrolment on the real schema; tenant-context leakage against real
  tables; attacker-controlled cross-tenant access proof (explicitly distinguished from a
  mere schema-enrolment proof, citing the Reuse Audit's own finding that "asserts data
  separation" and "attacker-controlled cross-tenant access" are not the same proof);
  connection-pool contamination proof — concrete, and the strongest verification set of
  any WP, appropriately so given the domain.
- **WP5**: worker-role narrowness proof (spike 1 criterion 3, directly reproducible);
  secret-handling proof — concrete.
- **WP6**: DST forward/backward transition proof (Phase 8A §22's explicit requirement,
  previously dropped by the original plan and restored by the correction pass);
  nonexistent/ambiguous local-time proofs; recurring-occurrences-across-a-full-transition
  proof (named as a *distinct* UNKNOWN from single-construction correctness, not
  conflated with it) — concrete and appropriately layered.
- **WP7**: session-cookie mechanics and CSRF-token *capability* checks, with the actual
  auth/authz-boundary proof explicitly deferred until real endpoints exist. This is
  weaker language ("capability check") than other WPs' proofs, but appropriately so given
  WP7 delivers scaffolding, not a working auth system — independently assessed as
  correctly scoped, not a weak completion criterion in the sense the task's brief warns
  against.
- **WP8**: boot gate refuses three named, concrete bad configurations (missing/
  placeholder/insufficiently-random secret; unverified-TLS DB URL; active demo/local
  fallback — named explicitly by the Post-Audit correction, not left as "three
  unspecified conditions" the way the original plan left it); secret-handling proof —
  concrete.
- **WP9**: its own "verification" is that every named proof above actually exists as a
  runnable check and produces a real Verification Evidence Record, "not an 'agent
  reviewed it' claim" (the document's own words) — appropriately meta, not circular in a
  way that weakens anything, since it does not itself claim any domain-specific property
  is satisfied.

**Is "verification later" used to justify any HIGH-risk work anywhere?** No instance
found, independently checked. **Does any WP's verification requirement itself require an
unapproved architecture choice?** No — every named proof (fail-closed enrolment,
tenant-context leakage, DST transition, worker-role narrowness, job-protocol
reproduction) is stated in a way that is agnostic to which specific mechanism is
eventually selected, consistent with the mechanism-neutral scaffolding it verifies.
**Can verification be performed without implementing the unresolved mechanism?** Yes for
every WP0–WP3 proof; No, by design, for WP4's proofs (which require the mechanism to
exist, and are correctly gated behind its selection).

**No WP's completion criteria were found too weak** to satisfy the safety property they
name, beyond the already-disclosed harness-mechanism-neutrality UNKNOWN (§9, §16), which
is itself correctly labeled UNKNOWN rather than silently assumed satisfied.

---

## 15. Provenance assessment

Independently separated, not deferred to either prior pass's own account:

**Recovered historical fact** (the only concrete WP-number-to-content anchor surviving
anywhere in the vault from the original live `planner`-agent output): WP4/WP5's subject
matter (tenant/membership schema with enrolment; worker-role separation and job-table
confirmation) — **Observed**, stated as such in the WP0–WP9 file's own Provenance section
and independently re-confirmed by both prior passes and this one.

**Reconstruction, explicitly not presented as recovered fact anywhere**: every other WP's
exact number, name, and boundary. **Observed** directly, checked across the WP0–WP9 file,
the Foundation Plan, its Adversarial Review, the Evidence Audit, and the prior Final
Pre-Authorization Audit: none of these documents, anywhere, cites the reconstructed
numbering as if it were `planner`'s original, recovered output. This is a genuinely
**Observed positive finding**, not an assumption — the provenance discipline held across
five independent authorial passes.

**Historical decisions not silently upgraded into owner-approved decisions**: checked
directly. Every DECISION-classified item in the current text (ADR-001's approval; the
three product-scope decisions; the Phase 8A stack approval) traces to an explicit,
dated, quoted or closely-paraphrased owner statement recorded in its own authoritative
document — none was found inferred from a standing RECOMMENDATION or from surrounding
context.

**The Post-Audit Correction Pass log entry, checked against the actual file diffs, not
trusted on its own narrative**: **Observed**, directly, by re-reading the current WP0–WP9
file and Foundation Plan text: all six corrections the log entry claims were made (seam
renaming; boot-gate conditions named; `packages/contracts` transport-neutrality stated;
integration-test-harness UNKNOWN stated; `packages/jobs` permanent-ownership caveat
stated; Playwright relocated out of WP0) are, in fact, present in the current prescriptive
text, not merely in a discussion section. The log entry's own claim that "no item from
the Evidence Audit's §9(1) list was declined or left unaddressed" is independently
confirmed accurate. **No discrepancy found between the log entry's narrative and the
actual file state.**

**No missing source was silently reconstructed as if it existed**: checked directly — the
WP0–WP9 file states plainly, and this audit independently re-confirmed by search, that no
transcript of the original live `planner`/`architect` invocation exists anywhere in the
vault under `wiki/projects/001-calendar-os/` or `raw/projects/001-calendar-os/`. This is
stated as a fact, not papered over.

**New provenance finding this pass, not identified by either prior review** (see §17,
§C): **neither the "WP0 Pre-Authorization Evidence Audit" nor the prior "Final
Pre-Authorization Audit" has its own dedicated `log.md` entry.** Both are referenced only
as pointers inside the "Post-Audit Correction Pass" log entry (`log.md` lines
~1611–1698). Independently re-verified by grep across the entire log file: no entry
titled or containing "WP0 Pre-Authorization Evidence Audit" or "Final Pre-Authorization
Audit" (beyond the two pointer mentions) exists anywhere in `log.md`. This is a genuine
gap against the vault's own root `CLAUDE.md` "Automatic logging" discipline ("Log...
completed phases... significant discoveries... Do not wait to be told 'log this'" — a
completed audit is explicitly named as something that must be logged). It does not affect
the safety or accuracy of either audit's *content* (both were independently re-verified
against the file state this pass, and both check out), only the vault's own institutional-
memory completeness for that specific pass. Not remediated by this audit beyond noting it
and this audit's own log entry now existing for this pass's own work — remediating the two
prior omissions would require editing/creating additional log entries beyond the one this
audit's own boundary permits.

---

## 16. Minimality assessment

Independently re-derived, per-item, not deferred to either prior table though the
conclusion substantially agrees with both:

**Unnecessary work correctly already removed/deferred**: `packages/kernel`,
`packages/obs` (no cited requirement anywhere — correctly downgraded to local files, not
merely flagged); Playwright E2E scaffolding (correctly deferred to the first real page);
`packages/config`, `packages/time`, `packages/contracts` package-level boundaries
correctly downgraded to conventions/local modules/thin-minimal-package respectively,
matched to their actual evidentiary requirement rather than a "looks tidy" default.

**Necessary work correctly present, not gold-plated**: `apps/*` (ADR-001 process-count +
Phase 8A stack requirement); `migrations/` tooling; the integration-test harness's
existence (Phase 8A §12, mirrors the spike methodology that actually caught real defects
in spikes 1/2/3/4 — mocking this would have missed exactly the class of bug the spikes
were run to find); `packages/db` (spike 1); `packages/jobs` protocol scope only (spike 4,
explicitly not a domain-handler framework); the boundary-lint tool (cheap, directly
evidence-motivated by the project's own SnagTime forensic finding of a hand-maintained
enrolment list failing open — this is not a generic best-practice reflex, it responds to a
documented real defect in the project's own reference material).

**One area independently flagged for further minimality scrutiny, not previously named**:
`packages/config`'s convention still implies *some* validated-env-loading code exists at
WP0 time even as "just a convention" — worth confirming, once real code exists, that this
convention doesn't accrete into a de facto standing module before a second entrypoint
actually needs it, the same trap `packages/time`/`packages/contracts` were already caught
falling into. This is a forward-looking watch item, not a current defect (nothing has been
built yet).

**Nothing found that should be added for safety's sake that isn't already present** —
consistent with this audit's own instruction not to recommend complexity merely because it
feels safer. The gaps found (M1–M3) are documentation-completeness, not missing
engineering guardrails.

---

## 17. Explicit comparison against the previous audit

The prior "Final Pre-Authorization Audit" (`Calquartz — Final Pre-Authorization
Audit.md`, last modified 2026-09-09 15:29:53 local) concluded 0 Critical / 0 High / 2
Medium / 2 Low, verdict READY WITH REQUIRED CORRECTIONS.

| Prior finding | This pass's verdict | Reasoning |
|---|---|---|
| Critical: none | **CONFIRM** | Independently re-derived by re-reading every authoritative source directly, not by trusting the prior "none found" — no Critical finding exists in the current text. |
| High: none | **CONFIRM** | Same method. Two candidates (the M1/M3 areas) were deliberately stress-tested for High classification and did not survive — the concrete prohibition/gate text, not the surrounding prose, is what actually protects against unsafe action in both cases. |
| M1 (tenant-context "No gate" reasoning) | **CONFIRM, with an independently sharpened finding** | The prior audit characterized this as the text "never states this reasoning" (an omission). This pass found the text's own "not touching a HIGH domain" sentence is not merely silent on the local-only qualifier — it is directly contradicted by the same document's own WP3/Evidence-Audit HIGH-domain-1 classification of the identical scaffolding item. Same severity (Medium), sharper diagnosis, same correction direction. |
| M2 (recurring-booking scope home) | **CONFIRM, unchanged** | Independently re-checked against the current WP6 text and the document's own "what this document does not do" section — the gap is exactly as the prior audit described it. |
| L1 (logger placement ambiguity) | **CONFIRM, unchanged** | Re-verified directly; still present, still Low, still non-blocking. |
| L2 (integration-harness minimality) | **CONFIRM, unchanged, still non-blocking** | Re-verified as a judgment call, not a defect, matching the prior audit's own framing. |
| *(not previously identified)* | **NEW — M3**, WP7/WP4 transitive-dependency gap | Neither the prior Evidence Audit nor the prior Final Pre-Authorization Audit's own WP0-authorization-boundary table (which covers WP0 sub-operations in detail) extended the same scrutiny to WP7's dependency on WP4. Independently found this pass by applying the same "does this WP's scope presuppose something a stated blocker gates?" test uniformly across all ten WPs rather than only WP0/WP3. |
| *(not previously identified)* | **NEW — provenance observation**, no log.md entry for either the Evidence Audit or the prior Final Pre-Authorization Audit | Found by direct grep across the full `log.md`, not by trusting either audit's own "files changed" section (both state "log.md — one new entry appended" for *their own* creation pass, but no entry matching either audit's own title was found to exist). Does not change either audit's content-level reliability (both were independently content-checked and found accurate) — it is a vault-hygiene finding, not a WP0–WP9 defect. |

**Net effect**: verdict unchanged (READY WITH REQUIRED CORRECTIONS); finding count changes
from 0/0/2/2 to **0/0/3/2** plus one non-severity-tagged provenance observation. No prior
finding was downgraded or dismissed — every one survived independent re-derivation intact.

---

## 18. Findings downgraded, upgraded, removed, or newly discovered by this pass

- **Downgraded**: none.
- **Upgraded**: none (M1 and M3 were both seriously tested for High and did not qualify —
  see §4, §17).
- **Removed/dismissed**: none — every prior finding (M1, M2, L1, L2) independently
  re-derived and confirmed.
- **Newly discovered**: M3 (WP7/WP4 transitive dependency, §5); the log.md provenance gap
  for the two prior audit passes (§15, §17); the sharpened diagnosis of M1's own internal
  inconsistency (§5) — a refinement of an existing finding, not a new one, but substantive
  enough to change how the correction should be worded (§7 item 1).

---

## 19. Exact wording/correction recommendations

Repeated here verbatim from §7 for a corrector who wants only this section:

1. **WP0/WP3 human-gate sentence** — replace *"this is LOW/reversible work not touching a
   HIGH domain"* with wording that (a) does not claim the tenant-context primitive avoids
   domain 1, and (b) states the actual basis: local/unmerged/no mechanism-specific code,
   gate activates at WP1 (remote push/CI) or at the first mechanism-specific line of code,
   whichever comes first.
2. **WP6 Scope line** — add: *"Recurring-series schema/module boundary is [in scope for
   this package, inheriting its gates / explicitly deferred to a named future phase
   beyond WP0–WP9]"* (pick one; either is acceptable).
3. **WP7 Blockers line and Summary-table "Blocked by" cell** — replace *"None (library
   selection deferred separately)"* with wording naming the transitive WP4/blocker-1
   dependency explicitly.
4. **WP0 or WP8** — add one sentence stating which package creates the shared
   `logger.ts`/`obs` file first.
5. **(Optional)** WP0's integration-test-harness scope note — consider narrowing to a
   connectivity smoke-check with fuller build-out explicitly deferred to WP3/WP4.

---

## 20. Exact files actually changed during THIS audit

Minimal, as required by the task's hard boundary:

1. `log.md` (vault root) — **one new entry appended**, documenting this audit. No other
   line in the file was touched.
2. `C:\Users\HP\Desktop\Calquartz — Final Adversarial Audit Handoff.md` — **this file,
   newly created**. It is explicitly **outside** the vault (`1. Shivam's Second Brain\`)
   and is a handoff artifact for an external reviewer, not vault content.

**That is the complete list. No other file was created, edited, renamed, or deleted by
this audit.** In particular: the Foundation Implementation Plan, the WP0–WP9 companion
file, the existing Adversarial Review, the WP0 Pre-Authorization Evidence Audit, and the
prior Final Pre-Authorization Audit were all **read only** and remain exactly as they
were before this pass began.

---

## 21. Confirmation that no implementation/repository/infrastructure/credential work was performed

Explicit, per the task's hard boundary: no application code was written; no repository
was created, cloned, or modified; no database, migration, or schema was created or
altered; no credential, secret, or production/VPS/deployment configuration was created,
read, or modified; no dependency was installed; no architecture, stack, or product
decision was made, changed, or silently converted from RECOMMENDATION/UNKNOWN to
DECISION; no owner decision was invented, inferred, or assumed on the owner's behalf. This
audit is a read (of the vault) and a write (of exactly the two files named in §20) — 
nothing else occurred.

---

## A. Files another reviewer should review

In no particular order beyond grouping; the tiered version for triage is §F.

- `wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation
  Implementation Plan.md` — the primary planning document under audit, including its
  "Adversarial Correction Pass" and "Post-Audit Correction Pass" sections.
- `wiki/projects/001-calendar-os/engineering/Calquartz — Foundation Work Packages
  (WP0–WP9).md` — the companion WP breakdown; the actual object most of this audit's
  findings (M1, M2, M3) are about.
- `wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation
  Implementation Plan — Adversarial Review.md` — the first review pass; its 12 numbered
  "Reviews" and final judgment table are directly relevant to cross-checking §17 above.
- `wiki/projects/001-calendar-os/engineering/Calquartz — WP0 Pre-Authorization Evidence
  Audit.md` — the second review pass; its §1–§9 tables are the direct ancestor of this
  audit's §10–§16.
- `wiki/projects/001-calendar-os/engineering/Calquartz — Final Pre-Authorization
  Audit.md` — the third review pass, the one this audit specifically re-checks; read this
  alongside §17 above.
- `wiki/projects/001-calendar-os/decisions/ADR-001 — A-B1 Modular Monolith
  Architecture.md` — the sole architectural DECISION of record; every "does WP0–WP9
  commit to something ADR-001 left open" claim in this handoff traces back here.
- `wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack
  Selection.md` — including its own "Adversarial Correction Pass" and "Owner Approval"
  sections; the stack DECISION every WP0–WP9 tooling choice cites.
- `wiki/projects/001-calendar-os/engineering/Authority, Risk, and Routing Model.md` —
  §2.1 (the nine risk domains), §2.3 (the classification method), §3.2 (the routing
  table) are the exact text M1 and M3 turn on.
- `wiki/projects/001-calendar-os/architecture/Security Floor.md` — the 16-item gate;
  items 12 and 15 specifically are load-bearing for M1/§13.
- `wiki/projects/001-calendar-os/engineering/Verification Evidence Format.md` — the
  record format §14's assessment is checked against.
- `wiki/projects/001-calendar-os/Calquartz — Pre-Approval Technical Spikes.md` — all
  four spike Results (2026-09-08); the empirical basis for nearly every "FACT" label used
  throughout this handoff.
- `wiki/projects/001-calendar-os/architecture/Phase 6 — Independent Adversarial
  Architecture Review.md` — §5 specifically, the recurring-bookings/A-B1-fit analysis
  behind §12's booking-correctness assessment.
- `wiki/projects/001-calendar-os/engineering/Phase 5B — Factual and Technical
  Prerequisites.md` — §B (the three owner decisions), §C/§D/§E (the evidence-sharpening
  passes that preceded the spikes).
- `wiki/projects/001-calendar-os/questions/Architecture Open Questions.md` — the current,
  authoritative status of every product-scope decision cited in §8.
- `wiki/projects/001-calendar-os/architecture/Architecture Requirements and
  Constraints.md` — §1 (product), §5 (v1 scope table), §6 (Phase 7/8A implementation
  constraints).
- `wiki/projects/001-calendar-os/engineering/Calquartz Booking Correctness —
  Operational Review Process.md` — §1–§9, the direct source for §12's assessment.
- `wiki/projects/001-calendar-os/architecture/Multi-Tenancy Trust Model.md` and
  `wiki/projects/001-calendar-os/architecture/Database-Level Tenant Isolation — RLS and
  Alternatives.md` — the source for §13's RLS-shaped-naming risk assessment.
- `wiki/projects/001-calendar-os/architecture/Threat Model.md` and `wiki/projects/
  001-calendar-os/architecture/Failure-Mode Analysis.md` — background for §12/§13,
  lower priority than the items above (see §F, tier 2).
- `wiki/projects/001-calendar-os/engineering/Calquartz Security Floor — Operational
  Review Process.md` — the per-item "how to check" procedure; relevant to §14.
- `wiki/projects/001-calendar-os/engineering/Destructive-Operation Protection Design.md`
  and `wiki/projects/001-calendar-os/engineering/Durable Decision Capture Policy.md` —
  process documents; relevant background for §15 (provenance) and this audit's own
  boundary compliance, lower priority (see §F).
- `log.md` (vault root) — specifically the "Foundation Implementation Plan — Adversarial
  Correction Pass" and "Post-Audit Correction Pass" entries (search for those exact
  titles), plus this audit's own new entry.
- `wiki/projects/001-calendar-os/CLAUDE.md` — the project operating contract, §46/§47
  specifically, for the Durable Decision Capture Policy pointer and the "current state"
  pointer list.

**Newly identified as relevant by this pass, not in the original request list**: none —
every file this audit found materially relevant was already named or directly resolvable
from the original request's list. The one genuine addition is the **absence** noted in
§15/§17 (no log.md entry for the two prior audit passes), which is a finding about a gap,
not an additional file to read.

---

## B. Files actually changed

Exhaustive, matching §20:

1. **`log.md`** (`C:\Users\HP\Desktop\Claude Workspace\1. Shivam's Second Brain\log.md`)
   — one new append-only entry added at the end of the file. **What changed**: a new
   `## [2026-09-09] phase | ...` section. **Why**: required by this audit's own task
   instructions (documenting the audit per the vault's logging discipline). **Nature**:
   documentation-only, zero effect on any WP0–WP9 file or any implementation.
   **Independent-review need**: low — it is a factual record of what this pass did;
   cross-check it against §1–§21 above for consistency, not against any external system.

2. **`C:\Users\HP\Desktop\Calquartz — Final Adversarial Audit Handoff.md`** (this file) —
   newly created. **What changed**: n/a (new file). **Why**: the task's required output
   artifact. **Nature**: documentation-only. **Independent-review need**: this is the
   file the independent reviewer is reading — no further meta-review of it is needed
   beyond checking its claims against the vault files it cites.

**No implementation file changed.** No file under `wiki/projects/001-calendar-os/`
(other than `log.md` at vault root, listed above) was touched. Stated explicitly per the
task's own requirement: **if no implementation files changed, say so explicitly — they did
not.**

---

## C. Newly discovered evidence

Two items were found in this pass that were not stated, in this specific form, by either
prior review:

**1. WP7's transitive dependency on WP4/blocker 1 (M3, §5, §11).** What was found: WP7's
own Scope text presupposes WP4's `users`/`memberships` tables already exist, but WP7's own
"Blockers" line and the Summary table's "Blocked by" cell both say "None." What it
affects: the WP0–WP9 sequencing assessment (§11) and the completeness of the corrections
list (§7). Whether it changes the verdict: no — it adds a third Medium finding but does
not change the overall READY WITH REQUIRED CORRECTIONS verdict, since (like M1/M2) it is
a documentation-completeness gap, not a live-risk gap. Whether the next reviewer should
independently check it: yes — re-read WP7's Scope and Blockers sections side-by-side with
WP4's Human-gate section in `Calquartz — Foundation Work Packages (WP0–WP9).md` and
confirm the same reading.

**2. Neither the "WP0 Pre-Authorization Evidence Audit" nor the prior "Final
Pre-Authorization Audit" has its own log.md entry (§15, §17).** What was found: grep
across the entirety of `log.md` for both documents' exact titles returns only two
pointer-mentions, both inside the unrelated "Post-Audit Correction Pass" entry — no
dedicated entry exists for either audit pass. What it affects: the provenance assessment
(§15) and the comparison table (§17) — it is a finding *about* the prior audits' own
process compliance, not about the WP0–WP9 planning content those audits reviewed.
Whether it changes the verdict: no. Whether the next reviewer should independently check
it: yes, trivially — `grep "Pre-Authorization" log.md` (or equivalent) and confirm no
entry beyond the two pointer-mentions exists.

Both items **strengthen**, not contradict, the overall provenance conclusion (§15): the
planning-content discipline held across every pass; the one gap found is a
logging-hygiene omission in the review-passes' own process, not a defect in what they
reviewed.

---

## D. Files that are missing

**None.** Every file named in this audit's own source-file request list was resolved to
a real, existing vault file, verified directly via directory listing before any content
was trusted. One naming resolution is worth recording explicitly: the request's
"Phase 8B — Booking Correctness" and "Booking Correctness Operational Review Process"
both resolve to the single actual file `wiki/projects/001-calendar-os/engineering/
Calquartz Booking Correctness — Operational Review Process.md` — there is no separate
"Phase 8B — Booking Correctness" document anywhere in the vault; the actual "Phase 8B"
document is titled `Phase 8B — Minimum ECC Engineering Foundation.md` and covers ECC
agent installation, not booking correctness. This resolution was made by direct listing
of the `engineering/` folder, not guessed.

The one genuine absence in the vault — the original live `planner`/`architect`-agent
transcript that first produced a WP0–WP9 breakdown — is **not** a missing-file finding of
this audit's own discovery; it is already explicitly documented, by the vault's own text,
in three places (`Calquartz — Foundation Work Packages (WP0–WP9).md`'s own "Provenance"
section; the Foundation Plan's §6; the Post-Audit Correction Pass log entry), and this
audit independently re-confirmed the absence rather than merely citing the prior claim
(see §15).

---

## E. Files that do NOT need review

With a one-line reason each, so the omission reads as deliberate:

- `wiki/projects/001-calendar-os/Calquartz — Architecture Decision Brief.md` — superseded
  in practice by ADR-001 (which this audit read directly) for everything this audit's
  questions turn on; the Brief's own content is cited *through* ADR-001 and Phase 6
  throughout the audited documents, not independently.
- `wiki/projects/001-calendar-os/architecture/Architecture Alternatives.md`,
  `Architecture Decision Matrix.md`, `Architecture Recommendation.md` — pre-ADR-001
  architecture-selection material; the selection itself is settled (ADR-001) and not
  reopened by anything this audit found, so the comparative reasoning behind it is
  background, not load-bearing for a WP0–WP9 foundation audit.
- `wiki/projects/001-calendar-os/architecture/Redis Justification Analysis.md`,
  `Infrastructure Constraint Analysis.md`, `Calquartz — VPS Infrastructure Inventory.md`
  — infrastructure/Redis-deferral material; no WP0–WP9 item touches Redis or VPS-level
  infrastructure decisions.
- `wiki/projects/001-calendar-os/architecture/Independent Review Response.md` — an
  earlier-phase (Phase 3) response to an earlier review round, superseded by Phase 6's
  later, more current independent review, which this audit read directly.
- `wiki/projects/001-calendar-os/engineering/Phase 8B — Minimum ECC Engineering
  Foundation.md` — read in part (first ~120 lines) this pass; confirms ECC-agent
  installation only, which has no bearing on WP0–WP9's own content — safe to skip
  entirely unless the next reviewer specifically wants to verify agent-installation
  claims.
- `wiki/projects/001-calendar-os/engineering/ECC Adoption Map.md` — read in part (first
  ~80 lines) this pass for the same reason as above; agent/skill adoption classification,
  not WP0–WP9 content.
- `wiki/projects/001-calendar-os/engineering/Learning Lifecycle and Promotion Policy.md`,
  `Synthetic Evaluation Tasks.md`, `Engineering System Evaluation Harness.md` — govern
  the ECC learning/evaluation system, not this project's application-foundation planning.
- `wiki/projects/001-calendar-os/engineering/drafts/agents/*.md` — the seven installed
  agent definitions; relevant to *how* future implementation work will be routed, not to
  whether the current WP0–WP9 planning text itself is safe to authorize.
- Every file under `wiki/projects/001-calendar-os/reuse/` (Reuse Audit Summary, SnagTime/
  Cal.diy Reuse Candidates, Build-Fresh and Rejected Components, Reuse Security and
  Licensing Findings, Test Reuse and Integration Risks, Engineering-Effort Comparison) —
  Phase 4 forensic material; its conclusions are already fully absorbed into the
  currently-audited documents (cited by file/section wherever load-bearing), and none of
  this audit's findings required re-deriving anything from the raw reuse evidence itself.
- `SnagTime Forensic Analysis.md`, `Cal.diy Forensic Analysis.md`,
  `synthesis/SnagTime vs Cal.diy Synthesis.md` — Phase 2 forensic source material;
  cited-through, not independently needed.
- `wiki/projects/001-calendar-os/README.md`, `STORAGE.md`, `Cal.diy Forensic Analysis.md`
  — project index/status pages; useful for orientation, not for this audit's specific
  findings.

---

## F. Recommended ChatGPT Upload Set

**1. Required** (everything needed to independently challenge this verdict):

- This handoff file itself.
- `Calquartz — Foundation Work Packages (WP0–WP9).md`
- `Calquartz — Application Foundation Implementation Plan.md`
- `Calquartz — Application Foundation Implementation Plan — Adversarial Review.md`
- `Calquartz — WP0 Pre-Authorization Evidence Audit.md`
- `Calquartz — Final Pre-Authorization Audit.md`
- `ADR-001 — A-B1 Modular Monolith Architecture.md`
- `Authority, Risk, and Routing Model.md`
- `Phase 8A — Implementation Stack Selection.md`

**2. Strongly recommended** (needed to verify the specific evidentiary claims behind
most findings):

- `Security Floor.md`
- `Calquartz — Pre-Approval Technical Spikes.md`
- `Phase 6 — Independent Adversarial Architecture Review.md`
- `Architecture Open Questions.md`
- `Architecture Requirements and Constraints.md`
- `Calquartz Booking Correctness — Operational Review Process.md`
- `Verification Evidence Format.md`
- `Multi-Tenancy Trust Model.md` and `Database-Level Tenant Isolation — RLS and
  Alternatives.md`

**3. Optional/background** (only needed for deep provenance or historical-context
verification):

- `Phase 5B — Factual and Technical Prerequisites.md`
- `Threat Model.md`, `Failure-Mode Analysis.md`
- `Calquartz Security Floor — Operational Review Process.md`
- `Destructive-Operation Protection Design.md`, `Durable Decision Capture Policy.md`
- The relevant `log.md` excerpt (the "Foundation Implementation Plan — Adversarial
  Correction Pass" and "Post-Audit Correction Pass" entries, plus this audit's own new
  entry)
- The project `CLAUDE.md` (§46–§47)

---

## G. If any source document was edited

**None was, beyond `log.md`.** No source document under `wiki/projects/001-calendar-os/`
was edited, corrected, or rewritten by this audit — every one of the five corrections
named in §7/§19 is a **recommendation for a future editing pass**, not something this
audit applied itself, per the task's own hard boundary ("Do not modify any other existing
file... you are reviewing them, not editing them"). At no point during this audit did I
find a reason that would have required editing something beyond `log.md` — had I found
one, the task's own instruction is to stop and record the recommendation instead, which is
exactly what §7/§19 do.

---

## H. Evidence-status discipline

Applied throughout §1–§21 above via the inline tags defined in "How to use this
document": **Observed** (I read the exact cited file/section myself, this pass),
**Documented** (the vault states this as FACT/DECISION and this pass did not re-derive it
further, typically because a prior spike or owner decision already settled it),
**Inferred** (my own reasoning chain from Observed/Documented material — always flagged
as such, never presented as if the source document said it directly), **Reconstructed**
(the vault's own explicit label for the WP0–WP9 numbering, carried forward accurately,
never upgraded to "recovered" anywhere in this handoff), **Recommended** (a proposed
correction or reading, explicitly not binding), **Unknown** (genuinely unresolved by
anyone, including this pass — e.g. the integration-test harness's actual
mechanism-neutrality, the tenant-isolation mechanism itself). Every substantive claim in
§1–§16 carries one of these tags, explicitly or by the section's own stated method
(§10/§14's tables state "Observed"/"Independently assessed" in their own column
headers rather than per-cell, since every cell in those tables follows the same method
stated once at the top).

---

## Final handoff completeness check

**Explicitly performed, per the task's own required final step**: "could an independent
reviewer reproduce or challenge this verdict using only the files and evidence identified
in this handoff?" Checked by re-reading §1–§21 and §A/§F as if I were that reviewer with
no session memory: every finding (M1, M2, M3, L1, L2) cites the exact file and the exact
sentence/section it rests on; every "Documented"/"FACT" claim traces to a named file
section; the comparison table in §17 names the prior audit's exact file and modification
timestamp; the Required upload tier in §F contains every file actually cited by number/
name in §1–§16. **Nothing was found missing from the Required tier that §1–§16 actually
depend on.** One gap was found and fixed during this check: the original draft of §F's
Required tier omitted `Phase 8A — Implementation Stack Selection.md` despite §10/§13/§14
citing it repeatedly — added before finalizing this file. This check is the reason that
file is present in §F tier 1 above, not a claim made without having actually performed
the check.
