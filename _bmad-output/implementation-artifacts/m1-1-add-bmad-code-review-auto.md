---
baseline_commit: 3056052c880429fa90f0dfe48e95b57ad103f56d
---

# Story M1.1: Add `bmad-code-review-auto`

Status: review

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a BMAD-METHOD maintainer,
I want `bmad-code-review-auto` to exist as a BMAD-owned automation surface,
so that Archon can invoke code review without a wrapper that reinterprets BMAD findings.

## Acceptance Criteria

1. Given BMAD-METHOD contains interactive `bmad-code-review`, when `bmad-code-review-auto` is added, then both surfaces are available and the interactive surface remains usable.
2. Given automation review runs, when findings are produced, then no code patch is applied and no interactive next-step choice is offered or executed.

## Tasks / Subtasks

- [x] Add the tracked source skill directory for `bmad-code-review-auto`.
  - [x] Create `src/bmm-skills/4-implementation/bmad-code-review-auto/SKILL.md`.
  - [x] Add `src/bmm-skills/4-implementation/bmad-code-review-auto/customize.toml`.
  - [x] Add step files under `src/bmm-skills/4-implementation/bmad-code-review-auto/steps/`.
  - [x] Ensure `SKILL.md` frontmatter has `name: bmad-code-review-auto` and a description that says what it does and when to use it.
  - [x] Keep `SKILL.md` body focused and push detailed execution into step files if the body would become large.
  - [x] Do not manually create or edit `.agents/skills/bmad-code-review-auto` or `_bmad/bmm/4-implementation/bmad-code-review-auto` as source.
  - [x] If generated install outputs are needed for local smoke testing, create them through the installer path and do not treat them as hand-authored source.
  - [x] AC: 1.

- [x] Register the new automation surface in the BMM help catalog.
  - [x] Update `src/bmm-skills/module-help.csv` with a `bmad-code-review-auto` row in phase `4-implementation`.
  - [x] Use a distinct menu code, such as `CRA`, unless an existing project convention suggests a better unused code.
  - [x] Make the row describe automation behavior clearly: writes findings and contracts only, does not apply patches, and does not present interactive choices.
  - [x] Preserve the existing `bmad-code-review` row and sequencing.
  - [x] AC: 1.

- [x] Define the non-interactive automation workflow.
  - [x] Use the existing `bmad-code-review` workflow as the semantic baseline, not as a wrapper whose interactive steps are called directly.
  - [x] Gather review target, story context, and diff inputs without presenting menus or waiting for user choices.
  - [x] If required inputs are missing or invalid, end with a structured failure result instead of asking follow-up questions.
  - [x] Preserve the ability to run Blind Hunter, Edge Case Hunter, and Acceptance Auditor where context allows, but do not complete Story M1.2 triage expansion in this story unless the implementation naturally requires shared scaffolding.
  - [x] If subagents are unavailable, return a structured automation failure instead of writing prompt files and asking the human to run them.
  - [x] AC: 2.

- [x] Prevent automated patch application and interactive next-step behavior.
  - [x] Ensure the new automation workflow never instructs the agent to apply patches.
  - [x] Ensure it never offers choices such as apply every patch, leave as action items, walk through each patch, continue, approve, or select a numbered menu item.
  - [x] Ensure `patch` findings remain findings or output records for later handling by downstream workflow routing.
  - [x] Ensure `decision_needed` findings remain findings or output records and are not converted into a patch path in this story.
  - [x] AC: 2.

- [x] Preserve the interactive `bmad-code-review` surface.
  - [x] Do not rename, remove, or change the trigger behavior of `src/bmm-skills/4-implementation/bmad-code-review/SKILL.md`.
  - [x] Do not remove the current interactive review step files.
  - [x] If implementation introduces shared reusable text or helpers, extract them to a shared location outside either skill directory.
  - [x] Do not reference private files inside `bmad-code-review` from `bmad-code-review-auto` by path.
  - [x] AC: 1.

