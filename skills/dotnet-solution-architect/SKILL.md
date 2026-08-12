---
name: dotnet-solution-architect
type: hero
description: Use when translating a raw .NET/C# feature requirement, including dictated stream-of-consciousness text, all the way through to a merged, tested, reviewed implementation
---

# dotnet-solution-architect

## Overview

Orchestrates a 6-stage pipeline that turns a raw requirement into a merged, tested, reviewed .NET 10/C# feature. Holds pipeline state on disk so an interrupted session can resume without repeating completed stages. Never takes a destructive action (e.g. deleting a worktree) automatically — every failure path pauses and returns control to the user.

## When to Use

- You have a raw requirement (possibly a dictated, unstructured markdown dump) for a .NET/C# feature and want it carried through spec, plan, implementation, verification, and review as one pipeline.
- Not for quick one-off fixes — this is the full pipeline; for something small, use the underlying skills (`superpowers:test-driven-development`, etc.) directly instead.

## Pipeline State

State is tracked in a `.das-state.json` file created next to the pipeline's artifacts (in the target repo, at the feature's working location):

```json
{
  "stage": 3,
  "artifacts": {
    "spec": "SPEC.md",
    "plan": "PLAN.md",
    "decisions": "DECISIONS.md"
  },
  "worktree": "../agent-worktree"
}
```

At the start of every invocation, check for this file. If present, resume from `stage` rather than starting at Stage 1. Update `stage` and `artifacts` after each stage completes.

## The 6 Stages

### Stage 1 — Requirements

**REQUIRED SUB-SKILL:** Use `dictation-spec-writer` on the raw dictation input, which itself hands off to `superpowers:brainstorming` for clarification and confirmation.
**Output:** `SPEC.md`. Update state: `stage: 2`, `artifacts.spec: "SPEC.md"`.

### Stage 2 — Architecture & Decisions

1. **REQUIRED SUB-SKILL:** Use `superpowers:writing-plans` against `SPEC.md` to draft `PLAN.md`.
2. **REQUIRED SUB-SKILL:** Use `decision-recorder` for each significant decision made while drafting the plan, appending to `DECISIONS.md`.
3. **REQUIRED SUB-SKILL:** Use `replan` against `SPEC.md`, `DECISIONS.md`, and the draft `PLAN.md` to do the final review pass and finalize the plan.

**Output:** `PLAN.md`, `DECISIONS.md`. Update state: `stage: 3`, `artifacts.plan: "PLAN.md"`, `artifacts.decisions: "DECISIONS.md"`.

### Stage 3 — Isolated TDD Implementation

1. **REQUIRED SUB-SKILL:** Use `superpowers:using-git-worktrees` to create an isolated worktree for this feature. Record its path in `worktree`.
2. **REQUIRED SUB-SKILL:** Use `superpowers:executing-plans` to walk `PLAN.md` task-by-task inside that worktree, applying `superpowers:test-driven-development`'s red-green-refactor cycle for each task.

**Output:** compiling source and passing tests in the worktree. Update state: `stage: 4`.

### Stage 4 — Hard Verification Gate

**REQUIRED SUB-SKILL:** Use `hard-verification-gate` against the worktree.

- If it reports PASS: stage the full worktree diff (`git add -A` in the worktree) — Stage 5's reviewer skills all read `git diff --staged` and produce nothing meaningful against an empty staging area. Then update state to `stage: 5` and continue.
- If it reports a paused pipeline (its own report of what's still broken): STOP here. Leave `stage: 4` in the state file. Present the report to the user and wait — do not proceed to Stage 5 and do not touch the worktree.

### Stage 5 — Triple Parallel Audit

**REQUIRED SUB-SKILL:** Use `superpowers:dispatching-parallel-agents` to run three concurrent, read-only agents against `git diff --staged` in the worktree (each capped to roughly 2,000 tokens of context: the diff plus `SPEC.md`):
- One running `reviewer-micro` → `REVIEW_MICRO.json`
- One running `reviewer-macro` → `REVIEW_MACRO.json`
- One running `reviewer-ops` → `REVIEW_OPS.json`

**Output:** the three JSON files. Update state: `stage: 6`.

### Stage 6 — Human Arbitration

1. **REQUIRED SUB-SKILL:** Use `superpowers:receiving-code-review` to consolidate the three `REVIEW_*.json` files into a triage matrix (finding, source reviewer, severity) for the user.
2. Present the matrix and ask the user to decide per finding (or overall): **Approve & Merge**, or **Fix & Loop**.
3. If **Approve & Merge**: **REQUIRED SUB-SKILL:** use `superpowers:finishing-a-development-branch` to merge. Delete the `.das-state.json` file — the pipeline is complete.
4. If **Fix & Loop**: set `stage: 3` in the state file (same `worktree`, do not re-provision it), pass the specific findings as targeted refactor instructions into Stage 3, and re-enter the pipeline at Stage 3.

## Quick Reference

| Stage | Entry skill(s) | Output |
|---|---|---|
| 1 | `dictation-spec-writer` → `superpowers:brainstorming` | `SPEC.md` |
| 2 | `superpowers:writing-plans` → `decision-recorder` → `replan` | `PLAN.md`, `DECISIONS.md` |
| 3 | `superpowers:using-git-worktrees` → `superpowers:executing-plans` (+ `superpowers:test-driven-development`) | passing tests in worktree |
| 4 | `hard-verification-gate` (→ `superpowers:systematic-debugging` on failure) | PASS + staged diff, or paused report |
| 5 | `superpowers:dispatching-parallel-agents` (`reviewer-micro`/`macro`/`ops`) | 3× `REVIEW_*.json` |
| 6 | `superpowers:receiving-code-review` → `superpowers:finishing-a-development-branch` | merge, or loop to Stage 3 |

## Common Mistakes

- **Skipping the state file.** Without it, an interrupted session restarts the whole pipeline from Stage 1 instead of resuming.
- **Re-provisioning the worktree on a Stage 3 re-entry from Fix & Loop.** Reuse the existing one recorded in state.
- **Letting Stage 5 run against a failing build.** Stage 4 PASS is a hard prerequisite — never dispatch reviewers otherwise.
- **Forgetting to stage the diff after Stage 4 PASS.** `git diff --staged` is empty until something runs `git add`; no sub-skill in Stages 3–4 does this on its own, so the orchestrator must do it directly as part of Stage 4's PASS path before entering Stage 5.
- **Auto-resolving Stage 6.** The merge-vs-loop decision is always the user's call, never automatic.
