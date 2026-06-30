# Story M3.2: Sync Linear References Into BMAD Artifacts

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a BMAD-METHOD maintainer,
I want Linear issue references synced into BMAD artifacts,
so that deferred decisions are visible from the story and review trail.

## Acceptance Criteria

1. Given a sync request supplies a Linear issue id, Linear URL, finding id, and story reference, when sync runs, then `decision-needed.json` records the issue reference and deferred status.
2. Given the story file is updated, when sync completes, then Review Findings include source gate, finding id, Linear id, Linear URL, and deferred status.
3. Given any sync target fails, when sync output is emitted, then the result is `ERROR`.
4. Given a sync request is missing required Linear reference fields, when sync validates the request, then the result is `ERROR` and no partial BMAD artifact update is reported as successful.

## Tasks / Subtasks

- [ ] Verify dependency and input-contract baseline before editing.
  - [ ] Confirm Story M1.1 has created `src/bmm-skills/4-implementation/bmad-code-review-auto/`.
  - [ ] Confirm Story M2.1 has implemented `code-review-auto.gate.json` and exposes `story_ref`, `story_file`, `gate`, and `decision_needed_file`.
  - [ ] Confirm Story M3.1 has implemented durable `decision-needed.json` with finding ids, source gate, story reference, status, and timestamps.
  - [ ] Confirm the Archon decision-needed issue-reference input contract is available from the integration context or fixtures.
  - [ ] If the Archon input contract is absent, define only a local BMAD fixture shape for tests and document that production invocation waits on the external Archon contract.
  - [ ] If dependency source or artifact behavior is missing, stop and complete the dependency story first.
  - [ ] AC: 1, 2, 3, 4.

- [ ] Define the sync request contract accepted by BMAD-METHOD.
  - [ ] Require `linear_issue_id`.
  - [ ] Require `linear_url`.
  - [ ] Require `finding_id`.
  - [ ] Require `story_ref`.
  - [ ] Require `sync_targets`.
  - [ ] Require or derive `story_file` from the CR contract or `decision-needed.json`.
  - [ ] Reject missing, empty, or type-invalid required fields before any artifact write.
  - [ ] Validate that `sync_targets` can only name BMAD-owned targets supported by this story.
  - [ ] Do not create or reuse Linear issues from this repository.
  - [ ] AC: 3, 4.

- [ ] Define deterministic sync output.
  - [ ] Emit a sync output object or file that includes `result`, `story_ref`, `finding_id`, `linear_issue_id`, `linear_url`, `updated_targets`, `failed_targets`, `error`, and `synced_at`.
  - [ ] Use `result: "OK"` only when every requested sync target was updated and validated.
  - [ ] Use `result: "ERROR"` when validation fails, a target write fails, a target validation fails, or target state is inconsistent.
  - [ ] Include enough failed-target detail for Archon PR preparation to stop and report the cause.
  - [ ] Do not report partial success as success.
  - [ ] AC: 3, 4.

- [ ] Update `decision-needed.json` idempotently.
  - [ ] Locate the persisted finding by `finding_id` and `story_ref`.
  - [ ] Record `linear_issue_id`.
  - [ ] Record `linear_url`.
  - [ ] Update status to `deferred_to_linear`.
  - [ ] Update `updated_at` and preserve original `created_at`.
  - [ ] Preserve original title, detail, source gate, evidence pointers, and human-judgment reason.
  - [ ] Treat missing matching finding, duplicate matching finding, malformed artifact, or failed validation as `ERROR`.
  - [ ] AC: 1, 3, 4.

- [ ] Update story Review Findings idempotently.
  - [ ] Locate the story file from `story_file` or the M3.1 artifact.
  - [ ] Locate the matching Review Finding by finding id or stable review marker.
  - [ ] Record source gate.
  - [ ] Record finding id.
  - [ ] Record Linear id.
  - [ ] Record Linear URL.
  - [ ] Record deferred status.
  - [ ] Preserve existing finding title, detail, and evidence.
  - [ ] Do not duplicate entries on repeated sync runs for the same finding and Linear issue.
  - [ ] Treat missing story file, missing Review Findings section, missing finding, or ambiguous matching as `ERROR` unless the implemented M3.1 story defines a safe creation path.
  - [ ] AC: 2, 3, 4.

- [ ] Update the decision log idempotently.
  - [ ] Use the existing decision log artifact if M3.1 or the implemented automation source defines one.
  - [ ] If no decision log artifact exists, create a deterministic BMAD-owned decision log under the automated review artifact directory.
  - [ ] Prefer structured JSON for a new decision log so future syncs do not parse prose.
  - [ ] Record story reference, finding id, source gate, Linear id, Linear URL, deferred status, original finding detail, and evidence pointers.
  - [ ] Preserve original finding details from `decision-needed.json`.
  - [ ] Do not duplicate log records on repeated sync runs for the same finding and Linear issue.
  - [ ] Treat failed decision-log write or validation as `ERROR`.
  - [ ] AC: 3, 4.

