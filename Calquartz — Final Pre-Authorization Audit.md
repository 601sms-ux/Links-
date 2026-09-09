---
type: synthesis
created: 2026-09-09
updated: 2026-09-09
sources: ["wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation Implementation Plan.md", "wiki/projects/001-calendar-os/engineering/Calquartz — Foundation Work Packages (WP0–WP9).md", "wiki/projects/001-calendar-os/engineering/Calquartz — Application Foundation Implementation Plan — Adversarial Review.md", "wiki/projects/001-calendar-os/engineering/Calquartz — WP0 Pre-Authorization Evidence Audit.md", "wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack Selection.md", "wiki/projects/001-calendar-os/engineering/Phase 8B — Minimum ECC Engineering Foundation.md", "wiki/projects/001-calendar-os/decisions/ADR-001 — A-B1 Modular Monolith Architecture.md", "wiki/projects/001-calendar-os/architecture/Security Floor.md", "wiki/projects/001-calendar-os/engineering/Authority, Risk, and Routing Model.md", "wiki/projects/001-calendar-os/engineering/Verification Evidence Format.md", "wiki/projects/001-calendar-os/engineering/Calquartz Booking Correctness — Operational Review Process.md", "wiki/projects/001-calendar-os/Calquartz — Pre-Approval Technical Spikes.md", "wiki/projects/001-calendar-os/architecture/Phase 6 — Independent Adversarial Architecture Review.md", "wiki/projects/001-calendar-os/engineering/Phase 5B — Factual and Technical Prerequisites.md", "wiki/projects/001-calendar-os/questions/Architecture Open Questions.md", "wiki/projects/001-calendar-os/architecture/Architecture Requirements and Constraints.md"]
tags: [engineering, audit, wp0, foundation, calendar-os, calquartz, final-audit]
confidence: medium
provenance: independent final pre-authorization audit, produced by direct re-reading of the current on-disk state of every authoritative source listed above and of the current WP0–WP9/Plan text being audited; not a specialist-agent output; not a restatement of the existing Adversarial Review or the WP0 Pre-Authorization Evidence Audit — those are read as prior material under audit, not treated as authority, and are left unmodified by this document
classification: RECOMMENDATION — an audit, not a decision, and not itself authorization for any implementation
scope: calquartz-specific, WP0–WP9 foundation scope
status: DRAFT — audit only, no implementation authorized
---

# Calquartz — Final Pre-Authorization Audit

**This document does not authorize anything.** No implementation is authorized by this
audit. No Calquartz repository creation is authorized. No migration is authorized. No
credential is authorized. No production or VPS change is authorized. No unresolved
architecture or product decision may be made, or is made, by this audit — every UNKNOWN
named below remains UNKNOWN after this document exists, exactly as before.

This audit treats the current on-disk state of the Foundation Implementation Plan and
the Foundation Work Packages (WP0–WP9) file (both as of the Post-Audit Correction Pass
logged in `log.md`, 2026-09-09) as the object under audit — not as authority. Authority
for every factual claim below is ADR-001, Phase 8A, the Security Floor, the Authority/
Risk/Routing Model, the Verification Evidence Format, the executed spikes, and the other
sources listed in frontmatter, each read directly in this pass.

---

## 1. Overall verdict

**READY WITH REQUIRED CORRECTIONS.**

Not READY FOR OWNER AUTHORIZATION outright: this audit independently found two Medium
findings (§3) that are real gaps in the current WP0–WP9 text's own internal reasoning
against the Authority/Risk/Routing Model's literal routing table — not merely
restylings of what the prior Evidence Audit already found. Not NOT READY: no Critical or
High finding was independently identified; the plan's own six-item Post-Audit Correction
Pass did genuinely close every item the prior Evidence Audit flagged as still
outstanding, verified directly against the current file text, not merely against the
correction pass's own account of itself. Internal consistency between documents is
necessary but not sufficient for a READY verdict — the standard applied here is whether
the plan is safe to authorize under the project's own authority/risk/routing/
verification/provenance/human-gate rules, independently re-derived in §2–§10 below.

---

## 2. Critical findings

