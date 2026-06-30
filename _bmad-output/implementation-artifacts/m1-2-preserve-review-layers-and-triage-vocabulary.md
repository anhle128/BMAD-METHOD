---
baseline_commit: 3056052c880429fa90f0dfe48e95b57ad103f56d
---

# Story M1.2: Preserve Review Layers And Triage Vocabulary

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a BMAD-METHOD maintainer,
I want automated review to preserve BMAD review layers and triage,
so that automation does not weaken BMAD code-review semantics.

## Acceptance Criteria

1. Given story context exists, when automated review runs, then applicable BMAD review layers include Blind Hunter, Edge Case Hunter, and Acceptance Auditor.
2. Given findings are classified, when triage completes, then categories are `patch`, `decision_needed`, `defer`, and `dismiss`, and no Archon-specific category is introduced.
3. Given a finding requires human product, design, or operator judgment, when classification runs, then it is recorded as `decision_needed`.
4. Given a finding is fixable by development work, when classification runs, then it is recorded as `patch`.

## Tasks / Subtasks

- [x] Verify the M1.1 source baseline before implementation.
  - [x] Confirm `src/bmm-skills/4-implementation/bmad-code-review-auto/` exists.
  - [x] Confirm the automation skill validates with `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json`.
  - [x] If the source directory is missing, stop and complete Story M1.1 before implementing this story.
  - [x] Read the M1.1 story file and the actual M1.1 source files before editing.
  - [x] AC: 1, 2, 3, 4.

- [x] Preserve BMAD review layer execution in `bmad-code-review-auto`.
  - [x] Ensure automated review keeps Blind Hunter and Edge Case Hunter as review layers.
  - [x] Ensure automated review includes Acceptance Auditor when story or spec context exists.
  - [x] Ensure Acceptance Auditor receives the diff plus story or spec context and loaded context docs.
  - [x] Ensure no layer prompt says Archon owns review semantics.
  - [x] Ensure failed, timed out, or empty layers are recorded as incomplete review evidence instead of being hidden.
  - [x] AC: 1.

- [x] Implement or formalize the automated finding model.
  - [x] Normalize findings from all applicable layers into one model before triage.
  - [x] Include at least `id`, `source`, `title`, `detail`, `location`, `evidence`, `reason`, and `source_context` when those values are available.
  - [x] Preserve merged source identity when duplicate findings are merged.
  - [x] Preserve enough evidence for later report, gate, and decision-needed stories.
  - [x] Do not require later `code-review-auto.gate.json` fields in this story.
  - [x] AC: 2.

- [x] Preserve BMAD triage categories exactly.
  - [x] Classify every non-dismissed finding into exactly one of `patch`, `decision_needed`, or `defer`.
  - [x] Classify noise, false positives, or findings handled elsewhere as `dismiss`.
  - [x] Preserve the dismiss count for later contract work even if dismissed findings are not emitted as active findings.
  - [x] Reject any new route category such as `intent_gap`, `bad_spec`, `reject`, `converted_to_patch`, `needs_archon`, or `archon_decision`.
  - [x] AC: 2.

- [x] Encode the `decision_needed` rule without interactive resolution.
  - [x] Classify a finding as `decision_needed` when the correct fix requires human product, design, operator, or policy judgment.
  - [x] Do not ask the human to resolve the finding inside `bmad-code-review-auto`.
  - [x] Do not convert `decision_needed` to `patch` during automated review.
  - [x] Preserve a human-judgment reason on the finding for Story M3.1.
  - [x] AC: 3.

- [x] Encode the `patch` rule without applying patches.
  - [x] Classify a finding as `patch` when the correct development fix is unambiguous.
  - [x] Keep `patch` as a finding category and output value only.
  - [x] Do not apply code changes from inside automated review.
  - [x] Preserve enough location and evidence data for later development routing.
  - [x] AC: 4.

- [x] Add focused validation for layer and triage semantics.
  - [x] Add lightweight fixtures, examples, or tests that exercise Blind Hunter, Edge Case Hunter, and Acceptance Auditor presence when story context exists.
  - [x] Add focused fixtures, examples, or tests for `patch`, `decision_needed`, `defer`, and `dismiss`.
  - [x] Add a validation check that forbidden Archon-specific categories are not present in `bmad-code-review-auto` output vocabulary.
  - [x] Keep full route gate outcome mapping, invalid evidence handling, and untrusted output fixtures scoped to Story M4.1 unless a small helper is needed now.
  - [x] AC: 1, 2, 3, 4.