- [ ] Update deferred-work tracking idempotently.
  - [ ] Use an existing deferred-work artifact if the implemented source already defines one.
  - [ ] If no automation-specific deferred-work artifact exists, use the established `{implementation_artifacts}/deferred-work.md` precedent or create a deterministic structured artifact in the automated review artifact directory.
  - [ ] Record story reference, finding id, source gate, Linear id, Linear URL, deferred status, original finding detail, and evidence pointers.
  - [ ] Preserve enough source context for later maintainers to understand what was deferred.
  - [ ] Do not duplicate deferred-work entries on repeated sync runs for the same finding and Linear issue.
  - [ ] Treat failed deferred-work write or validation as `ERROR`.
  - [ ] AC: 3, 4.

- [ ] Make the multi-target update safe and diagnosable.
  - [ ] Validate the sync request before writing any artifact.
  - [ ] Write artifacts through a temp-file or equivalent safe-write path when practical.
  - [ ] Validate all updated artifacts after writing.
  - [ ] If any target fails, emit `ERROR` and list all targets that were not confirmed consistent.
  - [ ] Do not claim success unless `decision-needed.json`, story Review Findings, decision log, and deferred-work tracking all reflect the same Linear id, Linear URL, finding id, story reference, source gate, and deferred status.
  - [ ] Keep repeated sync with the same request idempotent.
  - [ ] AC: 1, 2, 3, 4.

- [ ] Keep scope within BMAD artifact sync.
  - [ ] Do not call the Linear API.
  - [ ] Do not create Linear issues.
  - [ ] Do not choose which findings should be deferred.
  - [ ] Do not alter BMAD review triage categories.
  - [ ] Do not change CR gate mapping.
  - [ ] Do not parse Markdown reports for route decisions.
  - [ ] Do not implement Archon DAG routing, PR creation, or `decision-needed-check`.
  - [ ] AC: 1, 2, 3, 4.

- [ ] Add focused sync fixtures or tests.
  - [ ] Add a successful sync fixture with one Archon-shaped Linear reference and all four BMAD targets.
  - [ ] Assert `decision-needed.json` records Linear id, Linear URL, and `deferred_to_linear`.
  - [ ] Assert story Review Findings record source gate, finding id, Linear id, Linear URL, and deferred status.
  - [ ] Assert the decision log preserves original finding detail and evidence pointers.
  - [ ] Assert deferred-work tracking preserves original finding detail and evidence pointers.
  - [ ] Add a missing-required-field fixture and assert `ERROR` with no successful target update reported.
  - [ ] Add a target-failure fixture and assert `ERROR` with failed target details.
  - [ ] Add an idempotency fixture that repeats the same sync request and proves no duplicate target records.
  - [ ] AC: 1, 2, 3, 4.