**None identified.** No place was found where the current WP0–WP9 text or the plan it
summarizes silently resolves a genuinely open architecture/product decision, authorizes
implementation, credentials, or production access, or declares HIGH/CRITICAL work
complete without a human gate and evidence record.

---

## 3. High findings

**None identified at High severity.** The two findings below were considered for High
classification and are recorded at Medium instead, with reasoning stated, because in
each case the current text's own conservative framing (local-only scope; explicit
UNKNOWN labeling) already prevents the gap from causing an unauthorized action — the gap
is in stated reasoning/completeness, not in an actual unsafe classification. See §4.

---

## 4. Medium / low findings

**(M1) WP0/WP3's "No human gate" for the tenant-context-seam scaffolding is not
justified against the Authority/Risk/Routing Model's own literal routing table — Medium,
correction required.**

The Authority/Risk/Routing Model §2.1 lists tenant isolation as domain 1, one of nine
fixed HIGH/CRITICAL domains, "not overridable by confidence" (§2.1 header). §3.2's
routing table has no row titled "tenant-isolation scaffolding, non-decision" — the
nearest matching row by domain is "New feature, tenant/auth/security-sensitive," whose
human gate is **"Yes, before merge/deploy."** WP0's `packages/db` tenant-context
primitive (renamed `withTenantContext`/`withElevatedAccess`) and WP3's fail-closed
enrolment/harness scaffolding both plausibly touch domain 1 by §2.3's own two-part gate
("a match is a signal HIGH review is required... default to HIGH whenever genuinely
unsure"), yet both are classified "No" human gate in the current WP0–WP9 file.

Independent assessment: this is **defensible, not wrong** — the "Yes" gate in the
matching §3.2 row is scoped to **"before merge/deploy,"** and WP0/WP3 as currently
written are explicitly local-only, unpushed work (no remote repository exists until
WP1, which is separately gated). A local, unmerged, undeployed scaffold has not yet
reached the trigger condition the row names. But **the current text never states this
reasoning** — it asserts "No" gate without connecting it to the "before merge/deploy"
qualifier that is the only textual basis making "No" correct. A future reader (or a
future session) applying WP0/WP3's classification to a slightly different situation
(e.g., a feature branch that is technically "merged" into a local `main` but still
unpushed) could misapply the same "No" gate reasoning past where it actually holds.

**Correction required (audit only — not applied here)**: the WP0–WP9 file's WP0 and WP3
sections should state explicitly that their "No" human gate rests specifically on
remaining local-only/unmerged/undeployed, per the Authority/Risk/Routing Model §3.2's
"before merge/deploy" qualifier on the matching tenant/security-sensitive row — not on
an implicit claim that scaffolding-only work is categorically exempt from that row.

**(M2) No work package explicitly names itself as the home for recurring-booking
series/occurrence scope — Medium, correction required.**

Recurring bookings are DECIDED as in-scope for v1 (ADR-001 §Context; Architecture Open
Questions). WP6 in the current WP0–WP9 file is titled "Booking-domain skeleton (event
types, availability)" and its Scope names event-type/availability schema and the first
wall-clock-construction code path; its Verification-required section separately lists
"recurring occurrences across a full transition history" as a named proof, and the
Summary table's "What this document does not do" section correctly states the
recurring-series atomicity model is not decided anywhere. But **no WP's Scope section
states that recurring-series schema/module work belongs to it** — WP6 by name and scope
covers only event types and availability, not series/occurrence structure. This is not a
silent decision (no atomicity model or schema shape is asserted anywhere), but it is a
scope-completeness gap: a reader using this document to sequence actual foundation work
would not find recurring-booking scope homed anywhere in WP0–WP9 at all.

**Correction required (audit only)**: either extend WP6's Scope to explicitly name
series/occurrence schema and module boundary as in-scope (still gated identically to the
rest of WP6, since it inherits the same local-time and atomicity UNKNOWNs), or add an
explicit statement that recurring-series work is out of foundation scope entirely and
deferred to a later phase — either is acceptable, but the document should not leave this
implicit.

**(L1) `packages/obs`/logger placement is stated in two places with mildly inconsistent
framing — Low, no correction required before authorization, worth tidying.**

