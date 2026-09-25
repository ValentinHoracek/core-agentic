---
name: dotnet-solution-architect
type: hero
description: Use when translating a raw .NET/C# feature requirement, including dictated stream-of-consciousness text, all the way through to a merged, tested, reviewed implementation
---

# dotnet-solution-architect

## Overview

Orchestrates a 6-stage pipeline from a raw requirement to a merged, tested, reviewed .NET 10/C# feature. This skill is the only place where skills are chained, loops run, and `mode` is interpreted: the atomic `core-agentic` skills it calls stop at their output file and know nothing about this pipeline. Never takes a destructive action (e.g. deleting a worktree) automatically — every failure pauses and returns control to the user. Only Stages 1 and 6 branch on `mode`; Stage 2's ambiguity-stop and Stage 4's pause are unconditional.

## When to Use

- A raw .NET/C# requirement (possibly dictated, unstructured) needing spec, plan, implementation, verification, and review.
- Not for one-off fixes — use underlying skills (e.g. `superpowers:test-driven-development`) directly.

## Pipeline State

`<prefix>` = `docs/dotnet-solution-architect/<date>-<slug>-` (`<date>` = `YYYY-MM-DD` at Stage 1; `<slug>` as derived in Stage 1). State lives in `<prefix>.das-state.json`; every artifact is `<prefix><NAME>` — runs never collide.

```json
{
  "stage": 3,
  "mode": "with-user",
  "slug": "add-refund-api",
  "artifacts": {
    "draft": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-DRAFT.md",
    "spec": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-SPEC.md",
    "plan": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-PLAN.md",
    "decisions": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-DECISIONS.md",
    "verify": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-VERIFY.md"
  },
  "worktree": "../agent-worktree"
}
```

Check for `*-.das-state.json` under `docs/dotnet-solution-architect/` at invocation start; if present, resume from `stage`, not restart at Stage 1 (mode/slug carry forward, never re-asked); if absent, derive `slug`/`mode` in Stage 1, create the file. If more than one `*-.das-state.json` exists, resume the one whose slug matches the invocation's named feature; if none is named or more than one still matches, ask the user which run to resume rather than guessing. Each stage advances `stage` and records the artifact paths it wrote (Stages 4 and 6 differ, see below).

## Calling Atomic Skills

Every atomic `core-agentic` skill declares a `## Contract`: named inputs, one output file, and whether it asks the user. Pass every input by its contract name, always pass every output-path input (`output`, `plan`, `decisions`) as a `<prefix>` path, and read the output file — not the conversation — to decide the next step. An atomic skill never calls the next skill; this skill does.

## The 6 Stages

### Stage 1 — Requirements

`mode` = `"automatic"` only if explicitly requested, else `"with-user"`. Derive `slug` provisionally (kebab-case, 5-6 words) from the raw input before any draft exists and create the state file under that slug.

1. If the raw input is already a structured requirements document, skip to step 3 with the raw input as the starting context.
2. Otherwise, if the raw input is not already a file, write it verbatim to `<prefix>RAW.md`. Run `core-agentic:dictation-spec-writer` with `dictation` = the raw input's file path, `confirm` = `yes` in `with-user` mode or `no` in `automatic` mode, `output` = `<prefix>DRAFT.md`. Record `artifacts.draft`. Once the draft's Goal line exists, refine `slug` if it differs materially, renaming any artifacts already written under the old slug.
3. Run `superpowers:brainstorming` with the draft (or the structured input) as its starting context; its clarification loop and hard gate are unaffected by `mode`. Its result is `<prefix>SPEC.md`; record `artifacts.spec`.
4. If the draft's first line is the auto-approval note (`> Auto-approved without user confirmation — …`), copy that line to the top of `SPEC.md` so the audit trail survives.

Advance to `stage: 2`.

### Stage 2 — Architecture & Decisions

1. `superpowers:writing-plans` drafts `<prefix>PLAN.md` from `SPEC.md`.
2. `core-agentic:decision-recorder` runs once per significant decision made while drafting, with `decision` = the decision and its context, `decisions` = `<prefix>DECISIONS.md`.
3. `core-agentic:replan` runs with `spec` = `SPEC.md`, `plan` = `PLAN.md`, `decisions` = `DECISIONS.md`. It always stops to ask on any gap, ambiguity, or contradiction — unconditional in both modes.

Record `artifacts.plan` and `artifacts.decisions`; advance to `stage: 3`.

### Stage 3 — Isolated TDD Implementation