- [ ] Validate the repository.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json` and confirm the interactive skill still has no findings.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill has no findings.
  - [ ] Run the focused sync fixtures or tests added by this story.
  - [ ] Run `npm run validate:skills`.
  - [ ] Before marking the story done, run `npm run quality` on the exact checkout that will be committed.
  - [ ] AC: 1, 2, 3, 4.

## Dev Notes

### Implementation Precondition

Story M3.2 depends on Story M3.1 and the Archon decision-needed issue-reference input contract.
[Source: _bmad-output/planning-artifacts/epics.md:289]

At story creation time, `src/bmm-skills/4-implementation/bmad-code-review-auto/` does not exist in tracked source.

M1.1, M1.2, M2.1, M2.2, and M3.1 are currently ready for development, not completed implementation evidence.

Do not implement M3.2 until M3.1 has produced durable `decision-needed.json` records with stable finding ids and status fields.

The M3.1 story intentionally leaves Linear id and URL sync to this story.
[Source: _bmad-output/implementation-artifacts/m3-1-persist-decision-needed-json.md]

### Requirements Context

The epics define this story as syncing Linear references into BMAD artifacts.
[Source: _bmad-output/planning-artifacts/epics.md:281]

The required contract includes Linear issue id, Linear URL, finding id, story reference, sync target list, and sync output.
[Source: _bmad-output/planning-artifacts/epics.md:290]

The required integration validation must prove updates to story Review Findings, decision log, deferred-work tracking, and `decision-needed.json`.
[Source: _bmad-output/planning-artifacts/epics.md:292]

The PRD says BMAD-METHOD records Linear references after Archon creates or reuses issues.
[Source: _bmad-output/planning-artifacts/prd.md:117]

The PRD names the sync targets as story Review Findings, decision log, deferred-work tracking, and `decision-needed.json`.
[Source: _bmad-output/planning-artifacts/prd.md:118]

The architecture requires BMAD-METHOD artifacts to record Linear references and deferred status after Archon supplies them.
[Source: _bmad-output/planning-artifacts/architecture.md:67]

The architecture says failed or inconsistent sync target state produces `ERROR`.
[Source: _bmad-output/planning-artifacts/architecture.md:127]

The architecture says Linear issue creation and reference supply are Archon responsibilities.
[Source: _bmad-output/planning-artifacts/architecture.md:144]

The implementation readiness report specifically instructs Story M3.2 to include subtasks for story Review Findings, decision log, deferred-work tracking, and `decision-needed.json`.
[Source: _bmad-output/planning-artifacts/implementation-readiness-report-2026-06-30.md:329]

### Sync Contract

The sync path records references that already exist.

It does not own Linear issue creation.

It must validate the input before writing any BMAD artifact.

It must update all requested BMAD targets consistently.

It must emit `ERROR` for missing required fields, missing matching findings, malformed target artifacts, failed writes, failed validation, or inconsistent target state.

It must be idempotent for repeated sync of the same finding and Linear issue.

### Required Sync Targets

`decision-needed.json` is the durable machine-readable source for decision-needed findings.

The story Review Findings section is the human-readable story trail.

The decision log is the durable decision history for the deferred item.

Deferred-work tracking is the maintainer-facing list of deferred work.

All four targets must preserve original finding detail and evidence pointers.

All four targets must record source gate, finding id, Linear id, Linear URL, and deferred status when that target can represent those fields.

### Scope Boundaries

Story M3.1 owns creation and validation of `decision-needed.json`.
[Source: _bmad-output/planning-artifacts/epics.md:258]

Story M4.1 owns the broader deterministic outcome fixture matrix.
[Source: _bmad-output/planning-artifacts/epics.md:313]

This story may add focused sync fixtures for success, missing fields, target failure, and idempotency.

Do not implement BMAD-TEA gate behavior in this story.

Do not implement Archon DAG routing, PR creation, or Linear issue creation.

Do not require implementation agents to traverse outside this repository for parent workspace planning files.
[Source: _bmad-output/planning-artifacts/prd.md:13]

### Current Interactive Review Semantics

The current interactive review skill is `src/bmm-skills/4-implementation/bmad-code-review`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/SKILL.md:1]

Its presentation step writes Review Findings into the story file.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:19]

Its presentation step appends deferred findings to `{implementation_artifacts}/deferred-work.md`.
[Source: src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md:29]

Use those as human-artifact precedents, but do not copy the interactive resolution or patch-handling behavior into automated sync.

### Source Files To Read Before Editing

Read the implemented automation source under:

```text
src/bmm-skills/4-implementation/bmad-code-review-auto/
```

If the directory does not exist, implementation is blocked by Story M1.1.

Read the M3.1 implementation before relying on the `decision-needed.json` schema or status values.

Read any M3.1 persistence tests or fixtures before adding sync tests.

Read the M2.1 implementation before using CR contract fields to locate story and decision-needed artifacts.

Read the M2.2 implementation before touching human report references.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md` for story Review Findings and deferred-work precedents.

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

If sync helpers are shared between skills, place the shared material outside skill directories.

### Testing Requirements

Run these checks at minimum:

```bash
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json
npm run validate:skills
npm run quality
```

Run the focused Linear-reference sync fixtures or contract checks added by this story.

`npm run quality` is the required project-level pre-push check.
[Source: package.json:43]

The deterministic skill validator is included in `quality` through `npm run validate:skills`.
[Source: package.json:52]

### Latest Technical Information

No external API or third-party library research is required for this story.

This story records Linear references supplied by Archon and does not call the Linear API.

Use the repository versions and scripts from `package.json`.

### Previous Story Intelligence

The previous Epic M3 story is M3.1, `m3-1-persist-decision-needed-json`.

It is currently ready for development, not done, so no implemented M3.1 code patterns are available yet.

It established that `decision-needed.json` should preserve structured evidence and support a future `deferred_to_linear` status.

It also established that Linear issue id and URL sync are owned by this story.

Recent commits are Archon configuration and handoff planning work, not implementation for this story.

### Project Structure Notes

This is a brownfield source change.

Use tracked source under `src/bmm-skills/4-implementation/`.

Do not manually edit `.agents` or `_bmad` generated install output as source.

Keep sync artifacts structured and deterministic so repeated Archon handoff attempts do not duplicate records.

## Dev Agent Record

### Agent Model Used

Not set until implementation.

### Debug Log References

### Completion Notes List

### File List
