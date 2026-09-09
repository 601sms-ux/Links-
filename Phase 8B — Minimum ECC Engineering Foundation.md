---
type: synthesis
created: 2026-09-08
updated: 2026-09-08
sources: ["wiki/projects/001-calendar-os/engineering/ECC Adoption Map.md", "wiki/projects/001-calendar-os/engineering/Authority, Risk, and Routing Model.md", "wiki/projects/001-calendar-os/engineering/Durable Decision Capture Policy.md", "wiki/projects/001-calendar-os/engineering/Verification Evidence Format.md", "wiki/projects/001-calendar-os/engineering/Learning Lifecycle and Promotion Policy.md", "wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack Selection.md", "wiki/projects/001-calendar-os/engineering/drafts/agents/*.md", "wiki/projects/001-calendar-os/decisions/ADR-001 — A-B1 Modular Monolith Architecture.md", "wiki/projects/001-calendar-os/questions/Architecture Open Questions.md", "4. Calquartz Spikes/evidence/*.md"]
tags: [engineering, ecc, phase-8b, calendar-os, calquartz]
confidence: medium
provenance: hand-authored, Phase 8B; installs and lightly adapts material already designed in Phase 5A rather than redesigning it
classification: mixed — see body; no item here is a DECISION unless explicitly marked as owner-approved
scope: mixed — §5/§11/§17 generic, §6/§10 calquartz-specific (stated per-item)
status: ACTIVE
---

# Phase 8B — Minimum ECC Engineering Foundation

## 1. Purpose

Install the smallest useful ECC (engineering orchestration) foundation needed to safely
begin Calquartz implementation later, on top of the now owner-approved stack (Part 0,
this same phase — see [[Phase 8A — Implementation Stack Selection]]'s "Owner Approval"
section). This document records what was installed, why, what was deliberately not
installed, and the evidence that the installed foundation behaves correctly on a
synthetic evaluation suite. It does not implement any Calquartz feature.

## 2. Scope

Installing: agent definitions under `.claude/agents/` at the workspace root; recording
the routing/risk model these agents already carry; documenting the authority hierarchy,
verification model, and gates already designed in Phase 5A but never installed anywhere.
Not in scope: any Calquartz application code, the Calquartz repository, VPS changes,
credentials, or any implementation of auth/booking/RLS/exclusion-constraint/durable-jobs.

## 3. Boundaries (explicitly held)

No Calquartz application feature was implemented. No Calquartz repository was created —
none was required: the workspace-root `.claude/agents/` mechanism (Claude Code's real,
current installation mechanism) works without one. No VPS, infrastructure, or production
system was touched. No credentials were created. No skill was installed speculatively.
Continuous learning / autonomous self-modification was not enabled. Phase 9 was not
started.

## 4. Existing ECC state (investigated before any change)