- [x] Validate the repository.
  - [x] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json` and confirm the interactive skill still has no findings.
  - [x] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill has no findings.
  - [x] Run `npm run validate:skills`.
  - [x] Run the focused tests or fixture checks added by this story.
  - [x] Before marking the story done, run `npm run quality` on the exact checkout that will be committed.
  - [x] AC: 1, 2, 3, 4.

### Review Findings

- [x] [Review][Patch] `quality` does not run the focused code-review-auto regression test [package.json:43]
- [x] [Review][Patch] Explicit file-list review does not account for untracked files [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-01-collect-inputs.md:24]
- [x] [Review][Patch] Multiple review input sources are not rejected as ambiguous [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-01-collect-inputs.md:22]
- [x] [Review][Patch] Successful zero-finding review layers can be recorded as incomplete [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-02-run-review.md:28]
- [x] [Review][Patch] `review_layers_unavailable` failure path references an unset result file [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-02-run-review.md:32]
- [x] [Review][Patch] Input collection can miss staged or deleted file-list changes [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-01-collect-inputs.md:24]
- [x] [Review][Patch] `decision_needed` rule is not bidirectional [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-02-run-review.md:64]
- [x] [Review][Patch] `patch` rule is not bidirectional [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-02-run-review.md:59]
- [x] [Review][Patch] `defer` category lacks the preserved BMAD meaning [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-02-run-review.md:58]
- [x] [Review][Patch] Uncommitted review input can omit untracked files [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-01-collect-inputs.md:28]
- [x] [Review][Patch] File-list paths are not constrained to the project root [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-01-collect-inputs.md:33]
- [x] [Review][Patch] No-spec `decision_needed` findings are not reclassified [src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-02-run-review.md:68]

## Dev Notes

### Implementation Precondition

Story M1.2 depends on Story M1.1.
[Source: _bmad-output/planning-artifacts/epics.md:169]

At story creation time, `src/bmm-skills/4-implementation/bmad-code-review-auto/` does not exist.

Do not implement M1.2 until M1.1 has created and validated the automation skill source directory.

The M1.1 story created the source boundary and warns that `.agents` and `_bmad` are ignored install outputs, not hand-authored source.
[Source: _bmad-output/implementation-artifacts/m1-1-add-bmad-code-review-auto.md]

### Requirements Context

The PRD requires `bmad-code-review-auto` to use the same review reasoning model as BMAD code review where automation permits it.
[Source: _bmad-output/planning-artifacts/prd.md:66]

The PRD requires preserving review layers and triage vocabulary.
[Source: _bmad-output/planning-artifacts/prd.md:68]

The PRD forbids Archon-specific finding categories.
[Source: _bmad-output/planning-artifacts/prd.md:70]

The PRD acceptance criteria explicitly require Blind Hunter, Edge Case Hunter, Acceptance Auditor, `patch`, `decision_needed`, `defer`, `dismiss`, the human-judgment rule, and the fixable-development-work rule.
[Source: _bmad-output/planning-artifacts/prd.md:72]

The architecture says `bmad-code-review-auto` should reuse or mirror existing BMAD review reasoning rather than create an Archon-specific wrapper.
[Source: _bmad-output/planning-artifacts/architecture.md:20]

The architecture defines the flow from story context to review layers, triage, contract, report, and decision artifacts.
[Source: _bmad-output/planning-artifacts/architecture.md:25]

The architecture requires Blind Hunter, Edge Case Hunter, and Acceptance Auditor when story context exists.
[Source: _bmad-output/planning-artifacts/architecture.md:47]

The architecture requires `patch`, `decision_needed`, `defer`, and `dismiss`.
[Source: _bmad-output/planning-artifacts/architecture.md:51]

### Current Interactive Review Semantics

The current interactive review skill is `src/bmm-skills/4-implementation/bmad-code-review`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/SKILL.md:1]

Its `SKILL.md` goal is adversarial code review using parallel review layers and structured triage.
[Source: src/bmm-skills/4-implementation/bmad-code-review/SKILL.md:6]

Its review step launches Blind Hunter, Edge Case Hunter, and Acceptance Auditor when `review_mode` is `full`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-02-review.md:16]

Its Acceptance Auditor checks the diff against the spec and context docs.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-02-review.md:28]

Its triage step normalizes each finding to `id`, `source`, `title`, `detail`, and `location`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:19]

Its triage step deduplicates related findings and preserves merged source identity.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:26]

Its triage step routes findings into `decision_needed`, `patch`, `defer`, and `dismiss`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:39]

Its current interactive presentation step writes review findings to a story file and can ask the user to resolve or apply findings.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:19]

`bmad-code-review-auto` must preserve the review and triage semantics without inheriting the interactive resolution or patch-application behavior.

### Anti-Patterns To Avoid

Do not copy route categories from `bmad-dev-auto` or `bmad-quick-dev`.

Those workflows use categories such as `intent_gap`, `bad_spec`, and `reject`.
[Source: src/bmm-skills/4-implementation/bmad-dev-auto/step-04-review.md:44]

Those categories are not BMAD code review auto categories for this PRD.

Do not add Archon ownership language to any review prompt or triage instruction.

Do not add `converted_to_patch`.

The architecture explicitly says `decision_needed` findings are not converted to patches in v2.
[Source: _bmad-output/planning-artifacts/architecture.md:65]

Do not implement full `code-review-auto.gate.json` gate rules in this story.

Story M2.1 owns the route-facing CR gate contract.
[Source: _bmad-output/planning-artifacts/epics.md:197]

Do not implement durable `decision-needed.json` persistence in this story.

Story M3.1 owns durable decision-needed persistence.
[Source: _bmad-output/planning-artifacts/epics.md:258]

Do not implement the full deterministic outcome fixture matrix in this story.

Story M4.1 owns route-facing outcome mapping fixtures.
[Source: _bmad-output/planning-artifacts/epics.md:317]

### Source Files To Read Before Editing

Read the source files created by M1.1 under:

```text
src/bmm-skills/4-implementation/bmad-code-review-auto/
```

If those files do not exist, implementation is blocked by the M1.1 dependency.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-02-review.md` as the review-layer baseline.