`superpowers:using-git-worktrees` reuses the recorded `worktree`, never re-provisions; `superpowers:executing-plans` implements `PLAN.md` there. Advance to `stage: 4`.

### Stage 4 — Hard Verification Loop

1. Run `core-agentic:hard-verification-gate` with `directory` = the worktree, `output` = `<prefix>VERIFY.md`. Record `artifacts.verify`.
2. Line 1 of `VERIFY.md` is `PASS`: stage the diff (`git add -A` in the worktree; see Common Mistakes) and advance to `stage: 5`.
3. Line 1 is `FAIL`: on the first FAIL, start `superpowers:systematic-debugging` from Phase 1 (root cause investigation) with `VERIFY.md` as the evidence. On every later FAIL, continue the same systematic-debugging session with the new `VERIFY.md` as evidence — never restart it, so its fix attempts are counted across gate runs. Fixes are made in the worktree. After each fix, go back to step 1 — the full check suite, not only the failed check. Do not keep a separate attempt counter; systematic-debugging's own stop is the limit.
4. systematic-debugging reaches its own stopping point (3 failed fixes → question the architecture): STOP. Leave `stage: 4`, present `VERIFY.md` and what was tried, and never delete, reset, or modify the worktree. Wait for the user.

### Stage 5 — Triple Parallel Audit

`superpowers:dispatching-parallel-agents` runs `core-agentic:reviewer-micro`, `core-agentic:reviewer-macro` and `core-agentic:reviewer-ops`, each with `directory` = the worktree, `spec` = `SPEC.md`'s resolved content (the file lives in the target repo's `docs/`, not inside the worktree — never pass a bare filename), and `output` = `<prefix>REVIEW_MICRO.json` / `<prefix>REVIEW_MACRO.json` / `<prefix>REVIEW_OPS.json`. Advance to `stage: 6`.

### Stage 6 — Human Arbitration

`superpowers:receiving-code-review` builds a triage matrix (finding, reviewer, severity) from the three `REVIEW_*.json` files.

- With-user mode, or any `concerns`/`fail` verdict: ask per finding, **Approve & Merge** or **Fix & Loop**; automatic mode with all `pass`: skip that ask, go straight to Approve & Merge — but still ask `superpowers:finishing-a-development-branch`'s merge-method question (mechanics, not safety).
- **Approve & Merge:** `superpowers:finishing-a-development-branch` merges; delete `<prefix>.das-state.json` — pipeline complete.
- **Fix & Loop:** set `stage: 3` (reuse `worktree`), pass findings in as refactor instructions, re-enter Stage 3.

## Quick Reference

| Stage | Entry skill(s) | Output |
|---|---|---|
| 1 | `core-agentic:dictation-spec-writer` (unless input is structured) → `superpowers:brainstorming` | `<prefix>DRAFT.md`, `<prefix>SPEC.md` |
| 2 | `superpowers:writing-plans` → `core-agentic:decision-recorder` → `core-agentic:replan` | `<prefix>PLAN.md`, `<prefix>DECISIONS.md` |
| 3 | `superpowers:using-git-worktrees` → `superpowers:executing-plans` (+ `superpowers:test-driven-development`) | passing tests in worktree |
| 4 | `core-agentic:hard-verification-gate` → (FAIL) `superpowers:systematic-debugging` → gate again | `<prefix>VERIFY.md`; PASS + staged diff, or paused |
| 5 | `superpowers:dispatching-parallel-agents` (`core-agentic:reviewer-micro`, `core-agentic:reviewer-macro`, `core-agentic:reviewer-ops`) | `<prefix>REVIEW_{MICRO,MACRO,OPS}.json` |
| 6 | `superpowers:receiving-code-review` → `superpowers:finishing-a-development-branch` | merge, or loop to Stage 3 |

## Common Mistakes

- **Skipping the state file.** An interrupted session restarts from Stage 1.
- **Letting an atomic skill continue the chain.** Atomic skills stop at their output file; this skill calls the next skill.
- **Re-provisioning the worktree on Fix & Loop re-entry.** Reuse the recorded one.
- **Running Stage 5 against a failing build.** Line 1 of `VERIFY.md` must be `PASS`.
- **Forgetting to stage the diff after Stage 4 PASS.** No sub-skill runs `git add`; this skill does it.
- **Auto-resolving Stage 6 on anything short of all-`pass`.** Only skip the ask when mode is `"automatic"` and every verdict is `pass`; any `concerns`/`fail`, or `with-user` mode, means the user decides.