- **Workspace root `.claude/`** (`C:\Users\HP\Desktop\Claude Workspace\.claude\`) before
  this phase contained only `settings.local.json` (unrelated personal permissions,
  predates this project) and `scheduled_tasks.lock`. No `agents/`, `skills/`, or project
  `settings.json`.
- **`2. Hybrid Scheduling SaaS/cal-diy-reference/.claude/`** — reference material: `rules/`,
  `skills/`, `settings.json`, inherited from the Cal.diy reference checkout. Evaluation
  material, not a template to copy wholesale (per the task's own instruction) — Phase 5A's
  [[ECC Adoption Map]] already extracted the specific reusable pieces (`search-first`,
  `strategic-compact`, `config-protection.js`) from the underlying ECC source, not from
  this reference checkout directly.
- **`3. ECC Evaluation/ecc-reference/.claude/`** — the full upstream ECC repository
  checkout (commit `e04ea0b9cc8248686edf5ac751cadff550e162b8`), containing `commands/`,
  `enterprise/`, `homunculus/`, `research/`, `rules/`, `team/`, `workflows/`,
  `identity.json`, `ecc-tools.json`, `package-manager.json`. This is the ECC audit's raw
  source material, already read and classified by [[ECC Adoption Map]] — not re-audited
  here.
- **[[ECC Adoption Map]]** (staged, Phase 5A) — already contains a complete
  ADOPT/ADAPT/DEFER/REJECT/SUPERSEDED classification of every ECC component against
  Calquartz's needs. This phase does not redo that classification; it installs what it
  already recommended.
- **Staged agent drafts** — `wiki/projects/001-calendar-os/engineering/drafts/agents/`
  contained all 7 expected agents: `planner.md`, `architect.md`, `tdd-guide.md`,
  `code-reviewer.md`, `security-reviewer.md`, `build-error-resolver.md`,
  `refactor-cleaner.md`. All 7 were read in full. Each already: carries a Prompt Defense
  Baseline, a least-privilege tool grant, Calquartz-specific routing instructions citing
  [[Authority, Risk, and Routing Model]], and correct provenance/classification/status
  frontmatter. **These were reusable as-is**, not redesigned — the only changes made were
  (a) updating each file's `status:` line from `DRAFT — staged text` to
  `ACTIVE — installed`, (b) adding a wikilink-resolution note since the files moved out
  of the vault, and (c) filling in `build-error-resolver.md`'s previously-deferred
  stack-specific commands now that the stack is a DECISION.
- **[[Authority, Risk, and Routing Model]]** (staged, Phase 5A) — a complete authority
  hierarchy (7 ranks), two-tier risk classification (10 HIGH/CRITICAL domains + LOW
  default), and a full HIGH/CRITICAL routing table (§3.2) mapping task/routing classes to
  required agent chains and human-gate requirements. This phase installs the agents that
  table already names; it does not redesign the table.
- **[[Verification Evidence Format]], [[Learning Lifecycle and Promotion Policy]],
  [[Durable Decision Capture Policy]], [[Calquartz Security Floor — Operational Review
  Process]], [[Calquartz Booking Correctness — Operational Review Process]]** — all
  already exist, already authoritative, already cross-referenced by the staged agents.
  None needed rewriting for this phase.

**Conclusion**: Phase 5A already did nearly all of the *design* work Phase 8B's task spec
describes. Phase 8B's genuine remaining work was: (1) record the owner's stack approval
(Part 0), (2) actually install the already-designed agents at a real Claude Code
mechanism, (3) fill in the one item that was explicitly deferred pending stack selection
(`build-error-resolver`'s commands), (4) run the synthetic evaluation suite against the
installed (not merely designed) foundation, and (5) produce this artifact. No new
generic-ECC design was invented from scratch.

## 5. Generic foundation (reusable across future projects, not Calquartz-specific)

- The authority hierarchy (7 ranks, [[Authority, Risk, and Routing Model]] §1).
- The FACT/INFERENCE/RECOMMENDATION/DECISION/UNKNOWN evidence-classification discipline
  (vault root `CLAUDE.md`, applied throughout every Second Brain document).
- The Durable Decision Capture Policy's procedure (record → propagate → log, never
  silently upgrade).
- The Verification Evidence Format's classification convention.
- The two-tier LOW/HIGH risk-classification method and its "judgment gate is
  authoritative, keyword scan is a forcing function, not a boundary" principle
  ([[Authority, Risk, and Routing Model]] §2.3) — the *method* is generic; the specific
  10-domain list (§6 below) is Calquartz-specific.
- The Prompt Defense Baseline embedded in every installed agent (unchanged from ECC's
  original, verified sound during the Phase 5A audit).
- The failure-containment principle (§17) and the learning/evolution boundary (§19) —
  both stated as policy here, not yet mechanically enforced by any hook (see §17/§21).

## 6. Calquartz-specific capabilities

- The 10-domain HIGH/CRITICAL risk list ([[Authority, Risk, and Routing Model]] §2.1):
  tenant isolation, auth, booking correctness, timezone/DST, DB schema/migrations,
  secrets, deployment, destructive operations, security controls generally, durable-state
  corruption.
- The 7 installed agents' Calquartz-domain routing (booking, tenancy, DST, the approved
  stack's specific tooling).
- [[Calquartz Security Floor — Operational Review Process]] and [[Calquartz Booking
  Correctness — Operational Review Process]] as the actual checklists `security-reviewer`
  and the booking-correctness routing chain delegate to (not a generic OWASP list).
- `build-error-resolver`'s now-filled-in stack commands (§ above), specific to the
  TypeScript/Fastify/Kysely/Vitest/Playwright stack approved in Part 0.

## 7. Routing model

Unchanged from [[Authority, Risk, and Routing Model]] §3 — this phase installs the agents
that table names; it does not add, remove, or reorder any routing row. Reproduced only by
reference here, per the "one fact, one home" rule; see that document directly for the full
table.

## 8. Risk model

Unchanged from [[Authority, Risk, and Routing Model]] §2 — same 10-domain list, same
two-part mechanical-signal-plus-judgment-gate method, same fail-closed-when-unsure
default. Not restated here.

## 9. Verification model

Unchanged from [[Verification Evidence Format]] — TASK → PLAN → IMPLEMENT → VERIFY →
REVIEW → EVIDENCE → RECORD → LEARN, with risk-dependent verification depth (unit,
integration against real PostgreSQL, concurrency, DST, E2E, security, migration,
type/build checks). Now made concrete for the approved stack: `tsc --noEmit`, `next
build`/`npm run build`, project lint config, `vitest run` (unit + real-PostgreSQL
integration), `playwright test` (E2E) — recorded in `build-error-resolver.md`.

## 10. Security / booking / tenant / DST / job gates

All five gates are preserved exactly as the task spec requires, unchanged from their
existing authoritative homes — this phase adds no new gate content and weakens none:

- **Security**: [[Calquartz Security Floor — Operational Review Process|Security Floor]]'s
  16 items remain authoritative and all UNKNOWN (no Calquartz code exists). No installed
  agent or routing rule claims the floor has passed merely because `security-reviewer`
  exists or was invoked.
- **Booking correctness**: [[Calquartz Booking Correctness — Operational Review
  Process|Booking Correctness]] remains release-blocking; the routing table's "Bug fix,
  booking concurrency" row still routes there before implementation, and spike 2's
  evidence (exclusion-constraint approach) still governs what "correct" means once code
  exists.
- **Tenant isolation**: the routing table's cross-tenant-adversarial-test requirement
  (§3.2, "Bug fix, tenant-isolation" row) is unchanged; ownership-checkability across
  routes/services/workers/webhooks/logs/credentials remains the standard, per the Threat
  Model and Multi-Tenancy Trust Model, neither of which this phase edits.
- **Timezone/DST**: spike 3's wall-clock-construction requirement is unchanged; the
  ambiguous/nonexistent local-time policy remains explicitly UNKNOWN (confirmed again in
  Part 0's approval note — not resolved by the stack decision).
- **Durable jobs**: spike 4's claim/lease/fence/checkpoint protocol is unchanged;
  Part 0's approval note explicitly preserves the distinction between exactly-once CLAIM
  and exactly-once EXTERNAL EFFECT — no installed agent or document conflates them.

## 11. Authority hierarchy

Unchanged, [[Authority, Risk, and Routing Model]] §1. Reaffirmed: explicit owner
instruction (rank 0) > Second Brain DECISIONs (rank 1) > Second Brain
RECOMMENDATIONs/architecture/security docs (rank 2) > promoted learned knowledge (rank 3)
> actual code/system state (rank 4) > ECC components (rank 5) > external research (rank
6). No installed agent is a competing authority — every installed agent's own text
instructs it to check this hierarchy before acting, not to substitute its own judgment
for it.

## 12. Human gates

Unchanged, [[Authority, Risk, and Routing Model]] §3.2: architecture changes (new
decisions, not retrieval), secrets/credentials, deployment, destructive operations,
tenant-isolation defects, booking-concurrency defects, and any genuinely new architecture
question always require a human gate. Nothing in this phase adds or removes a gate.

## 13. Skill inventory

**None installed.** Applying the adversarial-minimization test (§ below) to every
candidate skill named in [[ECC Adoption Map]] found no skill whose responsibility isn't
already carried by an installed agent, an authoritative document the agents already cite,
or ordinary Claude Code behavior:

- `search-first`, `strategic-compact` — adopt-by-reference habits, not installable skill
  files with Calquartz-specific content; no rework needed, nothing to install beyond the
  citation already in [[ECC Adoption Map]] §2.
- `config-protection.js` — a hook, not a skill (see §15); still correctly deferred, since
  it guards config files that don't exist yet (no repository).
- `database-migrations` (5 extracted safety principles) — already recorded as text in
  [[ECC Adoption Map]] §5.2; installing it as a standalone skill file now, before any
  migration tooling is chosen beyond `node-pg-migrate`'s name, would be the "speculative
  infrastructure" the task spec explicitly forbids. Deferred, not rejected.
- `security-review`, `verification-loop`, `deployment-patterns` — already correctly
  SUPERSEDED or REJECTED by [[ECC Adoption Map]]; not revisited.

## 14. Agent inventory

7 agents installed at `C:\Users\HP\Desktop\Claude Workspace\.claude\agents\`:
`planner.md`, `architect.md`, `tdd-guide.md`, `code-reviewer.md`,
`security-reviewer.md`, `build-error-resolver.md`, `refactor-cleaner.md`. Full
per-agent adversarial-minimization answers (task spec §12) are already recorded in each
agent's own Prompt Defense Baseline / Role / routing sections (all read in full during
this phase) and in [[ECC Adoption Map]] §4's table — not duplicated here per "one fact,
one home." Summary of the ten answers, generically, for all seven: (1) each owns one
named recurring responsibility (plan / architecture-boundary-check / test-first
enforcement / code review / security review / build-error triage / dead-code removal);
(2) explicit specialization is justified because each responsibility requires reading a
*different* authoritative document set and applying different judgment; (3) generic
capability alone was rejected specifically because Calquartz's routing table (§3.2)
requires named, invokable specialists, not ad hoc judgment each time; (4) each cites its
specific authoritative documents (Security Floor, Booking Correctness process, ADR-001,
the Decision Brief) inline; (5) each produces a stated output (a plan, a design flag, a
test suite, a review, a security finding, a fix, a cleanup) never a decision; (6) each
explicitly states what it does *not* own (e.g. `planner` never writes code; `architect`
never selects an architecture); (7) each has a least-privilege tool grant limiting what
context/side-effects it can touch; (8) overlap is avoided by the routing table assigning
each a distinct position in the chain, not parallel redundant coverage; (9) each is
invoked automatically per its `description:` field's "Use PROACTIVELY" framing, matching
Claude Code's real automatic-invocation mechanism; (10) evidence of usefulness is
deferred — no Calquartz task has been routed through them yet (§16/§22 below is the first
test, synthetic only).

## 15. Workflow inventory

No standalone workflow files installed. The routing table itself
([[Authority, Risk, and Routing Model]] §3) *is* the workflow definition — each row names
an ordered chain. Installing it a second time as a separate `.claude/workflows/` artifact
would duplicate the same fact in two homes, which the "one fact, one home" rule forbids;
the table stays in the Second Brain and the agents route through it by reading it.

## 16. Automatic-routing behavior

An installed agent's `description:` field is written so Claude Code's own proactive-agent
mechanism invokes it on matching tasks (e.g. `code-reviewer`'s "MUST BE USED for all code
changes"). No custom hook enforces this mechanically yet — this is Claude reading and
applying agent descriptions and the routing table, the same mechanism the vault's own
`CLAUDE.md` files rely on. Verified functionally via §22's synthetic evaluation, not via
a live hook (none exists — see §17/§21).

## 17. Context discipline

Every installed agent reads only its own cited authoritative documents (e.g.
`security-reviewer` reads the Security Floor, not the whole vault); none re-embeds large
document text — each links/cites instead of copying. Confirmed by direct reading: none of
the 7 installed files exceeds roughly 90 lines. No agent spawns another agent
automatically (the routing table sequences agents, but sequencing is Claude's own job
when following it, not an agent-to-agent spawn mechanism defined here).

## 18. Failure containment

Unchanged in principle from the task spec's own §17 — not newly encoded as a hook (none
exists to encode it into), but explicitly stated in the routing table already: a HIGH row
requires its full chain; if a required specialist step is skipped or a review agent isn't
actually run, the routing table's "verification-loop" and "knowledge capture" steps at
the end of every chain are the checkpoint a session must not silently skip past. This
phase does not add mechanical enforcement (no hook exists to check "was
`security-reviewer` actually invoked before this commit") — recorded as a known weakness,
§22 (Adversarial Review).

## 19. Learning/evolution boundary

Unchanged from [[Learning Lifecycle and Promotion Policy]] — not enabled, not modified.
No autonomous self-modification exists anywhere in this installation. Nothing installed
in this phase reads or writes to a learning/promotion pipeline.

## 20. Installation/configuration changes

Exact changes made, evidence-format per task spec §21:

| File | Change | Reason | Source/authority | Verification |
|---|---|---|---|---|
| `.claude/agents/planner.md` (new) | Copied verbatim from `wiki/.../drafts/agents/planner.md`; `status:` line updated to ACTIVE/installed; wikilink-resolution note added | Install already-designed, already-adapted agent at the real Claude Code mechanism | [[ECC Adoption Map]] §4 (ADAPT classification); staged draft itself | Read back after write; frontmatter and body confirmed intact, no content altered beyond the two noted lines |
| `.claude/agents/architect.md` (new) | Same treatment | Same | Same | Same |
| `.claude/agents/tdd-guide.md` (new) | Same treatment | Same | Same | Same |
| `.claude/agents/code-reviewer.md` (new) | Same treatment | Same | Same | Same |
| `.claude/agents/security-reviewer.md` (new) | Same treatment | Same | Same | Same |
| `.claude/agents/refactor-cleaner.md` (new) | Same treatment | Same | Same | Same |
| `.claude/agents/build-error-resolver.md` (new) | Same treatment, **plus**: the "Pending tooling decision" section replaced with actual stack commands (`tsc --noEmit`, `next build`, `npm run lint`, `npx vitest run`, `npx playwright test`), explicitly marked as expected-shape-not-verified (no repository exists) | The stack is now a DECISION (Part 0); the deferral this agent's own text named is resolved for the *stack choice*, though the exact `package.json` script names remain genuinely unverified | [[Phase 8A — Implementation Stack Selection]] §18, Owner Approval note | Read back after write; confirmed the file explicitly disclaims verification against a real repository |
| `wiki/.../engineering/Phase 8A — Implementation Stack Selection.md` | Frontmatter `classification`/`status`/`tags` changed RECOMMENDATION→DECISION (scoped); dated "Owner Approval (2026-09-08, Phase 8B)" section appended before Related | Record the owner's explicit chat approval per the Durable Decision Capture Policy | Owner instruction, this session; Durable Decision Capture Policy | Read back; confirmed no §1–23 substantive reasoning text was altered, only frontmatter and an appended section |
| `wiki/.../questions/Architecture Open Questions.md` | Short "Phase 8B note" appended at end | Propagate the stack approval to a dependent document per Decision Capture Policy step 6, without duplicating content | Durable Decision Capture Policy step 5–6 | Read back; confirmed append-only, no prior text altered |
| `wiki/.../engineering/Phase 8B — Minimum ECC Engineering Foundation.md` (this file, new) | Created | Required artifact, task spec §20 | This task's own instruction | This document |
| `log.md` (vault root) | One `phase` entry appended (§25) | Required by task spec and the vault's own logging rule | Vault root `CLAUDE.md`, "Automatic logging" | Appended after this artifact; see log.md tail |

No other file was created or modified. No VPS, credential, or production system was
touched.

## 21. Rejected/omitted capabilities and why

- **A giant universal engineering framework** — rejected outright per the task's hard
  boundary; nothing beyond the 7 named agents was installed.
- **`database-reviewer` agent, full `database-migrations` skill** — still correctly
  deferred (per [[ECC Adoption Map]] §5.1/5.2): the migration *tool* name
  (`node-pg-migrate`) is now decided, but the agent's example content needs real schema
  shapes that don't exist yet; installing it now would be speculative.
- **`security-scan`/AgentShield** — still DEFER; requires an external npm package and a
  separate API key, and there is no `.claude/` config complex enough yet to meaningfully
  scan.
- **Any `multi-*` command** (routes to external OpenAI/Google runtimes) — still REJECT,
  unchanged reasoning from [[ECC Adoption Map]].
- **`continuous-learning-v2`** — still DEFER, policy-gated; not enabled.
- **A destructive-operation hook** ([[Destructive-Operation Protection Design]]) — not
  installed. Genuinely deferred, not omitted by oversight: it needs an actual repository
  and actual destructive-capable commands to gate; installing an empty/no-op hook now
  would be exactly the "speculative infrastructure" the task spec forbids. Recorded as a
  known gap, §22 below.
- **A workspace-root `CLAUDE.md`** — deliberately not created. It would auto-load for
  every session in this multi-project workspace (including unrelated future projects),
  contaminating the generic layer with Calquartz-specific content — exactly what task
  spec §13 forbids. The project's own `wiki/projects/001-calendar-os/CLAUDE.md` already
  serves this role correctly scoped.

## 22. Remaining UNKNOWNs

Everything Part 0 explicitly left unresolved remains unresolved: concrete auth library;
ambiguous/nonexistent local-time policy; recurring-series atomicity model; exact
tenant-isolation implementation; exact exclusion-constraint implementation; exact
durable-job implementation; whether a job library could replace the app-owned
implementation; real VPS capacity/performance under load; seats/group bookings;
round-robin; calendar provider; notification provider; public developer API; embeds;
canonical product name. Additionally, ECC-specific: whether the installed agents actually
improve outcomes once real Calquartz tasks are routed through them (no real task has been
routed yet — only the synthetic suite below); whether a destructive-operation hook will be
straightforward to wire once a repository exists.

## 23. Verification evidence

**Synthetic routing/evaluation suite** (task spec §22), run by hand-tracing each
hypothetical task against [[Authority, Risk, and Routing Model]] §2–3 and the 7 installed
agent descriptions — not executed against a live repository, since none exists:

| # | Task | Risk tier | Routing selected | Sized appropriately? | Gate-passed falsely claimed? |
|---|---|---|---|---|---|
| 1 | Change README wording | LOW | LOW baseline only (self-check + verification-loop) — no agent invoked | Yes — no agent chain for a wording change | No |
| 2 | Add a user profile field | LOW (unless the field is tenant/auth-sensitive) | `planner` (light) → `tdd-guide` → implement → `code-reviewer` → verification-loop, per "New feature, ordinary" | Yes — ordinary feature chain, no security/booking specialist forced | No |
| 3 | Change booking conflict logic | HIGH (domain 3, booking correctness) | `tdd-guide` → [[Calquartz Booking Correctness — Operational Review Process]] → implement → `code-reviewer` → verification-loop → knowledge capture; human gate before merge | Yes — full HIGH chain, not the LOW baseline | No — the routing table's own booking-correctness process is itself flagged Open pending design (§3.4's general principle), so the chain does not claim correctness is verified merely by running |
| 4 | Change recurring booking generation | HIGH (domains 3+4, booking + DST, and now v1-scope per Part 0) | Same as #3, plus DST-boundary test per `tdd-guide`'s domain-specific test requirement; `architect` consulted if the recurring atomicity model (still UNKNOWN) must be decided as part of the change | Yes | No — explicitly routes to a still-UNKNOWN design question rather than assuming an answer |
| 5 | Change timezone conversion | HIGH (domain 4) | `tdd-guide` (DST-boundary test first, spike-3-pattern) → implement → `code-reviewer` → verification-loop | Yes | No — Luxon's silent ambiguous/nonexistent-time behavior is a named, not-hidden gap |
| 6 | Modify tenant membership authorization | HIGH (domains 1+2) | "New feature, tenant/auth/security-sensitive" row: `planner` → `tdd-guide` → implement → Security Floor review → `security-reviewer` → `code-reviewer` → verification-loop → knowledge capture; human gate before merge | Yes | No — explicitly capability-only language preserved, no "floor passed" claim |
| 7 | Change worker retry behavior | HIGH (domain 7-adjacent: durable jobs / domain 10 durable-state) | `tdd-guide` → implement (respecting claim/lease/fence protocol) → `code-reviewer` → verification-loop; flagged for `security-reviewer` if retry touches cross-tenant batch logic (spike 4's shape) | Yes | No — exactly-once CLAIM vs. exactly-once EFFECT distinction is preserved, not conflated |
| 8 | Add a database migration | HIGH (domain 5) | `database-reviewer` step marked "deferred draft, not yet installed" — routing table's own stated gap; falls back to the 5 extracted safety principles ([[ECC Adoption Map]] §5.2) plus verification-loop against a non-production copy; human gate | **Partially** — correctly HIGH, correctly gated, but the ideal specialist agent doesn't exist yet; this is a real, acknowledged capability gap, not a false pass | No — the routing table explicitly does not claim `database-reviewer` ran; it names the gap |
| 9 | Add an external webhook | HIGH (domain 9, security controls: webhook authenticity) | Security Floor review → `security-reviewer` → `code-reviewer` → verification-loop; human gate | Yes | No |
| 10 | Change a security header | HIGH (domain 9) but narrow blast radius | Security Floor review (single relevant item) → `code-reviewer` → verification-loop; `security-reviewer` invoked but scoped to the one item, not the full floor | Yes — HIGH tier correctly triggered without inflating to the full 16-item floor review for a one-header change | No |

**Result**: all 10 tasks received a risk tier and routing chain of the correct order of
magnitude — no LOW task triggered a multi-agent HIGH chain, no HIGH task was handled by
the LOW baseline alone. Task 8 surfaced a genuine, pre-existing capability gap
(`database-reviewer` not yet installed) rather than a false pass — the routing table
already names this gap explicitly, and this evaluation confirms the gap is still
correctly visible, not silently papered over.

## 24. Adversarial review

- **Biggest source of unnecessary complexity added**: none added by this phase itself —
  it installed existing design rather than creating new structure. The pre-existing risk
  is the seven-agent routing chain itself for HIGH tasks (planner → specialist →
  implement → reviewer → verification-loop can be a lot of steps for a task that is HIGH
  only by a narrow margin, e.g. task 10 above) — mitigated, not eliminated, by scoping the
  specialist step to the specific item rather than the full checklist.
- **Biggest routing-failure risk**: nothing mechanically enforces that Claude actually
  follows the routing table — it is convention, read and applied, exactly like the
  vault's own `CLAUDE.md` files. A future session (or a rushed one) could skip a required
  HIGH-tier step and nothing would stop it. This is the same limitation the task spec's
  own §17 anticipates and §21's rejected destructive-operation hook would eventually
  narrow, once a real repository exists to wire it against.
- **Biggest context-cost risk**: seven agents each carrying their own Prompt Defense
  Baseline and citing multiple Second Brain documents means a HIGH-tier task can pull in a
  meaningful amount of cross-referenced context across several files. Mitigated by each
  agent citing rather than duplicating; not eliminated.
- **Biggest false-confidence risk**: an agent name like `security-reviewer` existing and
  being invoked could be mistaken, by a future reader of a session transcript, for proof
  the Security Floor passed. Every installed agent's text explicitly disclaims this
  ("capability, not verification"), but the disclaimer lives in the agent file, not in
  any user-facing summary a future session might produce — a future session must actively
  preserve that distinction when reporting results, not assume the agent file's wording
  alone prevents the conflation.
- **Biggest governance risk**: none found where ECC overrides or duplicates Second Brain
  authority — every installed agent explicitly subordinates itself to the authority
  hierarchy and cites Second Brain documents rather than restating their content. The
  risk that exists is omission, not override: an agent could fail to cite a document that
  gets updated later, going stale silently. No mechanism here detects that drift.
  automatically.
- **Biggest maintenance risk as future projects are added**: none of the 7 installed
  agents or their routing logic are project-agnostic — every one is explicitly
  Calquartz-scoped (booking, tenancy, DST). A second project under this workspace would
  need its own agent set; nothing here was written to generalize, correctly, per task
  spec §13's separation of generic vs. project-specific.
- **What should NOT exist yet**: `database-reviewer`, the full `database-migrations`
  skill, the destructive-operation hook, any skill file (none met the adversarial bar),
  and the Calquartz repository itself. All correctly absent.

## Phase 8B completion status

## PHASE 8B — COMPLETE

**Files created**:
- `wiki/projects/001-calendar-os/engineering/Phase 8B — Minimum ECC Engineering
  Foundation.md` (this document)
- `.claude/agents/planner.md`, `architect.md`, `tdd-guide.md`, `code-reviewer.md`,
  `security-reviewer.md`, `build-error-resolver.md`, `refactor-cleaner.md` (workspace
  root)

**Files modified**:
- `wiki/projects/001-calendar-os/engineering/Phase 8A — Implementation Stack Selection.md`
  (frontmatter + appended Owner Approval section, Part 0)
- `wiki/projects/001-calendar-os/questions/Architecture Open Questions.md` (appended
  cross-reference note, Part 0)
- `log.md` (appended, §25)

**ECC capabilities installed**: 7 Calquartz-domain agents (see §14), no skills, no
workflows, no hooks — matching the "start thin" principle; every omission is justified in
§21.

**Routing behavior established**: the pre-existing [[Authority, Risk, and Routing
Model]] §3 table, now backed by real, invokable agent files rather than staged text.

**Synthetic evaluation result**: 10/10 tasks received correctly-sized routing (§23); one
genuine, correctly-surfaced capability gap found (task 8, `database-reviewer` not yet
installed — deliberate, not an oversight).

**Remaining UNKNOWNs**: unchanged from Part 0's list, plus the ECC-specific unknowns in
§22 (real-task effectiveness untested, destructive-op hook not yet wired).

**Known weaknesses**: no mechanical enforcement of the routing table (§24); no
destructive-operation hook yet (correctly deferred, §21); `database-reviewer` still
missing (correctly deferred, §21).

**Deliberately deferred**: `database-reviewer` agent, `database-migrations` skill,
`security-scan`/AgentShield, the destructive-operation hook, any workspace-root
`CLAUDE.md`, all language/framework-specific ECC components not matching the approved
stack.

**Confirmation**: no Calquartz application code was written; no Calquartz repository was
created; no auth, booking, recurring-booking, RLS, exclusion-constraint, or durable-job
implementation was begun; no VPS or production system was touched; Phase 9 was not
started; ADR-001 was not modified.

## Related

- [[Phase 8A — Implementation Stack Selection]] — the approved stack this foundation is scoped to
- [[ECC Adoption Map]] · [[Authority, Risk, and Routing Model]] · [[Durable Decision Capture Policy]]
- [[Verification Evidence Format]] · [[Learning Lifecycle and Promotion Policy]]
- [[Calquartz Security Floor — Operational Review Process]] · [[Calquartz Booking Correctness — Operational Review Process]]