- [x] Add targeted validation coverage for both review surfaces.
  - [x] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json` and confirm it still returns no findings.
  - [x] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm it returns no findings.
  - [x] Run `npm run validate:skills`.
  - [x] Run `npm run test:install` or a narrower installer manifest test that proves both skill entrypoints are discoverable from `src/bmm-skills`.
  - [x] Before marking the story done, run `npm run quality` on the exact checkout that will be committed.
  - [x] AC: 1, 2.

## Dev Notes

### Source Boundary

The tracked source for BMM implementation skills is under `src/bmm-skills/4-implementation/`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/SKILL.md:1]

The installed `_bmad` tree and `.agents` skill tree are ignored local install outputs in this checkout.
[Source: .gitignore:49]
[Source: .gitignore:56]

Treat `src/bmm-skills/4-implementation/bmad-code-review-auto/` as the source of truth for the new surface.

Do not manually modify generated or ignored install outputs to satisfy acceptance criteria.

The installer and manifest generator discover skills by recursively finding source directories that contain a valid `SKILL.md` whose frontmatter name matches the directory name.
[Source: tools/installer/core/manifest-generator.js:104]

The skill manifest writer emits installed skill manifest rows from that discovered skill set.
[Source: tools/installer/core/manifest-generator.js:401]

### Existing Interactive Review Surface

The current interactive review surface is `bmad-code-review`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/SKILL.md:1]

It uses step-file architecture and explicitly halts at checkpoints and waits for human input.
[Source: src/bmm-skills/4-implementation/bmad-code-review/SKILL.md:75]

It launches Blind Hunter, Edge Case Hunter, and Acceptance Auditor when review context allows.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-02-review.md:16]

It normalizes findings and routes them into `decision_needed`, `patch`, `defer`, and `dismiss`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:39]

It currently offers interactive choices for decision-needed and patch findings.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:43]
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:51]

It can apply every patch when the human chooses that option.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:70]

`bmad-code-review-auto` must not call or copy that interactive action behavior into the automation flow.

### Automation Pattern

The repository already has unattended workflow precedent in `bmad-dev-auto`.
[Source: src/bmm-skills/4-implementation/bmad-dev-auto/SKILL.md:1]

That workflow defines a HALT result path and explicitly targets no-human-interaction execution.
[Source: src/bmm-skills/4-implementation/bmad-dev-auto/SKILL.md:6]

Use that style as the automation posture for failure handling and final status reporting.

Do not copy its categories into code review semantics.

Code review semantics remain owned by `bmad-code-review` and the PRD for this feature.

### Requirements Context

The PRD requires `bmad-code-review-auto` as an automation-native BMAD surface.
[Source: _bmad-output/planning-artifacts/prd.md:21]

The PRD requires the interactive `bmad-code-review` surface to remain available.
[Source: _bmad-output/planning-artifacts/prd.md:26]

The PRD makes automation additive unless a local BMAD-METHOD decision proves a shared implementation path is safer.
[Source: _bmad-output/planning-artifacts/prd.md:27]

The PRD requires automated review not to apply code patches and not to offer or execute interactive next-step choices.
[Source: _bmad-output/planning-artifacts/prd.md:79]

The architecture says the new surface should reuse or mirror existing BMAD review reasoning rather than creating an Archon-specific wrapper.
[Source: _bmad-output/planning-artifacts/architecture.md:20]

The architecture says `bmad-code-review-auto` lives in BMAD-METHOD and Archon does not own its semantics.
[Source: _bmad-output/planning-artifacts/architecture.md:41]

The architecture says automated review writes findings, reports, and contracts, and that development fixes happen later through route back to `dev-story`.
[Source: _bmad-output/planning-artifacts/architecture.md:53]

### Story Scope Boundaries

This story creates the automation surface and enforces no-patch, no-menu behavior.

Story M1.2 owns full preservation of review layers and triage vocabulary as explicit behavior.
[Source: _bmad-output/planning-artifacts/epics.md:161]

Story M2.1 owns the full `code-review-auto.gate.json` contract.
[Source: _bmad-output/planning-artifacts/architecture.md:71]

Story M3.1 owns durable `decision-needed.json` persistence.
[Source: _bmad-output/planning-artifacts/architecture.md:65]

Do not overbuild these later-story contracts in this story unless a small scaffold is necessary to keep the new skill coherent.

If a scaffold is added, mark it clearly as a stable extension point for later stories.

### File Structure Requirements

Create new source files under:

```text
src/bmm-skills/4-implementation/bmad-code-review-auto/
```

Expected initial files:

```text
src/bmm-skills/4-implementation/bmad-code-review-auto/SKILL.md
src/bmm-skills/4-implementation/bmad-code-review-auto/customize.toml
src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-01-*.md
src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-02-*.md
```

Use the exact step filename pattern enforced by the validator: `step-NN-description.md`.
[Source: tools/skill-validator.md:183]

Do not reference another skill directory by private file path.

If sharing is required, extract shared material into an appropriate shared source location outside both skill directories.
[Source: tools/skill-validator.md:153]

### Update Files To Read Before Editing

Read `src/bmm-skills/module-help.csv` before editing.

Current state: it contains the implementation workflow catalog rows, including `bmad-code-review`.
[Source: src/bmm-skills/module-help.csv:24]

What this story changes: add a `bmad-code-review-auto` row without changing the existing interactive row.

What must be preserved: the current `bmad-code-review` row and the standard CSV schema.

Read `src/bmm-skills/4-implementation/bmad-code-review/SKILL.md` and its steps before deciding whether to duplicate or extract shared material.

Current state: the interactive skill is valid and currently returns no deterministic validator findings.

What this story changes: ideally nothing in the existing interactive skill.

What must be preserved: trigger semantics, interactive checkpoints, step-file sequence, and user-facing review behavior.

Read `tools/skill-validator.md` and `tools/validate-skills.js` before adding the new skill.

Current state: deterministic validation checks `SKILL.md` metadata, directory name matching, step file names, step counts, and time estimates.
[Source: tools/validate-skills.js:1]

What this story changes: no validator changes are expected unless validation reveals a real gap.

What must be preserved: validation should keep passing for all existing skills.

### Testing Requirements

Run these checks at minimum:

```bash
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json
npm run validate:skills
npm run test:install
npm run quality
```

`npm run quality` is the required project-level pre-push check.
[Source: package.json:43]

The deterministic skill validator is included in `quality` through `npm run validate:skills`.
[Source: package.json:52]

### Latest Technical Information

No external API or third-party library research is required for this story.

The story adds BMAD skill source files and a BMM catalog row using the repository's existing Node and validation toolchain.

Use the repository versions and scripts from `package.json`.

### Previous Story Intelligence

No previous Epic M1 story file exists in `_bmad-output/implementation-artifacts`.

No previous-story implementation patterns are available for this epic.

Recent commits are Archon configuration and handoff planning work, not implementation for this story.

### Project Structure Notes

This is a brownfield source change.

Prefer additive source files over mutating existing review behavior.

Generated `_bmad` and `.agents` outputs may be useful for local smoke checks, but they are not the source contract for this repository.

## Dev Agent Record

### Agent Model Used

Codex GPT-5.

### Debug Log References

- `node test/test-bmad-code-review-auto.js` failed before implementation, then passed after the new auto skill and catalog row were added.
- `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json` returned no findings.
- `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` returned no findings.
- `npm run validate:skills` passed with 47 skills scanned and no findings.
- `npm run test:install` passed with 379 installation component tests.
- `npm run test` passed.
- `npm run docs:build` passed after correcting the Starlight autogenerated sidebar group shape.
- `npm run quality` passed.

### Completion Notes List

- Added `bmad-code-review-auto` as a source BMAD skill with a non-interactive three-step workflow.
- Registered the new `CRA` help catalog row while preserving the existing interactive `CR` row.
- Kept automation outputs to findings and structured results only, with explicit no-patch and no-interactive-choice behavior.
- Added a focused regression test and npm script for both review surfaces.
- Corrected quality blockers in local tooling ignores and the website Starlight sidebar config encountered during the required full quality gate.

### File List

- `.markdownlint-cli2.yaml`
- `.prettierignore`
- `package.json`
- `src/bmm-skills/4-implementation/bmad-code-review-auto/SKILL.md`
- `src/bmm-skills/4-implementation/bmad-code-review-auto/customize.toml`
- `src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-01-collect-inputs.md`
- `src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-02-run-review.md`
- `src/bmm-skills/4-implementation/bmad-code-review-auto/steps/step-03-write-results.md`
- `src/bmm-skills/module-help.csv`
- `test/test-bmad-code-review-auto.js`
- `website/astro.config.mjs`
