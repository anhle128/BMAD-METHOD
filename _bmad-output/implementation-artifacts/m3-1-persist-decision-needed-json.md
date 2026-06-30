# Story M3.1: Persist `decision-needed.json`

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a workflow operator,
I want decision-needed findings persisted in a durable BMAD artifact,
so that human-judgment items remain visible after automated review.

## Acceptance Criteria

1. Given automated review identifies `decision_needed`, when artifacts are written, then `decision-needed.json` records finding id, story reference, source gate, title, detail, evidence pointers, human-judgment reason, status, and timestamps.
2. Given v2 status values are written, when the artifact is validated, then `converted_to_patch` is not required.

## Tasks / Subtasks

- [ ] Verify dependency implementation before editing.
  - [ ] Confirm Story M1.1 has created `src/bmm-skills/4-implementation/bmad-code-review-auto/`.
  - [ ] Confirm Story M1.2 has implemented normalized automated findings with `patch`, `decision_needed`, `defer`, and `dismiss`.
  - [ ] Confirm Story M2.1 has implemented `code-review-auto.gate.json` with `decision_needed_count`, `decision_needed_file`, `gate`, `story_ref`, and `story_file`.
  - [ ] Confirm Story M2.2 has not made Markdown report parsing part of routing.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill validates.
  - [ ] If dependency source or contract behavior is missing, stop and complete the dependency story first.
  - [ ] AC: 1, 2.

- [ ] Define the durable `decision-needed.json` artifact.
  - [ ] Write `decision-needed.json` in the deterministic automated review artifact directory established by M2.1.
  - [ ] Use the same path that `code-review-auto.gate.json.decision_needed_file` exposes when `decision_needed_count` is greater than zero.
  - [ ] If M2.1 only created a placeholder path, replace the placeholder with the actual deterministic artifact path.
  - [ ] Write the artifact whenever one or more `decision_needed` findings exist, including cases where `patch` findings also make the source gate `FAIL`.
  - [ ] Keep `decision_needed_file` null or absent according to the M2.1 contract when no decision-needed findings exist.
  - [ ] AC: 1.

- [ ] Define the JSON schema and stable field names.
  - [ ] Include top-level metadata for `artifact_version`, `workflow`, `story_ref`, `story_file`, `source_contract_file`, `source_gate`, `decision_needed_count`, and `generated_at`.
  - [ ] Include a `findings` array.
  - [ ] For each finding, include `finding_id`, `story_ref`, `story_file`, `source_gate`, `title`, `detail`, `evidence`, `human_judgment_reason`, `status`, `created_at`, and `updated_at`.
  - [ ] Preserve source layer, source context, location, severity, and triage reason if M1.2 normalized findings expose those fields.
  - [ ] Keep `evidence` structured as an array or object rather than string-concatenating file paths and line references.
  - [ ] Preserve enough data for later report review and M3.2 Linear sync without forcing M3.2 to parse Markdown.
  - [ ] AC: 1.

- [ ] Define v2 status behavior without patch conversion.
  - [ ] Persist newly written decision-needed findings with status `unresolved`.
  - [ ] Allow the schema to support a future `deferred_to_linear` status for Story M3.2.
  - [ ] Do not require or emit `converted_to_patch`.
  - [ ] Reject any implementation that treats `converted_to_patch` as mandatory for v2 validation.
  - [ ] Do not ask the human to resolve the finding inside `bmad-code-review-auto`.
  - [ ] Do not convert `decision_needed` findings into `patch` findings during persistence.
  - [ ] AC: 2.

- [ ] Keep CR contract and decision-needed artifact consistent.
  - [ ] If `decision_needed_count` is greater than zero, confirm `code-review-auto.gate.json.decision_needed_file` points to the generated `decision-needed.json`.
  - [ ] Confirm `decision_needed_count` in the CR contract matches the number of persisted findings.
  - [ ] Confirm each persisted finding has `source_gate` equal to the final CR contract gate for that review run.
  - [ ] Confirm `story_ref` and `story_file` match between the CR contract and `decision-needed.json`.
  - [ ] Ensure contract validation does not depend on parsing the human-readable report.
  - [ ] AC: 1.

