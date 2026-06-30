# Story M4.1: Add Outcome Mapping Fixtures

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a BMAD-METHOD maintainer,
I want deterministic fixtures for automated review outcomes,
so that route-facing contracts can be trusted by Archon.

## Acceptance Criteria

1. Given a `patch` fixture runs, when automated review emits a contract, then `patch_count > 0` and `gate` is `FAIL`.
2. Given a `decision_needed` fixture runs with no patch findings, when automated review emits a contract, then `decision_needed_count > 0` and `gate` is `CONCERNS`.
3. Given `defer` and `dismiss` fixtures run, when automated review emits contracts, then the matching counts increment and human-readable reasons are preserved.
4. Given invalid evidence or untrusted output is present, when validation runs, then the result is `ERROR`.

## Tasks / Subtasks

- [ ] Verify dependency implementation before adding M4.1 fixtures.
  - [ ] Confirm Story M1.2 has implemented normalized automated findings with `patch`, `decision_needed`, `defer`, and `dismiss`.
  - [ ] Confirm Story M2.1 has implemented `code-review-auto.gate.json` with `gate`, all count fields, `blocking_findings_count`, `decision_needed_file`, `report_file`, and `story_file`.
  - [ ] Confirm Story M3.1 has implemented durable `decision-needed.json` for `decision_needed` findings.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill validates.
  - [ ] If the automation skill source directory or dependency behavior is missing, stop and complete the dependency story first.
  - [ ] AC: 1, 2, 3, 4.

- [ ] Locate or create the deterministic fixture harness in tracked source.
  - [ ] Prefer the fixture or test harness established by M1.2, M2.1, or M3.1.
  - [ ] If no harness exists, create one in a tracked repository test or fixture location such as `test/fixtures/` or a test path owned by the implemented auto-review source.
  - [ ] Keep fixtures deterministic and local.
  - [ ] Do not require network access, Linear access, Archon execution, or parent workspace files.
  - [ ] Do not edit `.agents` or `_bmad` generated install output as source.
  - [ ] AC: 1, 2, 3, 4.

- [ ] Add the outcome fixture matrix.
  - [ ] Add a `patch` fixture with at least one normalized finding categorized as `patch`.
  - [ ] Add a `decision_needed` fixture with at least one normalized finding categorized as `decision_needed` and no `patch` findings.
  - [ ] Add a `defer` fixture with at least one normalized finding categorized as `defer` and a preserved human-readable reason.
  - [ ] Add a `dismiss` fixture with at least one normalized finding categorized as `dismiss` and a preserved human-readable reason.
  - [ ] Add an invalid evidence fixture that validation must reject.
  - [ ] Add an untrusted output fixture that validation must reject.
  - [ ] Add a PASS baseline fixture with no `patch` and no `decision_needed` findings, because architecture requires validation for `PASS`, `FAIL`, `CONCERNS`, and `ERROR`.
  - [ ] AC: 1, 2, 3, 4.

- [ ] Assert route contract output for each valid fixture.
  - [ ] Assert the `patch` fixture emits `patch_count > 0`.
  - [ ] Assert the `patch` fixture emits `gate: "FAIL"`.
  - [ ] Assert the `patch` fixture emits `blocking_findings_count > 0`.
  - [ ] Assert the `decision_needed` fixture emits `decision_needed_count > 0`.
  - [ ] Assert the `decision_needed` fixture emits `patch_count: 0`.
  - [ ] Assert the `decision_needed` fixture emits `gate: "CONCERNS"`.
  - [ ] Assert the `decision_needed` fixture emits `decision_needed_file` pointing to `decision-needed.json`.
  - [ ] Assert the PASS baseline emits `gate: "PASS"` and zero active blocking counts.
  - [ ] AC: 1, 2.

- [ ] Assert `defer` and `dismiss` evidence preservation.
  - [ ] Assert the `defer` fixture increments `defer_count`.
  - [ ] Assert the `defer` fixture preserves the triage reason in the structured output or human report artifact implemented by earlier stories.
  - [ ] Assert the `dismiss` fixture increments `dismiss_count`.
  - [ ] Assert the `dismiss` fixture preserves the dismissal reason in the structured output or human report artifact implemented by earlier stories.
  - [ ] Assert `defer` and `dismiss` do not produce `FAIL` or `CONCERNS` by themselves.
  - [ ] Do not make Archon parse Markdown to recover these reasons.
  - [ ] AC: 3.

- [ ] Assert invalid or untrusted output handling.
  - [ ] Assert invalid evidence produces `ERROR`.
  - [ ] Assert untrusted output produces `ERROR`.
  - [ ] Assert validation failures include diagnosable error detail.
  - [ ] Assert invalid or untrusted fixtures never fall through to `PASS`.
  - [ ] Assert no partial success is reported for invalid contract validation.
  - [ ] AC: 4.

