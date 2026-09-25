---
name: dotnet-solution-architect
type: hero
description: Use when a raw .NET/C# feature requirement, including dictated stream-of-consciousness text, must go all the way to a merged, tested, reviewed implementation
---

# dotnet-solution-architect

## Overview

This skill runs a 6-stage pipeline. The pipeline takes a raw requirement to a merged, tested, reviewed .NET 10/C# feature.

- This skill is the only place that chains skills, runs loops, and reads `mode`. The atomic `core-agentic` skills that it calls stop at their output file. They know nothing about this pipeline.
- This skill never does a destructive action (for example, deleting a worktree) automatically. At each failure, it pauses and gives control back to the user.
- Only Stages 1 and 6 change their behavior with `mode`. The ambiguity stop in Stage 2 and the pause in Stage 4 apply in all modes.

## When to Use

- A raw .NET/C# requirement (possibly dictated and unstructured) needs a spec, a plan, implementation, verification, and review.
- Do not use it for one-off fixes. For those, use the underlying skills (for example, `superpowers:test-driven-development`) directly.

## Pipeline State

`<prefix>` = `docs/dotnet-solution-architect/<date>-<slug>-`. `<date>` is `YYYY-MM-DD` at Stage 1. Stage 1 sets `<slug>`. The state file is `<prefix>.das-state.json`. Every artifact is `<prefix><NAME>`. Thus, two runs never use the same file.

```json
{
  "stage": 3,
  "mode": "with-user",
  "slug": "add-refund-api",
  "artifacts": {
    "draft": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-DRAFT.md",
    "draft_ste": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-DRAFT-STE.md",
    "spec": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-SPEC.md",
    "plan": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-PLAN.md",
    "decisions": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-DECISIONS.md",
    "verify": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-VERIFY.md"
  },
  "worktree": "../agent-worktree"
}
```

At the start of each invocation, look for `*-.das-state.json` in `docs/dotnet-solution-architect/`:

- **No state file:** this is a new run. Stage 1 sets `slug` and `mode` and creates the file.
- **One state file:** continue from its `stage`. Do not start again at Stage 1. Keep its `mode` and `slug`. Never ask for them again.
- **More than one state file:** continue the run whose slug matches the feature that the invocation names. If the invocation names no feature, or more than one file still matches, ask the user which run to continue. Do not guess.

Each stage sets the next `stage` value and records the paths of the artifacts that it wrote. Stages 4 and 6 are different (see below).

The top-level state fields `ste` and `ste_lint_hard` are informational. The resume logic does not read them.

## Calling Atomic Skills

Each atomic `core-agentic` skill has a `## Contract` section. The contract gives the named inputs, the one output file, and whether the skill asks the user.

- Give each input by its contract name.
- Always give each output-path input (`output`, `plan`, `decisions`) as a `<prefix>` path.
- Read the output file, not the conversation, to decide the next step.
- An atomic skill never calls the next skill. This skill calls it.

## The 6 Stages

### Stage 1 — Requirements

Set `mode`. The value is `"automatic"` only if the user explicitly asked for it. In all other cases, the value is `"with-user"`. Make a provisional `slug` (kebab-case, 5-6 words) from the raw input before a draft exists. Create the state file with that slug.

1. **Structured input.** If the raw input is a structured requirements document:
   - If the input is not a file, write it verbatim to `<prefix>RAW.md`. Then `<prefix>RAW.md` is the **rewrite input**.
   - If the input is a file, that file is the **rewrite input**.
   - Do not run `core-agentic:dictation-spec-writer`. Go to step 3.
2. **Draft.** If the raw input is not structured:
   - If the raw input is not a file, write it verbatim to `<prefix>RAW.md`.
   - Run `core-agentic:dictation-spec-writer` with `dictation` = the file path of the raw input, `confirm` = `yes` in `with-user` mode or `no` in `automatic` mode, and `output` = `<prefix>DRAFT.md`.
   - Record `artifacts.draft`.
   - When the Goal line of the draft exists, compare it with `slug`. If the Goal is materially different, change `slug`. Rename each artifact that has the old slug.
   - The draft is the **rewrite input**.