- [ ] Validate the artifact before reporting success.
  - [ ] Parse `decision-needed.json` after writing.
  - [ ] Reject missing required top-level fields.
  - [ ] Reject missing required finding fields.
  - [ ] Reject invalid timestamps.
  - [ ] Reject non-array or malformed evidence pointers.
  - [ ] Reject status values outside the v2 set supported by this artifact.
  - [ ] Treat invalid persistence as an `ERROR` condition in the automated review result rather than silently reporting success.
  - [ ] AC: 1, 2.

- [ ] Keep M3.2 and later work out of scope.
  - [ ] Do not create Linear issues.
  - [ ] Do not implement Linear API adapters.
  - [ ] Do not sync Linear ids or URLs into story Review Findings, decision logs, deferred-work tracking, or `decision-needed.json`.
  - [ ] Do not implement the Archon decision-needed-check node.
  - [ ] Do not implement the full M4.1 fixture matrix beyond focused fixtures needed for this persistence story.
  - [ ] AC: 1, 2.

- [ ] Add focused persistence fixtures or tests.
  - [ ] Add a fixture with one `decision_needed` finding and no `patch` findings.
  - [ ] Assert the CR contract gate can be `CONCERNS`, `decision_needed_count` is greater than zero, and `decision_needed_file` points to `decision-needed.json`.
  - [ ] Assert the artifact includes finding id, story reference, source gate, title, detail, evidence pointers, human-judgment reason, status, and timestamps.
  - [ ] Add a fixture with both `patch` and `decision_needed` findings to prove `decision_needed` findings are still persisted when the source gate is `FAIL`.
  - [ ] Add a validation fixture proving `converted_to_patch` is not required.
  - [ ] AC: 1, 2.