Current state: it defines Blind Hunter, Edge Case Hunter, and Acceptance Auditor behavior.

What this story changes: mirror or reuse the layer semantics inside automation.

What must be preserved: the interactive skill remains usable.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md` as the triage vocabulary baseline.

Current state: it normalizes, deduplicates, assigns severity, and routes into BMAD triage buckets.

What this story changes: mirror or reuse the BMAD category semantics inside automation.

What must be preserved: no Archon-specific categories are introduced.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md` before deciding what not to copy.

Current state: it contains interactive resolution and patch handling.

What this story changes: none in the interactive skill unless shared extraction is unavoidable.

What must be preserved: `bmad-code-review-auto` must not apply patches or ask interactive next-step questions.

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

If shared review semantics are extracted, place shared material outside both skill directories and reference it in a validator-compliant way.

### Testing Requirements

Run these checks at minimum:

```bash
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json
npm run validate:skills
npm run quality
```

The current `bmad-code-review` source skill returns no deterministic validator findings.

`npm run quality` is the required project-level pre-push check.
[Source: package.json:43]

The deterministic skill validator is included in `quality` through `npm run validate:skills`.
[Source: package.json:52]

### Latest Technical Information

No external API or third-party library research is required for this story.

This story updates BMAD skill instructions, source skill files, and local validation fixtures using the repository's existing toolchain.

Use the repository versions and scripts from `package.json`.

### Previous Story Intelligence

The previous story is M1.1, `m1-1-add-bmad-code-review-auto`.

It is currently `ready-for-dev`, not `done`, so no implemented M1.1 code patterns are available yet.

It established that source changes belong under `src/bmm-skills/4-implementation/` and that ignored install outputs should not be manually edited as source.

It also established that M1.2 owns full preservation of review layers and triage vocabulary.
[Source: _bmad-output/implementation-artifacts/m1-1-add-bmad-code-review-auto.md]

Recent commits are Archon configuration and handoff planning work, not implementation for this story.

### Project Structure Notes

This is a brownfield source change.

Prefer additive changes inside `bmad-code-review-auto` over modifying the existing interactive review skill.

Only edit `bmad-code-review` if a shared extraction is needed to prevent semantic drift.

Generated `_bmad` and `.agents` outputs may be useful for local smoke checks, but they are not the source contract for this repository.

## Change Log

- 2026-06-30: Preserved automated review layer and triage semantics in `bmad-code-review-auto`, and added focused tests for M1.2.

## Dev Agent Record

### Agent Model Used

Codex GPT-5.

### Debug Log References

- `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` returned no findings before implementation.
- `node test/test-bmad-code-review-auto.js` failed in red phase with four missing M1.2 semantics checks.
- `node test/test-bmad-code-review-auto.js` passed after updating `bmad-code-review-auto`.
- `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json` returned no findings.
- `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` returned no findings.
- `npm run validate:skills` passed with 47 skills scanned and no findings.
- `npm run test:code-review-auto` passed with 9 focused checks.
- `npm run test` passed.
- `npm run quality` passed.

### Completion Notes List

- Verified the M1.1 auto skill source exists and validates before changing M1.2 behavior.
- Expanded automated review instructions to preserve Blind Hunter, Edge Case Hunter, and Acceptance Auditor with loaded context docs.
- Formalized incomplete review evidence, normalized finding fields, merged source identity, and dismiss count handling.
- Preserved BMAD triage categories exactly and kept `decision_needed` and `patch` routing non-interactive.
- Extended focused tests to cover M1.2 review layer, finding model, triage vocabulary, and routing semantics.

### File List

- `src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-02-run-review.md`
- `src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-03-write-results.md`
- `test/test-bmad-code-review-auto.js`
- `_bmad-output/implementation-artifacts/m1-2-preserve-review-layers-and-triage-vocabulary.md`
- `_bmad-output/implementation-artifacts/sprint-status.yaml`