3. **STE rewrite.** Run `asd-ste100` on the rewrite input in one pass:
   - Use Strict mode for the Requirements, Edge Cases, and Acceptance Criteria sections (or the matching sections of a structured input). Use STE-flavored mode for all other sections. Apply the two modes per section in this one `asd-ste100` pass.
   - Keep every heading exactly as it is. When line 1 is the auto-approval note, keep line 1 exactly as it is.
   - Write only the rewritten document to `<prefix>DRAFT-STE.md`. Do not write a `Kept as-is:` line, a glossary suggestion, or other commentary from `asd-ste100`.
   - If `asd-ste100` reports that the text already complies, copy the rewrite input without changes to `<prefix>DRAFT-STE.md`.
   - Record `artifacts.draft_ste`. Never overwrite the rewrite input.
4. **Lint.** Run `python3 <asd-ste100 skill directory>/scripts/ste-lint.py --json <prefix>DRAFT-STE.md`. Record its `hard_count` as the top-level state field `ste_lint_hard`.
   - The lint is a signal, not a gate. Exit code 1 only means that hard violations exist.
   - Continue for all counts. Do not run the rewrite again because of the count.
5. **Spec.** Run `superpowers:brainstorming` with `DRAFT-STE.md` as its starting context. `mode` does not change its clarification loop or its hard gate. Its result is `<prefix>SPEC.md`. Record `artifacts.spec`.
6. **Audit trail.** Do this step only if `DRAFT.md` exists. If line 1 of `DRAFT.md` is the auto-approval note (`> Auto-approved without user confirmation — …`), copy that line to the top of `SPEC.md`.

If `asd-ste100` is not installed:

- Do not do steps 3 and 4.
- Set the top-level state field `"ste": "skipped: asd-ste100 not installed"`.
- In step 5, give brainstorming the rewrite input, not `DRAFT-STE.md`.

Set `stage: 2`.

### Stage 2 — Architecture & Decisions

1. Run `superpowers:writing-plans`. It writes `<prefix>PLAN.md` from `SPEC.md`.
2. For each significant decision made while `superpowers:writing-plans` drafts the plan, run `core-agentic:decision-recorder` one time. Give `decision` = the decision and its context, and `decisions` = `<prefix>DECISIONS.md`.
3. Run `core-agentic:replan` with `spec` = `SPEC.md`, `plan` = `PLAN.md`, and `decisions` = `DECISIONS.md`. It always stops and asks the user about each gap, ambiguity, or contradiction. This occurs in both modes.

Record `artifacts.plan` and `artifacts.decisions`. Set `stage: 3`.

### Stage 3 — Isolated TDD Implementation

Run `superpowers:using-git-worktrees`. It uses the recorded `worktree` again and never makes a new one. Then run `superpowers:executing-plans`. It implements `PLAN.md` in that worktree. Set `stage: 4`.

### Stage 4 — Hard Verification Loop

1. Run `core-agentic:hard-verification-gate` with `directory` = the worktree and `output` = `<prefix>VERIFY.md`. Record `artifacts.verify`.
2. If line 1 of `VERIFY.md` is `PASS`: stage the diff (`git add -A` in the worktree, see Common Mistakes). Set `stage: 5`.
3. If line 1 is `FAIL`:
   - On the first FAIL, start `superpowers:systematic-debugging` from Phase 1 (root cause investigation). Give it `VERIFY.md` as the evidence.
   - On each later FAIL, continue the same systematic-debugging session. Give it the new `VERIFY.md` as evidence. Never start it again. Thus, it counts its fix attempts across all gate runs.
   - Make the fixes in the worktree.
   - After each fix, go back to step 1. Run the full check suite, not only the failed check.
   - Do not keep a separate attempt counter. The stop of systematic-debugging is the limit.