- [ ] Validate the repository.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json` and confirm the interactive skill still has no findings.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill has no findings.
  - [ ] Run the focused persistence fixtures or tests added by this story.
  - [ ] Run `npm run validate:skills`.
  - [ ] Before marking the story done, run `npm run quality` on the exact checkout that will be committed.
  - [ ] AC: 1, 2.

## Dev Notes

### Implementation Precondition

Story M3.1 depends on Story M2.1.
[Source: _bmad-output/planning-artifacts/epics.md:266]

At story creation time, `src/bmm-skills/4-implementation/bmad-code-review-auto/` does not exist in tracked source.

M1.1, M1.2, M2.1, and M2.2 are currently ready for development, not completed implementation evidence.

Do not implement M3.1 until M2.1 has produced `code-review-auto.gate.json` with a stable `decision_needed_file` field.

The M2.1 story explicitly leaves durable `decision-needed.json` content to this story.
[Source: _bmad-output/implementation-artifacts/m2-1-emit-code-review-auto-gate-json.md]

### Requirements Context

The epics define this story as durable `decision-needed.json` persistence.
[Source: _bmad-output/planning-artifacts/epics.md:258]

The artifact blocks Archon Story A5.1 because Archon cannot create Linear follow-up from BMAD findings until the artifact exists.
[Source: _bmad-output/planning-artifacts/epics.md:268]

The epics require unresolved decision-needed findings to be written and referenced from the CR contract.
[Source: _bmad-output/planning-artifacts/epics.md:269]

The PRD requires `decision_needed` findings to be persisted in `decision-needed.json`.
[Source: _bmad-output/planning-artifacts/prd.md:105]

The PRD requires the artifact to be durable and readable by Archon `decision-needed-check`.
[Source: _bmad-output/planning-artifacts/prd.md:106]

The PRD says the artifact must not require a `converted_to_patch` path for v2.
[Source: _bmad-output/planning-artifacts/prd.md:107]

The PRD requires finding id, story reference, source gate, title, detail, evidence pointers, human-judgment reason, status, and timestamps.
[Source: _bmad-output/planning-artifacts/prd.md:111]

The architecture says `decision_needed` findings are written to `decision-needed.json` and are not converted to patches in v2.
[Source: _bmad-output/planning-artifacts/architecture.md:67]

The architecture says `decision-needed.json` should be durable and stable enough for Archon `decision-needed-check`.
[Source: _bmad-output/planning-artifacts/architecture.md:99]

The architecture lists durable artifact fields and reserves Linear id and URL for synced findings.
[Source: _bmad-output/planning-artifacts/architecture.md:100]

### Artifact Contract

`decision-needed.json` is a durable BMAD artifact for human-judgment findings.

It should be generated from normalized `decision_needed` findings, not from a Markdown report.

It should be linked by `code-review-auto.gate.json.decision_needed_file` when one or more decision-needed findings exist.

It should preserve evidence pointers as structured data.

It should preserve the human-judgment reason supplied by M1.2 triage.

It should remain readable by Archon without requiring Archon to reinterpret BMAD finding categories.

It should not include Archon-specific categories.

It should not require `converted_to_patch`.

### Status Rules

New M3.1 findings should be written as `unresolved`.

Story M3.2 may later update synced findings to `deferred_to_linear` when Archon supplies Linear references.

`converted_to_patch` is out of scope for v2.

Do not model decision-needed findings as pending patches.

### Scope Boundaries

Story M2.1 owns gate contract emission and gate mapping.
[Source: _bmad-output/planning-artifacts/epics.md:197]

Story M2.2 owns the human-readable report.
[Source: _bmad-output/planning-artifacts/epics.md:234]

Story M3.2 owns Linear reference sync into BMAD artifacts.
[Source: _bmad-output/planning-artifacts/epics.md:282]

Story M4.1 owns the broader deterministic outcome fixture matrix.
[Source: _bmad-output/planning-artifacts/epics.md:317]

This story may add focused persistence fixtures for `decision_needed` and `converted_to_patch` exclusion.

Do not implement BMAD-TEA gate behavior in this story.

Do not require implementation agents to traverse outside this repository for parent workspace planning files.
[Source: _bmad-output/planning-artifacts/prd.md:13]

### Current Interactive Review Semantics

The current interactive review skill is `src/bmm-skills/4-implementation/bmad-code-review`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/SKILL.md:1]

Its triage step normalizes findings to id, source, title, detail, and location.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:19]

Its triage step routes findings into `decision_needed`, `patch`, `defer`, and `dismiss`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md:39]

Its interactive presentation step can ask the user to resolve decision-needed findings.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:43]

`bmad-code-review-auto` must persist decision-needed findings without asking the user to resolve them.

### Source Files To Read Before Editing

Read the implemented automation source under:

```text
src/bmm-skills/4-implementation/bmad-code-review-auto/
```

If the directory does not exist, implementation is blocked by Story M1.1.

Read the M1.2 implementation before relying on normalized finding fields.

Read the M2.1 implementation before writing or validating `decision_needed_file`.

Read any M2.1 contract tests or fixtures before adding persistence tests.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md` for finding fields and triage behavior.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md` only to preserve the human-judgment concept without copying interactive resolution behavior.

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

If persistence helpers are shared between skills, place the shared material outside skill directories.

### Testing Requirements

Run these checks at minimum:

```bash
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json
npm run validate:skills
npm run quality
```

Run the focused persistence fixture or contract checks added by this story.

`npm run quality` is the required project-level pre-push check.
[Source: package.json:43]

The deterministic skill validator is included in `quality` through `npm run validate:skills`.
[Source: package.json:52]

### Latest Technical Information

No external API or third-party library research is required for this story.

This story updates BMAD skill instructions, JSON persistence output, and local validation fixtures using the repository's existing Node toolchain.

Use the repository versions and scripts from `package.json`.

### Previous Story Intelligence

No previous Epic M3 story file exists in `_bmad-output/implementation-artifacts`.

The dependency stories M1.1, M1.2, M2.1, and M2.2 are currently ready for development, not done.

M2.1 established that `decision_needed_file` points to `decision-needed.json` when `decision_needed_count` is greater than zero.

M2.2 established that Markdown reports are human evidence and must not become the routing source.

Recent commits are Archon configuration and handoff planning work, not implementation for this story.

### Project Structure Notes

This is a brownfield source change.

Use tracked source under `src/bmm-skills/4-implementation/`.

Do not manually edit `.agents` or `_bmad` generated install output as source.

Keep `decision-needed.json` structured and deterministic so later Linear sync can update it without text parsing.

## Dev Agent Record

### Agent Model Used

Not set until implementation.

### Debug Log References

### Completion Notes List

### File List