- [ ] Protect BMAD vocabulary and routing boundaries.
  - [ ] Assert no fixture introduces Archon-specific finding categories such as `intent_gap`, `bad_spec`, `reject`, `needs_archon`, or `archon_decision`.
  - [ ] Assert route gate values remain only `PASS`, `FAIL`, `CONCERNS`, and `ERROR`.
  - [ ] Assert JSON contract validation reads structured output, not the human-readable Markdown report.
  - [ ] Assert `decision_needed` fixtures do not require or emit `converted_to_patch`.
  - [ ] AC: 1, 2, 3, 4.

- [ ] Validate the repository.
  - [ ] Run the focused outcome mapping fixture tests added by this story.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json` and confirm the interactive skill still has no findings.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill has no findings.
  - [ ] Run `npm run validate:skills`.
  - [ ] Before marking the story done, run `npm run quality` on the exact checkout that will be committed.
  - [ ] AC: 1, 2, 3, 4.

## Dev Notes

### Implementation Precondition

Story M4.1 depends on Stories M1.2, M2.1, and M3.1.
[Source: _bmad-output/planning-artifacts/epics.md:323]

At story creation time, `src/bmm-skills/4-implementation/bmad-code-review-auto/` does not exist in tracked source.

M1.1, M1.2, M2.1, M2.2, M3.1, and M3.2 are currently ready for development, not completed implementation evidence.

Do not implement M4.1 until M1.2 has normalized the BMAD finding categories, M2.1 has produced the route-facing contract, and M3.1 has produced durable `decision-needed.json`.

M4.1 is validation coverage over those completed contracts.

It must not invent replacement review semantics or a separate Archon wrapper.

### Requirements Context

The epics define Epic M4 as BMAD review auto validation.
[Source: _bmad-output/planning-artifacts/epics.md:313]

The epics define this story as deterministic fixtures for automated review outcomes.
[Source: _bmad-output/planning-artifacts/epics.md:317]

The required fixture cases are `patch`, `decision_needed`, `defer`, `dismiss`, and invalid or untrusted output.
[Source: _bmad-output/planning-artifacts/epics.md:326]

The PRD requires fixtures or deterministic tests for the same outcome set.
[Source: _bmad-output/planning-artifacts/prd.md:129]

The PRD requires `patch_count > 0` and `FAIL` for a `patch` fixture.
[Source: _bmad-output/planning-artifacts/prd.md:133]

The PRD requires `decision_needed_count > 0` and `CONCERNS` when a `decision_needed` fixture has no patch findings.
[Source: _bmad-output/planning-artifacts/prd.md:134]

The PRD requires `defer_count` and `dismiss_count` to increment while preserving reasons.
[Source: _bmad-output/planning-artifacts/prd.md:135]
[Source: _bmad-output/planning-artifacts/prd.md:136]

The PRD requires invalid evidence or untrusted output to produce `ERROR`.
[Source: _bmad-output/planning-artifacts/prd.md:137]

The implementation readiness report assesses M4.1 as focused on deterministic outcome fixtures and dependent on prior contracts and persistence.
[Source: _bmad-output/planning-artifacts/implementation-readiness-report-2026-06-30.md:283]

### Architecture Guardrails

The architecture says `bmad-code-review-auto` lives in BMAD-METHOD and Archon does not own its review semantics.
[Source: _bmad-output/planning-artifacts/architecture.md:38]

The architecture requires the BMAD categories `patch`, `decision_needed`, `defer`, and `dismiss`.
[Source: _bmad-output/planning-artifacts/architecture.md:51]

The architecture defines `code-review-auto.gate.json` as the machine-readable route API.
[Source: _bmad-output/planning-artifacts/architecture.md:61]

The architecture says Markdown review reports are human evidence and Archon routes on JSON only.
[Source: _bmad-output/planning-artifacts/architecture.md:62]
[Source: _bmad-output/planning-artifacts/architecture.md:63]

The architecture requires the CR contract to include `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate`, all count fields, `blocking_findings_count`, `decision_needed_file`, `report_file`, and `story_file`.
[Source: _bmad-output/planning-artifacts/architecture.md:73]

The architecture gate rules are `FAIL` for patch findings, `CONCERNS` for decision-needed findings without patch findings, `PASS` for no patch or decision-needed findings, and `ERROR` for reviewer execution failure, invalid evidence, or untrusted output.
[Source: _bmad-output/planning-artifacts/architecture.md:92]

The architecture validation rules require fixture coverage for `patch`, `decision_needed`, `defer`, `dismiss`, and invalid or untrusted output.
[Source: _bmad-output/planning-artifacts/architecture.md:138]

The architecture validation rules require JSON contract validation to prove `PASS`, `FAIL`, `CONCERNS`, and `ERROR`.
[Source: _bmad-output/planning-artifacts/architecture.md:139]

Implementation agents must not traverse out of this repository to read parent workspace planning files.
[Source: _bmad-output/planning-artifacts/architecture.md:13]

### Fixture Contract

Fixtures should start from normalized findings or review outputs produced by the implemented `bmad-code-review-auto` pipeline.

Fixtures should avoid snapshots that make harmless Markdown wording changes fail route validation.

The routing source is `code-review-auto.gate.json`.

The human report can be asserted only for evidence and reason preservation.

The `decision_needed` fixture should also assert that `decision-needed.json` is produced or referenced consistently when M3.1 is implemented.

The invalid evidence fixture should cover malformed or missing evidence pointers that the implemented validator rejects.

The untrusted output fixture should cover output that cannot be trusted as BMAD review evidence.

Error cases should produce `ERROR` before content-derived gates are considered.

### Scope Boundaries

Do not implement the M1.2 finding model in this story.

Do not implement the M2.1 CR contract writer in this story.

Do not implement the M3.1 `decision-needed.json` writer in this story.

Do not implement Linear sync in this story.

Do not call the Linear API.

Do not create Archon DAG routing, PR creation, or `decision-needed-check`.

Do not introduce BMAD-TEA gate behavior.

Do not parse Markdown reports for route decisions.

Do not add `converted_to_patch`.

### Current Interactive Review Semantics

The current interactive review skill is `src/bmm-skills/4-implementation/bmad-code-review`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/SKILL.md:1]

Its review step launches Blind Hunter, Edge Case Hunter, and Acceptance Auditor when full review context exists.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-02-review.md:16]

Its triage step routes findings into `decision_needed`, `patch`, `defer`, and `dismiss`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:39]

Its presentation step can ask the user to resolve decision-needed findings and apply patches.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:43]
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:51]

M4.1 fixtures should validate the automated pipeline without importing interactive resolution or patch application.

### Source Files To Read Before Editing

Read the implemented automation source under:

```text
src/bmm-skills/4-implementation/bmad-code-review-auto/
```

If the directory does not exist, implementation is blocked by Story M1.1.

Read the M1.2 implementation before relying on normalized finding fields or category names.

Read the M2.1 implementation before asserting route contract field names and gate precedence.

Read the M3.1 implementation before asserting `decision-needed.json` paths or persisted decision-needed fields.

Read any fixtures or tests added by M1.2, M2.1, and M3.1 before adding the M4.1 matrix.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md` for the baseline BMAD triage vocabulary.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md` only to understand human artifact precedents and behavior that automation must not copy.

Read `tools/skill-validator.md` before adding or renaming skill step files.

Read `package.json` before relying on validation scripts.

### Skill Structure Rules

Every skill directory requires `SKILL.md`.
[Source: tools/skill-validator.md:47]

The `name` field must match the skill directory name.
[Source: tools/skill-validator.md:79]

The description must state what the skill does and when to use it.
[Source: tools/skill-validator.md:87]

Step files must use the `step-NN-description.md` naming pattern.
[Source: tools/skill-validator.md:183]

Do not reference private files inside another skill directory by path.
[Source: tools/skill-validator.md:158]

If fixture helpers are shared between skills, place the shared material outside skill directories.

### Testing Requirements

Run these checks at minimum:

```bash
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json
npm run validate:skills
npm run quality
```

Run the focused outcome mapping fixtures or tests added by this story.

`npm run quality` is the required project-level pre-push check.
[Source: package.json:43]

The deterministic skill validator is included in `quality` through `npm run validate:skills`.
[Source: package.json:52]

### Latest Technical Information

No external API or third-party library research is required for this story.

This story updates local fixtures, contract validation tests, and BMAD automation validation using the repository's existing Node toolchain.

Use the repository versions and scripts from `package.json`.

### Previous Story Intelligence

No previous Epic M4 story file exists in `_bmad-output/implementation-artifacts`.

The dependency stories M1.1, M1.2, M2.1, M2.2, M3.1, and M3.2 are currently ready for development, not done.

M1.2 establishes the review layers, normalized finding model, and BMAD triage vocabulary that this story validates.

M2.1 establishes the CR gate contract, count fields, and gate precedence that this story validates.

M3.1 establishes durable `decision-needed.json` persistence that this story validates for decision-needed fixtures.

M3.2 may add additional sync fixtures, but Linear sync is not required for M4.1 outcome mapping.

Recent commits are Archon configuration and handoff planning work, not implementation for this story.

### Project Structure Notes

This is a brownfield source and test change.

Use tracked source under `src/bmm-skills/4-implementation/` and tracked tests under `test/`.

Do not manually edit `.agents` or `_bmad` generated install output as source.

Keep fixtures deterministic and small enough to be useful as contract tests.

Keep route-facing assertions on JSON.

Keep human-readable report assertions limited to reason preservation.

## Dev Agent Record

### Agent Model Used

Not set until implementation.

### Debug Log References

### Completion Notes List

### File List