4. If systematic-debugging gets to its stop (3 failed fixes, then it questions the architecture): STOP.
   - Keep `stage: 4`.
   - Show `VERIFY.md` and the attempted fixes to the user.
   - Never delete, reset, or change the worktree.
   - Wait for the user.

### Stage 5 — Triple Parallel Audit

Run `superpowers:dispatching-parallel-agents` with three agents: `core-agentic:reviewer-micro`, `core-agentic:reviewer-macro`, and `core-agentic:reviewer-ops`. Give each agent these inputs:

- `directory` = the worktree.
- `spec` = the content of `SPEC.md`. The file is in `docs/` of the target repo, not in the worktree. Thus, never give only the file name.
- `output` = `<prefix>REVIEW_MICRO.json`, `<prefix>REVIEW_MACRO.json`, or `<prefix>REVIEW_OPS.json`.

Set `stage: 6`.

### Stage 6 — Human Arbitration

Run `superpowers:receiving-code-review`. It makes a triage matrix (finding, reviewer, severity) from the three `REVIEW_*.json` files.

- **Ask the user** in `with-user` mode, or when one or more verdicts are `concerns` or `fail`. For each finding, ask: **Approve & Merge** or **Fix & Loop**.
- **Do not ask** in `automatic` mode when all verdicts are `pass`. Go directly to Approve & Merge. But still ask the merge-method question of `superpowers:finishing-a-development-branch`. That question is about mechanics, not safety.
- **Approve & Merge:** `superpowers:finishing-a-development-branch` merges. Delete `<prefix>.das-state.json`. The pipeline is complete.
- **Fix & Loop:** set `stage: 3` and use the same `worktree`. Give the findings to Stage 3 as refactor instructions. Start Stage 3 again.

## Quick Reference

| Stage | Entry skill(s) | Output |
|---|---|---|
| 1 | `core-agentic:dictation-spec-writer` (unless input is structured) → `asd-ste100` → `superpowers:brainstorming` | `<prefix>DRAFT.md`, `<prefix>DRAFT-STE.md`, `<prefix>SPEC.md` |
| 2 | `superpowers:writing-plans` → `core-agentic:decision-recorder` → `core-agentic:replan` | `<prefix>PLAN.md`, `<prefix>DECISIONS.md` |
| 3 | `superpowers:using-git-worktrees` → `superpowers:executing-plans` (+ `superpowers:test-driven-development`) | passing tests in worktree |
| 4 | `core-agentic:hard-verification-gate` → (FAIL) `superpowers:systematic-debugging` → gate again | `<prefix>VERIFY.md`. Then PASS + staged diff, or paused |
| 5 | `superpowers:dispatching-parallel-agents` (`core-agentic:reviewer-micro`, `core-agentic:reviewer-macro`, `core-agentic:reviewer-ops`) | `<prefix>REVIEW_{MICRO,MACRO,OPS}.json` |
| 6 | `superpowers:receiving-code-review` → `superpowers:finishing-a-development-branch` | merge, or loop to Stage 3 |

## Common Mistakes

- **No state file.** Without it, an interrupted session starts again at Stage 1.
- **Using `ste-lint` as a gate, or overwriting the rewrite input.** Record the lint count. Never act on it. The rewrite always goes to `DRAFT-STE.md`.
- **An atomic skill that continues the chain.** Atomic skills stop at their output file. This skill calls the next skill.
- **A new worktree on Fix & Loop.** Use the recorded worktree again.
- **Stage 5 on a failing build.** Line 1 of `VERIFY.md` must be `PASS`.
- **No staged diff after the Stage 4 PASS.** No sub-skill runs `git add`. This skill runs it.
- **Automatic decision in Stage 6 without all `pass` verdicts.** Skip the question only when `mode` is `"automatic"` and every verdict is `pass`. If a verdict is `concerns` or `fail`, or in `with-user` mode, the user decides.
