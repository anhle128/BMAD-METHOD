# Story M2.1: Emit `code-review-auto.gate.json`

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As an Archon workflow maintainer,
I want BMAD-METHOD automated review to emit a stable CR gate contract,
so that Archon can route on JSON without parsing markdown.

## Acceptance Criteria

1. Given automated review completes, when route-facing output is written, then `code-review-auto.gate.json` includes the required envelope, count fields, and evidence pointers.
2. Given patch findings exist, when the contract is emitted, then `gate` is `FAIL` and `blocking_findings_count` is greater than zero.
3. Given only decision-needed findings exist, when the contract is emitted, then `gate` is `CONCERNS` and `decision_needed_file` points to `decision-needed.json`.
4. Given no `patch` and no `decision_needed` findings exist, when the contract is emitted, then `gate` is `PASS`.
5. Given reviewer execution fails, evidence is invalid, or output is untrusted, when the contract is emitted or validated, then `gate` is `ERROR`.

## Tasks / Subtasks

- [ ] Verify the M1 dependency baseline before implementation.
  - [ ] Confirm Story M1.1 has created `src/bmm-skills/4-implementation/bmad-code-review-auto/`.
  - [ ] Confirm Story M1.2 has implemented normalized automated findings with `patch`, `decision_needed`, `defer`, and `dismiss`.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill validates.
  - [ ] If the automation skill source directory or M1.2 finding model is missing, stop and complete the dependency stories before implementing this story.
  - [ ] AC: 1, 2, 3, 4, 5.

- [ ] Define the route-facing contract writer for `bmad-code-review-auto`.
  - [ ] Emit a JSON file named `code-review-auto.gate.json`.
  - [ ] Write the file in the deterministic automated review artifact directory established by M1.1.
  - [ ] If M1.1 did not establish an artifact directory, use the review output directory under `{implementation_artifacts}` and document the path in the automation skill.
  - [ ] Ensure the contract is generated from normalized review findings and execution status, not by parsing a markdown report.
  - [ ] AC: 1.

- [ ] Include the required contract fields exactly.
  - [ ] Include `contract_version`.
  - [ ] Include `workflow`.
  - [ ] Include `story_ref`.
  - [ ] Include `node`.
  - [ ] Include `round`.
  - [ ] Include `gate`.
  - [ ] Include `patch_count`.
  - [ ] Include `decision_needed_count`.
  - [ ] Include `defer_count`.
  - [ ] Include `dismiss_count`.
  - [ ] Include `blocking_findings_count`.
  - [ ] Include `decision_needed_file`.
  - [ ] Include `report_file`.
  - [ ] Include `story_file`.
  - [ ] Do not add extra route category fields that Archon would need to reinterpret.
  - [ ] AC: 1.

- [ ] Define stable field semantics.
  - [ ] Set `contract_version` to a stable initial value such as `1`.
  - [ ] Set `workflow` to identify BMAD automated code review.
  - [ ] Set `story_ref` and `story_file` from the review input or story context.
  - [ ] Set `node` and `round` from invocation context when provided.
  - [ ] Use deterministic defaults for `node` and `round` when invocation context is absent.
  - [ ] Count `patch`, `decision_needed`, `defer`, and `dismiss` from the normalized M1.2 findings.
  - [ ] Keep all count fields as non-negative integers.
  - [ ] Set `blocking_findings_count` to `patch_count` unless a later architecture decision defines a broader blocker calculation.
  - [ ] Set `decision_needed_file` to the deterministic `decision-needed.json` path when `decision_needed_count` is greater than zero.
  - [ ] Use `null` for `decision_needed_file` when there are no decision-needed findings.
  - [ ] Set `report_file` to the deterministic human-readable report path or placeholder path owned by Story M2.2.
  - [ ] Do not implement the full human report content in this story.
  - [ ] AC: 1, 2, 3.

