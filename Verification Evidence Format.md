---
type: synthesis
created: 2026-09-08
updated: 2026-09-08
sources: ["3. ECC Evaluation/Calquartz — ECC Adoption Audit.md"]
tags: [engineering, verification, phase-5a, calendar-os, calquartz]
confidence: medium
provenance: hand-authored; structure informed by ECC's verification-loop skill (superseded, not adopted) and this project's own FACT/INFERENCE/RECOMMENDATION discipline
classification: RECOMMENDATION
scope: calquartz-specific
status: DRAFT — format defined, not yet exercised
---

# Calquartz — Verification Evidence Format

Deliverable 8 of [[Phase 5A — Engineering Capability Foundation]]. A compact, durable
record of *what was actually verified*, replacing a bare "PASS"/"tests passed" claim with
something that can be checked, cited, and trusted later — including by a future session
that wasn't present when the verification ran.

**Why this exists rather than adopting ECC's `verification-loop` skill directly**: that
skill produces a report, but not a *record* — nothing persists it, and its Phase 5
("Security Scan") is a shallow grep that could be mistaken for satisfying the actual
Security Floor. This format fixes both: it's a record (something written down, with a
timestamp, referenceable later), and it explicitly separates "generic checks ran" from
"the Security Floor / Booking Correctness process ran," so one is never mistaken for the
other.

---

## The record

```yaml
verification_id: <YYYY-MM-DD>-<short-task-slug>-<sequence>
task: <one-line description of what was changed>
risk_tier: LOW | HIGH/CRITICAL
risk_domains: [list of triggered domains, or "none" if LOW]
timestamp: <ISO 8601, UTC>
environment:
  branch_or_commit: <if applicable — UNKNOWN if no repo exists yet>
  stack_versions: <UNKNOWN until Calquartz's stack is chosen>
  database: <UNKNOWN until chosen>
checks:
  - name: <what was checked>
    method: <exact command, test file, or manual procedure — not a vague description>
    result: PASS | FAIL | SKIPPED | UNKNOWN
    evidence_ref: <path to test output, log excerpt, or a specific citation>
  - name: ...
security_floor_items_applicable: [list of item numbers from the Security Floor that applied]
security_floor_result: PASS | FAIL | NOT_APPLICABLE | UNKNOWN  # per applicable item — see below
booking_correctness_categories_applicable: [list from the Booking Correctness process]
booking_correctness_result: PASS | FAIL | NOT_APPLICABLE | UNKNOWN  # per applicable category
overall_result: READY | NOT_READY | READY_WITH_CAVEATS
limitations: <what was not tested, and why — never silently omitted>
unresolved_failures: <explicit list; empty list only if genuinely empty, not because a failure was dropped>
human_gate_required: yes | no
human_gate_status: <not yet requested | requested | granted | denied — omit if human_gate_required is no>
reviewer: <agent name(s) or "human" that produced this record>
```

**`security_floor_result` and `booking_correctness_result` are per-item/per-category, not
a single roll-up** — record each applicable item's own PASS/FAIL/UNKNOWN individually
(a table, not a single field) when more than one item applies. The template above shows
the field name as a placeholder for that table, not a single value, to keep the YAML
skeleton short; the actual record expands it.

---

## Field notes

- **`verification_id`** — stable, so a record can be cited elsewhere (a PR description, a
  future session's provenance trail) without ambiguity.
- **`risk_tier`/`risk_domains`** — copied directly from the classification made per
  [[Authority, Risk, and Routing Model]] §2, not re-derived here. If they don't match what
  was actually checked below, that mismatch is itself a finding.
- **`checks`** — every check names its *method* precisely enough that someone else could
  re-run it. "Ran the tests" is not sufficient; "`npm test -- booking.concurrency.spec.ts`,
  20 concurrent requests, exactly 1 success expected" is.
- **`limitations`** — this field exists specifically so that a verification record never
  reads as more complete than it is. A LOW-risk task with only a build+lint check run
  should say so plainly, not omit the field because "nothing's missing" — state
  "no domain-specific checks applied; risk tier LOW" explicitly.
- **`unresolved_failures`** — a record with failures is still a valid, honest record. What
  makes it dishonest is a failure that gets fixed by *not mentioning it happened*. This
  field exists to make that impossible by construction, the same way the Security Floor's
  item 8 makes safe logging structural rather than dependent on remembering to redact.
- **`human_gate_required`/`human_gate_status`** — derived from
  [[Authority, Risk, and Routing Model]] §3's routing table. A record for a task whose
  routing required a human gate is incomplete until this field shows `granted` (or
  `denied`, in which case the change doesn't proceed).

---

## Minimal example (a hypothetical LOW-risk task, for format illustration only — not a real verification)

```yaml
verification_id: 2026-09-08-fix-typo-availability-label-01
task: "Fixed a typo in the availability page's empty-state label"
risk_tier: LOW
risk_domains: [none]
timestamp: 2026-09-08T12:00:00Z
environment:
  branch_or_commit: UNKNOWN — no Calquartz repository exists yet
  stack_versions: UNKNOWN
  database: UNKNOWN
checks:
  - name: "build"
    method: "UNKNOWN — no build tooling chosen yet"
    result: SKIPPED
    evidence_ref: "n/a"
security_floor_items_applicable: []
booking_correctness_categories_applicable: []
overall_result: READY_WITH_CAVEATS
limitations: "This is a format-illustration example only. No real code exists to verify. Do not treat this record as evidence of anything."
unresolved_failures: []
human_gate_required: no
reviewer: "n/a — illustrative example"
```

This example is included specifically so the format is legible before any real code
exists, and is labeled clearly enough that it cannot be mistaken for a real verification
record later.

---

## Where records live, once real ones exist

**RECOMMENDATION, not yet decided**: records could live as a structured comment on a PR,
a file under a `verification/` directory in the Calquartz repo, or a Second Brain page per
significant change (mirroring how this vault already logs meaningful phases). This
document does not decide that — it's a UNKNOWN pending the Calquartz repo's own tooling
and the owner's preference, noted here rather than silently assumed.

---

## Related

- [[Phase 5A — Engineering Capability Foundation]]
- [[Authority, Risk, and Routing Model]] — supplies `risk_tier`/`risk_domains` and the human-gate requirement
- [[Calquartz Security Floor — Operational Review Process]] · [[Calquartz Booking Correctness — Operational Review Process]] — the two processes whose output this format records
- [[Destructive-Operation Protection Design]] — every destructive operation produces one of these records
