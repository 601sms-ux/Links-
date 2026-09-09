---
type: synthesis
created: 2026-09-08
updated: 2026-09-08
sources: ["wiki/projects/001-calendar-os/Calquartz — Architecture Decision Brief.md", "wiki/projects/001-calendar-os/architecture/Security Floor.md", "3. ECC Evaluation/Calquartz — ECC Adoption Audit.md"]
tags: [engineering, authority, risk, routing, phase-5a, calendar-os, calquartz]
confidence: medium
provenance: hand-authored, deliverables A2/A3/A4 of Phase 5A
classification: RECOMMENDATION
scope: calquartz-specific
status: DRAFT — not yet approved, not yet enforced by any live mechanism
---

# Calquartz — Authority, Risk, and Routing Model

Deliverables 2, 3, and 4 of [[Phase 5A — Engineering Capability Foundation]]. Defines
**where Claude gets information from and in what order it trusts sources when they
conflict** (Authority), **how a task's risk tier is determined** (Risk/Impact), and **what
that tier requires before the task is considered done** (Routing).

> **Status.** This is a design, not an installed mechanism. Nothing here is enforced by a
> hook, a script, or Claude Code configuration today — it is followed by Claude reading
> and applying it, the same way this project's own `CLAUDE.md` files are followed. It
> becomes an actual gate only once §4's hard-coded triggers are wired into a real
> destructive-operation hook (see [[Destructive-Operation Protection Design]]) inside an
> actual Calquartz repository, which does not yet exist.

---

## 1. Authority hierarchy — source-of-truth precedence

Seven information sources exist for any given question. When two disagree, this is the
order that wins, from highest to lowest. **A lower source may fill a genuine gap a higher
source doesn't cover; it may never silently override what a higher source states.**

**Correction (independent review, this pass)**: this line previously said "six
information sources" while the table below has always had seven rows, ranks 0 through 6
— an off-by-one in the summary sentence, not a change to the hierarchy itself. No rank
was added, removed, or reordered; only the count in the prose was wrong.