- [ ] Implement gate mapping with error precedence.
  - [ ] Emit `ERROR` when reviewer execution fails, evidence is invalid, output is untrusted, or contract validation fails.
  - [ ] Emit `FAIL` when `patch_count` is greater than zero and no error condition exists.
  - [ ] Emit `CONCERNS` when `decision_needed_count` is greater than zero, `patch_count` is zero, and no error condition exists.
  - [ ] Emit `PASS` when `patch_count` and `decision_needed_count` are both zero and no error condition exists.
  - [ ] Ensure `defer_count` and `dismiss_count` never block the gate by themselves.
  - [ ] AC: 2, 3, 4, 5.

- [ ] Validate the emitted contract before reporting success.
  - [ ] Parse the JSON after writing or before final success output.
  - [ ] Reject missing required fields.
  - [ ] Reject unsupported `gate` values outside `PASS`, `FAIL`, `CONCERNS`, and `ERROR`.
  - [ ] Reject negative or non-integer count fields.
  - [ ] Reject untrusted output and invalid evidence as `ERROR` instead of falling through to `PASS`.
  - [ ] Ensure validation failures are diagnosable as review execution, evidence, contract, or trust failures.
  - [ ] AC: 1, 5.

- [ ] Preserve automation behavior from M1.
  - [ ] Do not apply patches from `patch` findings.
  - [ ] Do not present interactive next-step choices or menus.
  - [ ] Do not ask a human to resolve `decision_needed` inside `bmad-code-review-auto`.
  - [ ] Do not convert `decision_needed` findings to patches.
  - [ ] Do not introduce Archon-specific finding categories or gate values.
  - [ ] AC: 2, 3, 5.

- [ ] Add focused contract fixtures or tests.
  - [ ] Add a PASS fixture with no `patch` and no `decision_needed` findings.
  - [ ] Add a FAIL fixture with at least one `patch` finding and `blocking_findings_count` greater than zero.
  - [ ] Add a CONCERNS fixture with at least one `decision_needed` finding and no `patch` findings.
  - [ ] Add an ERROR fixture for reviewer execution failure, invalid evidence, or untrusted output.
  - [ ] Keep broader `defer` and `dismiss` outcome mapping fixtures scoped to Story M4.1 unless small local fixtures are required to prove count fields.
  - [ ] AC: 1, 2, 3, 4, 5.

