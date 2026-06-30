# Story M2.2: Preserve Human Review Report

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a human reviewer,
I want automated review to produce a human-readable report,
so that I can inspect BMAD evidence without affecting Archon routing.

## Acceptance Criteria

1. Given automated review writes the CR contract, when the human-readable report is generated, then the report preserves source findings, triage reasons, and evidence pointers, and Archon does not need to parse it for routing.

## Tasks / Subtasks

- [ ] Verify dependency implementation before editing.
  - [ ] Confirm Story M1.1 has created `src/bmm-skills/4-implementation/bmad-code-review-auto/`.
  - [ ] Confirm Story M1.2 has implemented normalized automated findings with `patch`, `decision_needed`, `defer`, and `dismiss`.
  - [ ] Confirm Story M2.1 has implemented `code-review-auto.gate.json` with a stable `report_file` field.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill validates.
  - [ ] If any dependency source or contract behavior is missing, stop and complete the dependency story first.
  - [ ] AC: 1.

- [ ] Define the human-readable report artifact.
  - [ ] Generate a deterministic report file for each automated review run.
  - [ ] Use the `report_file` path written by the M2.1 CR contract as the report artifact path.
  - [ ] If M2.1 only created a placeholder path, replace the placeholder with the actual deterministic report path.
  - [ ] Store the report in the same automated review artifact directory as `code-review-auto.gate.json`.
  - [ ] Prefer Markdown for the first report format unless the implemented M2.1 artifact model already chose another human-readable format.
  - [ ] AC: 1.

- [ ] Populate the report from normalized review data.
  - [ ] Include run metadata: workflow, story reference, story file, node, round, generated timestamp, contract path, and gate value.
  - [ ] Include count summary fields matching the CR contract: `patch_count`, `decision_needed_count`, `defer_count`, `dismiss_count`, and `blocking_findings_count`.
  - [ ] Include review-layer status for Blind Hunter, Edge Case Hunter, Acceptance Auditor, and any failed or skipped layers.
  - [ ] Include each retained finding with id, category, severity, source layer, title, detail, location, triage reason, and evidence pointers when available.
  - [ ] Preserve merged source identity when M1.2 deduplicated related findings.
  - [ ] Include dismissed finding details only if M1.2 retained them.
  - [ ] If M1.2 retained only `dismiss_count`, report the count and do not fabricate dismissed finding details.
  - [ ] AC: 1.

- [ ] Preserve evidence without creating a routing dependency on Markdown.
  - [ ] Ensure source findings are readable enough for a human reviewer to understand what each review layer found.
  - [ ] Ensure triage reasons explain why each finding was classified as `patch`, `decision_needed`, `defer`, or `dismiss`.
  - [ ] Ensure evidence pointers refer to file paths, line references, diff context, acceptance criteria, or source context when available.
  - [ ] Ensure `decision_needed` findings show the human-judgment reason but do not ask the human to resolve the finding inside `bmad-code-review-auto`.
  - [ ] Ensure `patch` findings remain findings and do not trigger code changes.
  - [ ] Ensure Archon can route from `code-review-auto.gate.json` alone.
  - [ ] AC: 1.

- [ ] Keep report generation inside the M2 scope boundary.
  - [ ] Do not implement durable `decision-needed.json` content beyond linking to the path provided by the CR contract.
  - [ ] Do not implement Linear sync or deferred Linear issue references.
  - [ ] Do not change M2.1 gate mapping except where needed to make `report_file` point to the actual report.
  - [ ] Do not add new gate values or Archon-specific finding categories.
  - [ ] Do not parse the report to calculate the gate.
  - [ ] AC: 1.

- [ ] Validate report and contract consistency.
  - [ ] Confirm `code-review-auto.gate.json.report_file` points to the generated report.
  - [ ] Confirm the report exists before automated review reports success.
  - [ ] Confirm the report is readable and non-empty for PASS, FAIL, CONCERNS, and ERROR outcomes.
  - [ ] Confirm the report preserves the same counts and gate value that the CR contract exposes.
  - [ ] Confirm contract validation does not depend on parsing report text.
  - [ ] AC: 1.