| Rank | Source | What it is | When it's authoritative | Failure mode if this rank is violated |
|---|---|---|---|---|
| 0 | **Explicit human instruction, this conversation** | What the owner says right now | For the task at hand, always — but a one-off instruction doesn't retroactively rewrite the Second Brain unless the owner is making a decision meant to persist (in which case it gets written into rank 1) | Claude either ignores a live instruction (bad) or treats a one-off remark as a permanent architectural decision without being told to persist it (also bad — this is exactly the "do not silently turn recommendations into decisions" rule) |
| 1 | **Second Brain — DECISION entries** | An explicitly approved, recorded decision (a future ADR, an owner-confirmed scope item like the v1 payments deferral) | Always, until explicitly superseded by a new decision | Treating a RECOMMENDATION as if it were a DECISION — the single most repeated warning across every phase log entry in this project |
| 2 | **Second Brain — RECOMMENDATION / architecture / security documents** | The Architecture Decision Brief, the Security Floor, the Threat Model, the Failure-Mode Analysis, the Reuse Audit | Authoritative as the current best-evidence position, pending owner decision on anything it flags as open | Inventing a plausible-sounding architectural opinion that contradicts a document that already reasoned through the same question |
| 3 | **Approved + Promoted learned knowledge** | An instinct that has completed the full lifecycle in [[Learning Lifecycle and Promotion Policy]] and reached `Promoted` | Equal to rank 2 once promoted, with provenance preserved back to the observations that produced it | Treating an unpromoted (`Observation`/`Candidate`/`Explicit Validation`) instinct as if it were already authoritative — this is the exact ECC continuous-learning-v2 risk flagged in the ECC audit §6 |
| 4 | **Actual project source code / running system state** | Once a Calquartz repository exists: what the code actually does, what a migration actually applied, what a config file actually says | Ground truth for "what currently is" | Assuming the Second Brain's description of the architecture matches what's actually deployed without checking — drift between design and reality is a **finding to report**, not something to silently paper over by trusting whichever one is more convenient |
| 5 | **ECC (adopted or adapted components)** | Generic agents/skills/rules, whether copied as-is or Calquartz-adapted per [[ECC Adoption Map]] | A template and a mechanism, never a decision-maker | Letting a generic ECC skill's generic advice (e.g., a coverage percentage, a deployment strategy) stand in for a Calquartz-specific decision that document already makes differently |
| 6 | **External research** (web search, general model knowledge, an external reviewer's raw output before incorporation) | Anything not captured in ranks 0–5 | Only to fill a genuine UNKNOWN, and only when labeled as such | Treating a plausible general-knowledge answer as equivalent to project-specific evidence |

**ChatGPT is not a rank in this table.** Its role (per the operating rules: "independent
senior reviewer/challenger") is a *process step*, not a knowledge source to be ranked.
Its output is an input to a human-approval step; once the owner accepts a specific point
from a review, that point is written into the Second Brain at rank 1 or 2 as appropriate
(exactly the pattern already used for the Phase 3 Independent Review Response). Raw,
unincorporated ChatGPT output sits at the same trust level as rank 6 until that happens.

**Unpromoted learned knowledge is not a source at all.** An `Observation` or `Candidate`
instinct (per the learning lifecycle) is data under evaluation. It must never be cited as
if it were true, regardless of its confidence score — this is stated here because rank 3
only applies post-promotion, and the gap between "observed" and "promoted" is exactly
where ECC's default behavior would otherwise let something slip through.

---

## 2. Risk/impact classification

Two tiers only, per instruction not to over-engineer this. A task is **HIGH/CRITICAL** if
it plausibly touches any of the nine named domains below (plus the tenth catch-all
category, §2.3), or if it's genuinely ambiguous. Everything else is **LOW**.

**Correction (consistency audit, this pass)**: this line previously said "eight named
domains" while §2.1 below has always listed nine discrete items plus a tenth catch-all —
an internal miscount in this document's own summary sentence, not a change in the list
itself. The wrong count of eight had already propagated into [[drafts/agents/planner]]
(fixed in the same pass). Fixed by counting §2.1's actual items rather than trusting the
prior prose.

### 2.1 HIGH/CRITICAL domains (fixed list, not learned, not overridable by confidence)

1. Tenant isolation
2. Authentication / authorization
3. Booking correctness (double-booking, idempotency, concurrency)
4. Timezone / DST handling
5. Database schema or data migrations
6. Secrets / credentials
7. Deployment (anything touching a running or production environment)
8. Destructive operations (see [[Destructive-Operation Protection Design]] for the
   specific, enumerated list — that document is the detailed specification this
   classifier defers to for "is this destructive")
9. Security controls generally (rate limiting, headers, audit logging, webhook
   authenticity — the remaining Security Floor items not already covered by 1–2)
10. Anything capable of corrupting durable state (a category, not a list — see 2.3)

### 2.2 LOW — the default, not a separate checklist

Ordinary implementation, documentation, and refactoring with limited blast radius:
adding a UI component, writing documentation, renaming a variable, adjusting a log
message, fixing a typo, adding a test for already-correct behavior. **LOW is defined by
exclusion from 2.1, not by its own positive checklist** — this keeps the classifier from
growing a second, competing taxonomy.

### 2.3 The classification method — a two-part gate, not a single regex

**Part 1 — mechanical signal (fast, not sufficient alone).** Task description and any
named file paths are checked against the domain list above as keyword/path signals:
`tenant`, `workspace`, `auth`, `session`, `permission`, `role`, `booking`, `slot`,
`occupancy`, `reschedule`, `cancel`, `timezone`, `dst`, `migration`, `schema`, `secret`,
`credential`, `.env`, `key`, `deploy`, `production`, `vps`, `docker`, `rate-limit`,
`webhook`, `audit`. A match is a **signal that HIGH review is required**, never
proof that LOW is safe — absence of a keyword match does not clear a task, because a
booking-correctness bug can be described without the word "booking" in it.

**Part 2 — judgment gate (authoritative).** Claude's own read of the task against the
domain list is what actually decides. The keyword list in Part 1 exists to force that
judgment to happen explicitly rather than being skipped, not to replace it. **Default to
HIGH whenever genuinely unsure** — this mirrors the Security Floor's own fail-closed
principle (item 2: "an authorization check that cannot complete must deny, never
default-allow") applied to the classifier itself, not just to Calquartz's eventual
application code.

**Why not a pure regex/keyword classifier**: the same reasoning that rules out a
regex-only destructive-operation gate (§G of the phase brief; detailed in
[[Destructive-Operation Protection Design]]) applies here. A keyword scan is a cheap,
useful first pass and a documented forcing function, not a security boundary in itself.

### 2.4 Risk domains vs. task/routing classes vs. escalation triggers (clarified this pass)

An independent review found that §3.2 below uses "Architecture change" as a routing
category, and it is easy to misread that as an eleventh entry in §2.1's domain list — it
is not, and **it is not being added as one**. §2.1 stays exactly nine discrete domains
plus the tenth catch-all category, unchanged. The confusion is that this document
conflates three distinct things under the word "domain." Naming them separately fixes
the ambiguity without touching the guardrail list:

- **(a) Risk domains — §2.1.** What a task's *content* touches (tenant data, auth, a
  secret, a production system). Fixed, nine-item, not learned, not overridable by
  confidence. This is the list [[Learning Lifecycle and Promotion Policy]] §2 mirrors
  exactly, and the only list that word "domain" should refer to unqualified.
- **(b) Task/routing classes — §3.2's left column.** What *shape* of work this is
  (a new feature, a bug fix, a migration, an architecture question). Some routing classes
  map one-to-one onto a risk domain (`Secrets/credentials` ≈ domain 6;
  `Destructive operation` ≈ domain 8); others are cross-cutting shapes whose actual risk
  tier depends on which domain(s) the specific instance touches (`New feature, ordinary`
  vs. `New feature, ... security-sensitive` are the same shape, split by whether a domain
  applies). **`Architecture change` is a routing class, not a risk domain** — it is
  triggered by a different question entirely (does this require selecting or altering the
  system's foundational structure), answered by checking rank 1–2 authority
  ([[Calquartz — Architecture Decision Brief]] and this project's ADRs, once any exist),
  not by matching against §2.1's content-domain keywords.
- **(c) Escalation triggers.** The specific condition that actually forces
  `Human gate: yes, always` on a given routing class. For most HIGH/CRITICAL rows this
  trigger is simply "the row exists" (secrets, deployment, and destructive operations gate
  unconditionally, because the underlying domain is unconditionally high-stakes). For
  `Architecture change` the trigger is narrower and is restated precisely in §3.2's row
  below: escalation is required only when the task would need to **resolve a genuinely
  new architectural question** — not merely when the task uses architecture-adjacent
  language, and not when an existing rank 1–2 document already answers the specific
  question asked.

**What this does not change**: no risk domain was added, removed, or renamed; no
HIGH/CRITICAL classification became LOW or vice versa; the principle that a genuinely new
architectural decision requires human approval is unchanged and restated, not weakened,
in §3.2's row below.

---

## 3. Automatic routing model

For each task type, the required chain once risk is classified. **A HIGH/CRITICAL
classification adds required steps; it never removes the LOW baseline.**

### 3.1 The LOW baseline (applies to every task)

```
implement → self-check against relevant existing Second Brain docs (rank 1–2)
→ verification-loop (generic checks: build/type/lint/test where applicable)
→ knowledge capture (only if something durable was learned — see the Second Brain's
  own "what counts as meaningful" rule; not every LOW task produces a log entry)
```

### 3.2 HIGH/CRITICAL additions, by task/routing class

**Correction (independent review, this pass)**: this section and its table's left column
were headed "by domain" / "Domain trigger." Per §2.4 above, the rows here are
task/routing classes, not §2.1 risk domains — several rows (`Architecture change`,
`New feature, ordinary`, `Ambiguous / unclear scope`) have no corresponding risk-domain
entry at all. Retitled for accuracy; no row's content changed.

| Task/routing class | Additional required steps (inserted before implementation, except review/verification which follow it) | Human gate? |
|---|---|---|
| Architecture change — **retrieval** (the question is already answered by an existing rank 1–2 document) | `architect` agent checks [[Calquartz — Architecture Decision Brief]] and any recorded ADRs first. If the specific question is already answered there, **cite the existing answer** — this is a normal rank-2 information lookup, not a new decision event | **No** — citing a settled answer is not itself a new architecture decision, and gating it would train the wrong lesson (that retrieving Second Brain knowledge always requires owner sign-off) |
| Architecture change — **new decision** (the question is genuinely not covered by any existing document) | `architect` agent review against [[Calquartz — Architecture Decision Brief]] → **stop**: architecture decisions are owner-only per that document's own §18 approval checklist | **Yes, always** — no genuinely new architecture question is ever resolved by Claude alone |
| New feature, ordinary | `planner` → `tdd-guide` → implement → `code-reviewer` → verification-loop | No, unless it turns out to touch a HIGH domain mid-implementation, in which case re-classify |
| New feature, tenant/auth/security-sensitive | `planner` → `tdd-guide` → implement → **[[Calquartz Security Floor — Operational Review Process]]** → `security-reviewer` → `code-reviewer` → verification-loop → knowledge capture | Yes, before merge/deploy — per the Security Floor's own gate framing (an alternative that fails the floor is unacceptable regardless of other scores) |
| Bug fix, ordinary | `tdd-guide` (regression test first, named after the defect) → `build-error-resolver` if needed → implement → `code-reviewer` → verification-loop | No |
| Bug fix, tenant-isolation | Same as "new feature, security-sensitive" above, **plus** an explicit cross-tenant adversarial test per [[Test Reuse and Integration Risks]] §I.7 (item 1) | **Yes** — a tenant-isolation defect is exactly the class of finding the Security Floor calls release-blocking |
| Bug fix, booking concurrency | `tdd-guide` → **[[Calquartz Booking Correctness — Operational Review Process]]** → implement → `code-reviewer` → verification-loop → knowledge capture | Yes, before merge/deploy |
| Bug fix, timezone/DST | `tdd-guide` (DST-boundary test first, per the spike named in [[Calquartz — Pre-Approval Technical Spikes]]) → implement → `code-reviewer` → verification-loop | No, unless the fix also touches booking correctness (then follow that row) |
| Database schema/data migration | `database-reviewer` (deferred draft — not yet drafted; adaptation plan at §5.1 of [[ECC Adoption Map]]) → migration-safety checklist (the five extracted principles at §5.2 of the same document, pending full `database-migrations` adaptation) → implement → verification-loop against a non-production copy | **Yes** — migrations touching a shape any prior Second Brain document assumed (tenancy shape, booking invariant) require the same architecture-level sign-off as the underlying decision |
| Secrets/credentials | [[Destructive-Operation Protection Design]] gate → explicit human confirmation → execute → Verification Evidence Record | **Yes, always** |
| Deployment | [[Destructive-Operation Protection Design]] gate → pre-deploy verification-loop → explicit human confirmation → deploy → post-deploy health check → Verification Evidence Record | **Yes, always** |
| Destructive operation | [[Destructive-Operation Protection Design]] gate, in full | **Yes, always** |
| Ambiguous / unclear scope | Do not guess a risk tier. Ask, or default to the HIGH-domain chain that most plausibly applies and say why | Depends on what the ambiguity resolves to |

### 3.3 What this table does not do

It does not select an architecture, does not approve any ECC component for installation
beyond what [[ECC Adoption Map]] already classifies, and does not itself constitute the
destructive-operation gate — it routes *to* that gate, the gate itself is specified
separately because it needs its own capability-boundary design, not just a routing
entry.

### 3.4 Refinements found during synthetic evaluation

Two of [[Synthetic Evaluation Tasks]]'s dry-run traces independently exposed the same
general shape of gap: a §3.2 row names a chain, but the document that chain routes *into*
turns out to already flag the specific category as open pending a technical spike. Neither
trace's discovery was folded back into the row itself when first found, per the standing
rule not to silently edit a routing row on the strength of one evaluation finding.
Recorded together here because a second independent occurrence of the same pattern is
itself a reason to make the general principle explicit — the general principle (below)
still stands unchanged; only Task 8's specific status has since changed:

- **Task 8** (ambiguous architecture question) — **now actually incorporated into §3.2**,
  not merely noted. When first traced, this found that the "Architecture change" row's
  escalate-to-`architect`-and-stop instruction was incomplete: if the Second Brain already
  has a reasoned answer to the specific question, the correct action is to **cite that
  answer**, not re-escalate a question already settled. That finding sat as a marginal
  note until the owner independently named the same ambiguity during the 2026-09-08
  review — at that point it stopped being "one synthetic trace's opinion" and became an
  explicit correction request, which is a different footing than the standing
  don't-silently-edit rule was written for. §3.2's "Architecture change" row is now
  actually split into "retrieval" (no human gate) and "new decision" (human gate, always),
  and §2.4 states the general retrieve-vs-decide principle. See
  [[Synthetic Evaluation Tasks]] Dry-run trace 2 for the full trace, corrected in the same
  pass to match.
- **Task 4** (booking concurrency bug) found that the "Bug fix, booking concurrency" row
  correctly routes to [[Calquartz Booking Correctness — Operational Review Process]], but
  that process's §1 (double booking) is itself explicitly open pending spike 2 — so the
  row's chain identifies the right *review process* without being able to guarantee a real
  fix exists to review, since the enforcement mechanism itself isn't chosen yet. See
  [[Synthetic Evaluation Tasks]] Dry-run trace 4.

**The general principle, stated once rather than patched per-row**: a §3.2 HIGH/CRITICAL
row names the correct chain assuming the domain's underlying design question is already
resolved. When the operational process document it routes to shows the specific category
as **Open, pending a named spike**, that spike is a prerequisite the chain does not
currently make explicit — treat it as one anyway, and treat "the process says this is
open" as a stop condition equivalent to the destructive-operation gate's Layer 1 (no
standing capability), not as a caveat to note in passing. This does not change any row's
required steps; it states a reading rule for when a row and its target document disagree
about whether the ground is settled.

---

## Related

- [[Phase 5A — Engineering Capability Foundation]] — the phase this belongs to
- [[ECC Adoption Map]] — which agents/skills this routing table invokes
- [[Calquartz Security Floor — Operational Review Process]] · [[Calquartz Booking Correctness — Operational Review Process]]
- [[Destructive-Operation Protection Design]]
- [[Learning Lifecycle and Promotion Policy]] — governs rank 3 of the authority hierarchy
- [[Calquartz — Architecture Decision Brief]] · [[Security Floor]]