- [ ] Validate the repository.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json` and confirm the interactive skill still has no findings.
  - [ ] Run `node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json` and confirm the automation skill has no findings.
  - [ ] Run the focused contract tests or fixture checks added by this story.
  - [ ] Run `npm run validate:skills`.
  - [ ] Before marking the story done, run `npm run quality` on the exact checkout that will be committed.
  - [ ] AC: 1, 2, 3, 4, 5.

## Dev Notes

### Implementation Precondition

Story M2.1 depends on Stories M1.1 and M1.2.
[Source: _bmad-output/planning-artifacts/epics.md:205]

At story creation time, `src/bmm-skills/4-implementation/bmad-code-review-auto/` does not exist in tracked source.

Do not implement M2.1 until M1.1 has created the automation skill and M1.2 has implemented the normalized triage vocabulary.

The M1.1 and M1.2 story files are ready for development, not completed implementation evidence.
[Source: _bmad-output/implementation-artifacts/m1-1-add-bmad-code-review-auto.md]
[Source: _bmad-output/implementation-artifacts/m1-2-preserve-review-layers-and-triage-vocabulary.md]

### Requirements Context

The PRD requires `bmad-code-review-auto` to emit `code-review-auto.gate.json`.
[Source: _bmad-output/planning-artifacts/prd.md:93]

The PRD requires the contract fields `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate`, `patch_count`, `decision_needed_count`, `defer_count`, `dismiss_count`, `blocking_findings_count`, `decision_needed_file`, `report_file`, and `story_file`.
[Source: _bmad-output/planning-artifacts/prd.md:94]

The PRD defines `FAIL`, `CONCERNS`, `PASS`, and `ERROR` routing conditions.
[Source: _bmad-output/planning-artifacts/prd.md:98]

The architecture defines `code-review-auto.gate.json` as the machine-readable route API.
[Source: _bmad-output/planning-artifacts/architecture.md:61]

The architecture says markdown reports are human evidence and may evolve.
[Source: _bmad-output/planning-artifacts/architecture.md:62]

The architecture says Archon routes on JSON only.
[Source: _bmad-output/planning-artifacts/architecture.md:63]

The architecture lists the same required contract shape and gate rules.
[Source: _bmad-output/planning-artifacts/architecture.md:71]

### Contract Rules

`code-review-auto.gate.json` is the route contract.

Markdown report content must not be the routing source.

The route gate values are only `PASS`, `FAIL`, `CONCERNS`, and `ERROR`.

Error conditions take precedence over content-derived gates because downstream routing must not trust invalid review output.

Patch findings produce a blocking gate.

Decision-needed findings produce a concerns gate only when no patch findings exist.

Defer and dismiss findings are counted for evidence but do not block routing by themselves.

### Scope Boundaries

Story M2.2 owns the full human-readable report content.
[Source: _bmad-output/planning-artifacts/epics.md:234]

Story M3.1 owns durable `decision-needed.json` content.
[Source: _bmad-output/planning-artifacts/epics.md:258]

Story M4.1 owns the broader outcome mapping fixture matrix for `patch`, `decision_needed`, `defer`, `dismiss`, and invalid or untrusted output.
[Source: _bmad-output/planning-artifacts/epics.md:317]

This story may add only the focused contract fixtures needed to prove PASS, FAIL, CONCERNS, and ERROR.

Do not implement Linear sync in this story.

Do not implement BMAD-TEA gate behavior in this story.

Do not require implementation agents to traverse outside this repository for parent workspace planning files.
[Source: _bmad-output/planning-artifacts/prd.md:13]

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

`bmad-code-review-auto` must not inherit interactive resolution or patch application.

### Source Files To Read Before Editing

Read the M1.1 and M1.2 implemented source under:

```text
src/bmm-skills/4-implementation/bmad-code-review-auto/
```

If the directory does not exist, implementation is blocked by Story M1.1.

Read any test or fixture files added by M1.2 before designing contract fixtures.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-03-triage.md` before deriving count fields from triage buckets.

Read `src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md` before deciding what behavior must stay out of automation.

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

If contract helpers are shared between skills, place the shared material outside both skill directories.

### Testing Requirements

Run these checks at minimum:

```bash
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review --json
node tools/validate-skills.js src/bmm-skills/4-implementation/bmad-code-review-auto --json
npm run validate:skills
npm run quality
```

Run the focused PASS, FAIL, CONCERNS, and ERROR contract fixtures or tests added by this story.

`npm run quality` is the required project-level pre-push check.
[Source: package.json:43]

The deterministic skill validator is included in `quality` through `npm run validate:skills`.
[Source: package.json:52]

### Latest Technical Information

No external API or third-party library research is required for this story.

This story updates BMAD skill instructions, JSON contract output, and local validation fixtures using the repository's existing Node toolchain.

Use the repository versions and scripts from `package.json`.

### Previous Story Intelligence

No previous Epic M2 story file exists in `_bmad-output/implementation-artifacts`.

The dependency stories M1.1 and M1.2 are currently ready for development, not done.

M1.1 establishes the source boundary under `src/bmm-skills/4-implementation/`.

M1.2 establishes the finding model and triage vocabulary that this story counts.

Recent commits are Archon configuration and handoff planning work, not implementation for this story.

### Project Structure Notes

This is a brownfield source change.

Use tracked source under `src/bmm-skills/4-implementation/`.

Do not manually edit `.agents` or `_bmad` generated install output as source.

Keep JSON contract routing stable even if human-readable report wording changes later.

## Dev Agent Record

### Agent Model Used

Not set until implementation.

### Debug Log References

### Completion Notes List

### File List