WP0's scope text places `packages/kernel` and `packages/obs` "as local files, not
packages." WP8's scope text separately describes "the allowlist-by-construction logger
(as a local file per correction 7, not a standing package)" as part of *its own* scope.
Read together this is consistent (the logger is a local file, built once, referenced
from both places) but the document does not state which WP actually creates the file
first — a minor cross-reference gap, not a hidden decision or risk misclassification.

**(L2) The integration-test harness's foundation-necessity is asserted, not argued from
a minimality-first premise.**

The current text and the prior Evidence Audit both classify the harness's *existence* as
FOUNDATION-REQUIRED, citing Phase 8A §12. Independent check: this is correct as far as
it goes, but a stricter minimality reading could ask whether WP0 needs a fully-built
harness or only a single smoke-level proof that a real-Postgres, template-clone
connection can be established, with fuller harness development deferred to the first
package that actually needs to run tenant-isolation or booking tests (WP3/WP4). This is
a judgment call, not a defect — recorded as a Low-severity minimality note (§10), not a
correction.

---

## 5. WP0 authorization boundary (precise, independently re-derived)

WP0 is **not** genuinely LOW/reversible as a single unit — the current document's own
"Mixed — the package as a whole is not one task; see split below" framing is correct and
is affirmed here independently, not merely accepted. Checked against the Authority/
Risk/Routing Model §2.1's actual six-way split named in this audit's Part A instruction:

| Sub-operation | Domain touched | Independent classification | Human gate before merge/deploy | Currently allowed local-only |
|---|---|---|---|---|
| Local git init, directory layout, pnpm workspace, boundary-lint config | None of §2.1's nine | LOW | No | Yes |
| `packages/kernel`/`obs` as local files | None | LOW | No | Yes |
| `packages/config` as convention | Domain 6 (secrets), indirectly — boot-gate mechanism, not secret values | LOW-adjacent-to-HIGH; correctly kept out of HIGH because no secret is read or validated yet, only a convention is documented | No | Yes |
| DB pool/tenant-context primitive (`withTenantContext`/`withElevatedAccess`, naming only, no `SET LOCAL`/RLS/`BYPASSRLS`) | Domain 1 (tenant isolation) | **HIGH by domain-touch**, correctly labeled HIGH tier in the current text; "No" gate is defensible only under the local-only/unmerged qualifier named in M1 above, which the text should state explicitly | Not yet triggered (local-only) — will trigger at WP1's remote push per the matching §3.2 row | Yes, scaffolding only, no mechanism-specific code |
| Durable-job protocol scaffolding (claim/lease/fence/`SKIP LOCKED`/checkpoint, zero domain handlers) | Domain 5 (schema, adjacent) | HIGH tier, correctly labeled; protocol itself is spike-4-validated FACT; permanent-ownership question correctly left UNKNOWN | No for protocol scaffolding; Yes specifically for the job-table migration being applied (already correctly gated in WP2) | Yes, protocol only |
| Integration-test harness existence | Not itself a domain, but tests domain-1/3/5 behavior | LOW for existence; the harness's specific mechanism-neutrality is correctly marked UNKNOWN, not FACT, pending real harness code | No | Yes, provided it does not encode RLS-specific mechanics |
| Package/boundary structure (`apps/`, `modules/`, `packages/contracts`) | None directly; `packages/contracts` risks domain-adjacent scope creep (API paradigm) if it leaks transport assumptions | LOW, with a stated constraint (transport-neutral only) already correctly added by the Post-Audit Correction Pass | No | Yes |
| Configuration/boot structure (convention only, not the boot gate's actual refusal logic) | Domain 6, indirectly | LOW for the convention; the boot gate's actual implementation (WP8) is correctly HIGH/gated separately | No for the convention; Yes for the actual gate logic (WP8, already so gated) | Yes for the convention only |

**Conclusion**: no sub-operation was found to be HIGH/CRITICAL *and* incorrectly marked
"No" gate in a way that would allow unsafe action to proceed under this document's own
authorization boundary — the DB tenant-context primitive is the one sub-operation
genuinely at the edge, and its "No" gate is correct only under the local-only/unmerged
reading (M1). The correction the WP0 boundary needs is **stated reasoning, not a changed
classification**: state explicitly that "No" gate for the tenant-context primitive and
the fail-closed harness scaffolding holds specifically because nothing is merged or
deployed yet, and that the gate activates automatically once WP1 (remote push/CI)
occurs — which the document should cross-reference explicitly rather than leave as an
inference a reader must make themselves.

---

## 6. WP0–WP9 corrections required

1. (M1) State explicitly, in WP0 and WP3's own text, that the "No" human gate for
   tenant-context-primitive/harness scaffolding rests on remaining local-only/unmerged/
   undeployed, per the Authority/Risk/Routing Model §3.2's "before merge/deploy"
   qualifier — not on an implicit scaffolding-only exemption.
2. (M2) Either extend WP6's Scope to explicitly name recurring-series schema/module
   boundary as in-scope (inheriting WP6's existing gates), or explicitly state that
   recurring-series foundation work is deferred entirely out of WP0–WP9's scope.
3. (L1) State, in either WP0 or WP8, which work package actually creates the shared
   local `logger.ts`/`obs` file first, to avoid an ambiguous double-ownership read.
4. (L2, optional, not required for authorization) Consider whether WP0's integration-
   test harness needs only a connectivity smoke-check, with fuller harness development
   deferred to WP3/WP4 where it is first actually exercised.

No other correction is required by this audit's independent re-derivation. The six
corrections applied by the 2026-09-09 Post-Audit Correction Pass (seam renaming, boot-
gate conditions, contracts transport-neutrality, harness-shape UNKNOWN, jobs caveat,
Playwright removal) were each independently re-verified present in the current file text
and are not repeated here as outstanding.

---

## 7. Unresolved decisions that must remain UNKNOWN

Checked directly against the current WP0–WP9 file's own text for each item named in this
audit's Part B instruction — none is silently decided by any WP:

- **Tenant-isolation mechanism** (RLS vs. alternative) — WP3/WP4 both state this is not
  decided by their scaffolding/schema work; WP4 is gated on it explicitly.
- **Booking conflict/exclusion mechanism** — not addressed by any WP0–WP9 package;
  correctly deferred beyond foundation scope (consistent with spike 2's own scope, which
  is not a foundation-phase spike).
- **Recurring-series atomicity** — WP6/verification section names it as UNKNOWN, not
  resolved; see M2 above for the separate scope-completeness gap (not a decision leak).
- **Ambiguous/nonexistent local-time policy** — WP6 gates on this explicitly; a
  candidate default is offered as RECOMMENDATION only, per the plan's correction 6,
  correctly not converted to DECISION anywhere in the current text.
- **Concrete auth library** — WP7 explicitly excludes library selection from its scope.
- **Exact durable-job implementation (hand-rolled vs. library)** — WP0/WP5 carry the
  permanent-ownership caveat explicitly; not decided.
- **Public API paradigm** — `packages/contracts`'s transport-neutrality constraint
  (Post-Audit correction C) explicitly avoids deciding this; Phase 8A §19 UNKNOWN,
  unchanged.
- **Calendar provider** — not touched by any WP0–WP9 package.
- **Notification provider** — not touched by any WP0–WP9 package.

All nine items remain genuinely UNKNOWN in the current document. This audit does not
resolve any of them and states explicitly that none may be resolved by Claude alone —
each requires owner decision per the Authority/Risk/Routing Model §1 rank-1 rule.

---

## 8. Verification / evidence requirements

Independently re-checked against the Verification Evidence Format and each WP's stated
"Verification required" section: every WP names at least one concrete proof, and no WP
anywhere declares HIGH/CRITICAL work complete on the basis of "verification later" or an
agent's own unevidenced "reviewed it" claim — this was checked specifically, per this
audit's Part D instruction, and no instance of that pattern was found in the current
text. WP9 explicitly states its own role is to prevent exactly that failure mode
("not an 'agent reviewed it' claim").

Two additions this audit independently identifies as still missing from the named-proof
sets in WP0–WP9 (beyond what the plan's own correction 11 already lists):

- No WP currently names a proof that the boot gate's *config-validation convention*
  (WP0's `packages/config`) and its *actual refusal logic* (WP8) stay consistent with
  each other once both exist — i.e., a proof that WP8's gate actually consumes WP0's
  convention rather than re-implementing config loading separately. Minor, but a real
  gap in cross-WP verification coverage.
- No WP names a Verification Evidence Record explicitly for WP1 (CI activation) beyond
  "CI actually runs the verification-loop on push" — the record format itself (who
  produces it, where it's filed) is correctly left UNKNOWN per the Verification Evidence
  Format's own "Where records live" section, but WP1's text does not cross-reference that
  open question, leaving a reader to rediscover it independently.

Neither addition blocks authorization; both are recorded as verification-coverage
refinements.

---

## 9. Provenance assessment

Independently separated, per this audit's Part F instruction:

1. **Facts recovered from authoritative historical material**: the WP4/WP5 subject-
   matter fragment (tenant/membership schema with enrolment check; worker-role/
   credential separation and job-table confirmation) — the only concrete WP-number-to-
   content anchor the WP0–WP9 file itself claims, and correctly labeled as such in its
   own Provenance section. Every FACT-labeled claim tied to ADR-001, Phase 8A, the
   Security Floor, or a named spike result.
2. **Current decisions**: ADR-001's approval (A/B1); Phase 8A §18's approved stack list;
   the three product-scope decisions (hosted-only, flat tenancy, recurring-bookings-in-
   v1) named in ADR-001's Context. None of these is altered, re-derived, or restated as
   if newly decided by the WP0–WP9 file or this audit.
3. **Claude's reconstruction/synthesis**: **the WP0–WP9 numbering, naming, and package-
   by-package boundary structure is reconstruction, not recovered historical fact** —
   verified directly and stated plainly here, per this audit's explicit instruction not
   to let inferred numbering be presented as fact. The WP0–WP9 file's own Provenance
   section already states this correctly and is not contradicted anywhere else in either
   target document; this audit found no place where the reconstructed numbering is
   presented, elsewhere in the vault, as if it were `planner`'s original, recovered
   output. This is a **positive finding**, not a defect — the provenance framing is
   honest throughout.
4. **Recommendations**: the local-time default (reject-nonexistent/require-explicit-
   offset-for-ambiguous), the package-vs-file downgrades (kernel/obs/config/time), the
   mechanism-neutral seam renaming, and every risk-tier/human-gate assignment not
   directly dictated by the Authority/Risk/Routing Model's own fixed domain list.
5. **UNKNOWNs**: enumerated in full in §7 above.

**No instance was found** of the reconstructed WP numbering, or any RECOMMENDATION-tier
claim, being presented elsewhere as if it were a recovered FACT. Provenance discipline
across both target documents is assessed as **sound**.

---

## 10. Minimality assessment

Independently re-derived per this audit's Part G instruction, not deferred to the prior
audit's table, though substantially agreeing with it on every row checked:

- **Empty modules/packages** (`modules/`): correctly empty, created on demand — agree,
  no correction needed.
- **Docker structure**: three-entrypoint local config only, correctly scoped to
  configuration rather than any actual deployment; agree it is
  FOUNDATION-JUSTIFIED-BUT-REVERSIBLE, not premature, given ADR-001's process-count
  requirement plus Phase 8A §5's split rationale (correctly cited, per correction 10).
- **Config conventions**: agree with the downgrade from package to per-entrypoint
  convention — no concrete second-consumer duplication is measured yet.
- **Integration-test-architecture**: agree the harness's *existence* is foundation-
  required; flag (L2, §4) that its *full build-out* at WP0 time, rather than a minimal
  connectivity check with fuller development deferred to WP3/WP4, is a judgment call the
  document does not argue for explicitly — not a defect, a refinement worth naming.
- **Job scaffolding**: agree, protocol-only scope is correctly minimal and evidence-
  backed (spike 4); the permanent-ownership caveat correctly prevents scope creep into a
  full framework decision.
- **Contract/type packages**: agree `packages/contracts` is FOUNDATION-JUSTIFIED-BUT-
  REVERSIBLE, not premature, since Phase 8A §18 approves end-to-end type sharing as a
  stated reason for the stack pairing — this is DECISION-tier justification, not merely
  a plan-author preference, and the Post-Audit Correction Pass's transport-neutrality
  addition closes the one real risk (paradigm leakage) this audit could find.
- **Repository structure** (`apps/`, `packages/`, `migrations/`, `test/`, `docker/`):
  agree, directly required by ADR-001's process shape and Phase 8A's stack DECISION.
- **Anything that could constrain an unresolved decision**: independently checked each
  package against every UNKNOWN in §7 — no package's proposed shape was found to
  functionally foreclose an UNKNOWN, beyond the already-disclosed RLS-shaped-naming risk
  in the tenant-context seam (already named as a RISK, not a functional commitment, by
  the plan's own correction 5, and independently re-confirmed here: a
  `withTenantContext(tenantId, callback)` signature remains implementable under either
  RLS or an application-layer alternative).

**Verdict on minimality**: the corrected WP0–WP9 structure is close to the smallest safe
foundation, with `packages/kernel` and `packages/obs` correctly downgraded as having no
cited requirement at all, and no other package or structural element found, by this
audit's independent re-check, to be premature beyond what the current text already
concedes.

---

## 11. Exact files changed

Exactly one file changed by Part 1 of this task: `log.md` (vault root) — one new entry
appended.

No other file was modified. This audit document itself is the one new file created by
Part 2 of this task, per the task's explicit file-creation allowance.

---

## 12. Exact log entry created

`log.md`, lines 1611–1698 (vault root; entry begins
`## [2026-09-09] phase | Post-Audit Correction Pass — Foundation Implementation Plan +
WP0–WP9 file: COMPLETE` and runs to the entry's final sentence naming this audit as the
next required reading). The entry records: verification that both target files were
last modified 2026-09-09 ~15:09 local before this pass; the authoritative sources
re-derived from the target files' own citations; the plain statement that the WP0–WP9
file is a reconstruction and that no original planner-agent transcript is durably stored
anywhere in the vault; all six corrections actually made, each verified present in the
current file text before being logged; the explicit finding that no item was declined;
provenance limitations; risk/gate implications (none of the six corrections changed any
risk tier or gate); the current UNKNOWN list; the implementation boundary (no code/repo/
migration/credential/production authorized); the fact that no verification evidence was
produced (planning-only pass); final status COMPLETE; and the next required owner
action, including a pointer to this audit document.

---

## 13. What the owner must explicitly approve next

In priority order, per the current WP0–WP9 authorization boundary as re-derived in §5:

1. **The tenant-isolation mechanism** (RLS vs. a validated alternative) — blocks WP4 only.
2. **Repository hosting and CI provider** — blocks WP1 only; also the point at which the
   "before merge/deploy" gate on WP0/WP3's tenant-context scaffolding (M1, §4) actually
   activates.
3. **The ambiguous/nonexistent local-time product policy** — blocks WP6's wall-clock-
   construction portion only; the plan's stated candidate default may be explicitly
   accepted or replaced by the owner, but is not decided by this audit.
4. **Confirmation (not design) of the durable-job table migration** against spike 4's
   already-specified shape, before that migration is applied (WP2).
5. **The four corrections named in §6 above** to the WP0–WP9 file's own text — these are
   editorial/completeness corrections to the planning document, not owner decisions, but
   are listed here because the owner should be aware they exist before treating the
   corrected document as final.

No other owner action is required before the locally-scoped, non-tenant, non-booking-
time portions of WP0 (repository init, directory layout, boundary-lint, package/file
structure, job-protocol scaffolding, harness existence) may proceed under the project's
already-granted engineering autonomy — consistent with, and not loosening, the
boundary already stated in the current WP0–WP9 file's own summary table.

---

## Related

- [[Calquartz — Application Foundation Implementation Plan]]
- [[Calquartz — Foundation Work Packages (WP0–WP9)]]
- [[Calquartz — Application Foundation Implementation Plan — Adversarial Review]]
- [[Calquartz — WP0 Pre-Authorization Evidence Audit]]
- [[../decisions/ADR-001 — A-B1 Modular Monolith Architecture|ADR-001]]
- [[Phase 8A — Implementation Stack Selection]]
- [[Authority, Risk, and Routing Model]] · [[Verification Evidence Format]]
- [[../architecture/Security Floor|Security Floor]]
- [[Calquartz Booking Correctness — Operational Review Process]]
- [[../questions/Architecture Open Questions|Architecture Open Questions]]