- [ ] Add focused report fixtures or tests.
  - [ ] Add a PASS report fixture with clean review metadata and zero blocking findings.
  - [ ] Add a FAIL report fixture that preserves a `patch` finding, triage reason, and evidence pointer.
  - [ ] Add a CONCERNS report fixture that preserves a `decision_needed` finding and human-judgment reason.
  - [ ] Add an ERROR report fixture that preserves review execution, invalid evidence, or untrusted-output diagnostics.
  - [ ] Reuse M2.1 gate fixtures where practical so report and contract assertions stay aligned.
  - [ ] Keep the broader full outcome matrix scoped to Story M4.1.
  - [ ] AC: 1.

- [ ] Validate the repository.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json` and confirm the interactive skill still has no findings.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill has no findings.
  - [ ] Run the focused report fixture or snapshot checks added by this story.
  - [ ] Run `npm run validate:skills`.
  - [ ] Before marking the story done, run `npm run quality` on the exact checkout that will be committed.
  - [ ] AC: 1.

## Dev Notes

### Implementation Precondition

Story M2.2 depends on Story M2.1.
[Source: _bmad-output/planning-artifacts/epics.md:242]

At story creation time, `src/bmm-skills/4-implementation/bmad-code-review-auto/` does not exist in tracked source.

M1.1, M1.2, and M2.1 are currently ready for development, not completed implementation evidence.

Do not implement M2.2 until M2.1 has produced a CR contract with a stable `report_file` field.

The M2.1 story explicitly leaves full human-readable report content to this story.
[Source: _bmad-output/implementation-artifacts/m2-1-emit-code-review-auto-gate-json.md]

### Requirements Context

The epics define this story as human-readable review evidence for BMAD automated code review.
[Source: _bmad-output/planning-artifacts/epics.md:234]

The report must preserve source findings, triage reasons, and evidence pointers.
[Source: _bmad-output/planning-artifacts/epics.md:249]

The CR contract must include a review report path through `report_file`.
[Source: _bmad-output/planning-artifacts/epics.md:243]

The PRD says BMAD-METHOD must emit a stable machine-readable CR gate contract and human-readable artifacts for review.
[Source: _bmad-output/planning-artifacts/prd.md:24]

The PRD includes human-readable review evidence in BMAD-METHOD-owned scope.
[Source: _bmad-output/planning-artifacts/prd.md:40]

The architecture says the route API is versioned JSON and the human report remains a human evidence surface.
[Source: _bmad-output/planning-artifacts/architecture.md:22]

The architecture says `code-review-auto.gate.json` is the machine-readable route API.
[Source: _bmad-output/planning-artifacts/architecture.md:61]

The architecture says Markdown review reports are human evidence and may evolve.
[Source: _bmad-output/planning-artifacts/architecture.md:62]

The architecture says Archon routes on JSON only.
[Source: _bmad-output/planning-artifacts/architecture.md:63]

### Report Content Contract

The report is evidence for humans, not a second route API.

The report should be deterministic enough for tests and PR handoff links.

The report should include enough metadata for a human to connect it to the matching `code-review-auto.gate.json`.

The report should preserve review-layer output after M1.2 normalization and deduplication.

The report should preserve triage reasons so a human can understand why a finding is `patch`, `decision_needed`, `defer`, or `dismiss`.

The report should preserve evidence pointers instead of copying large unrelated source content.

The report should surface failed, skipped, invalid, or untrusted review evidence clearly for human diagnosis.

### Scope Boundaries

Story M2.1 owns gate contract emission and gate mapping.
[Source: _bmad-output/planning-artifacts/epics.md:197]

Story M3.1 owns durable `decision-needed.json` content.
[Source: _bmad-output/planning-artifacts/epics.md:258]

Story M3.2 owns Linear reference sync into BMAD artifacts.
[Source: _bmad-output/planning-artifacts/epics.md:282]

Story M4.1 owns the broader deterministic outcome mapping fixture matrix.
[Source: _bmad-output/planning-artifacts/epics.md:317]

This story may add focused report fixtures for PASS, FAIL, CONCERNS, and ERROR, but should not absorb all M4.1 fixture scope.

Do not implement BMAD-TEA gate behavior in this story.

Do not require implementation agents to traverse outside this repository for parent workspace planning files.
[Source: _bmad-output/planning-artifacts/prd.md:13]

### Current Interactive Review Semantics

The current interactive review skill is `src/bmm-skills/4-implementation/bmad-code-review`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/SKILL.md:1]

Its triage step normalizes findings to id, source, title, detail, and location.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:19]

Its triage step deduplicates related findings and preserves merged source identity.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:26]

Its triage step routes findings into `decision_needed`, `patch`, `defer`, and `dismiss`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:39]

Its presentation step writes human-readable findings to the story file and deferred-work file, then offers interactive resolution and patch handling.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:19]

`bmad-code-review-auto` should preserve evidence presentation without inheriting interactive resolution or patch application.

### Source Files To Read Before Editing

Read the implemented automation source under:

```text
src/bmm-skills/4-implementation/bmad-code-review-auto/
```

If the directory does not exist, implementation is blocked by Story M1.1.

Read the M2.1 implementation before adding report generation.

The `report_file` value in `code-review-auto.gate.json` is the integration point for this story.

Read any M2.1 contract tests or fixtures before adding report tests.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md` for finding fields and triage behavior.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md` only as a human-evidence precedent, not as automation behavior to copy wholesale.

Read `tools/skill-validator.md` before adding or renaming skill step files.

Read `package.json` before relying on validation scripts.

### Skill Structure Rules

Every skill directory requires `SKILL.md`.
[Source: tools/skill-validator.md:337]

The `name` field must match the skill directory name.
[Source: tools/skill-validator.md:75]

The description must state what the skill does and when to use it.
[Source: tools/skill-validator.md:83]

Step files must use the `step-NN-description.md` naming pattern.
[Source: tools/skill-validator.md:179]

Do not reference private files inside another skill directory by path.
[Source: tools/skill-validator.md:154]

If report helpers are shared between skills, place the shared material outside both skill directories.

### Testing Requirements

Run these checks at minimum:

```bash
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json
npm run validate:skills
npm run quality
```

Run the focused report fixture or snapshot checks added by this story.

`npm run quality` is the required project-level pre-push check.
[Source: package.json:43]

The deterministic skill validator is included in `quality` through `npm run validate:skills`.
[Source: package.json:52]

### Latest Technical Information

No external API or third-party library research is required for this story.

This story updates BMAD skill instructions, review report output, and local validation fixtures using the repository's existing Node toolchain.

Use the repository versions and scripts from `package.json`.

### Previous Story Intelligence

The previous Epic M2 story is M2.1, `m2-1-emit-code-review-auto-gate-json`.

It is currently ready for development, not done, so no implemented M2.1 code patterns are available yet.

It established that `code-review-auto.gate.json` is the route contract and that Markdown report content must not be the routing source.

It also established that `report_file` points to the human-readable report path and that this story owns the full report content.

Recent commits are Archon configuration and handoff planning work, not implementation for this story.

### Project Structure Notes

This is a brownfield source change.

Use tracked source under `src/bmm-skills/4-implementation/`.

Do not manually edit `.agents` or `_bmad` generated install output as source.

Keep the human report readable for review while keeping JSON as the only route contract.

## Dev Agent Record

### Agent Model Used

Not set until implementation.

### Debug Log References

### Completion Notes List

### File List
