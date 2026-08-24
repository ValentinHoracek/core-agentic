---
name: dotnet-solution-architect
type: hero
description: Use when translating a raw .NET/C# feature requirement, including dictated stream-of-consciousness text, all the way through to a merged, tested, reviewed implementation
---

# dotnet-solution-architect

## Overview

Orchestrates a 6-stage pipeline from a raw requirement to a merged, tested, reviewed .NET 10/C# feature. Never takes a destructive action (e.g. deleting a worktree) automatically — every failure pauses and returns control to the user. Only Stages 1 and 6 branch on `mode`; Stage 2's ambiguity-stop and Stage 4's destructive-action pause are unconditional.

## When to Use

- A raw .NET/C# requirement (possibly dictated, unstructured) needing spec, plan, implementation, verification, and review.
- Not for one-off fixes — use underlying skills (e.g. `superpowers:test-driven-development`) directly.

## Pipeline State

State lives in `docs/dotnet-solution-architect/<date>-<slug>-.das-state.json` (`<date>` = `YYYY-MM-DD` at Stage 1; `<slug>` kebab-case, 5-6 words, from the Goal line). Artifacts share that prefix — runs never collide.

```json
{
  "stage": 3,
  "mode": "with-user",
  "slug": "add-refund-api",
  "artifacts": {
    "spec": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-SPEC.md",
    "plan": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-PLAN.md",
    "decisions": "docs/dotnet-solution-architect/2026-08-24-add-refund-api-DECISIONS.md"
  },
  "worktree": "../agent-worktree"
}
```

Check for `*-.das-state.json` under `docs/dotnet-solution-architect/` at invocation start; if present, resume from `stage`, not restart at Stage 1 (mode/slug carry forward, never re-asked); if absent, derive `slug`/`mode` in Stage 1, create the file. Each stage advances `stage`, recording new paths (Stages 4, 6 differ).

## The 6 Stages

### Stage 1 — Requirements

`mode` = `"automatic"` only if explicitly requested, else `"with-user"`; `slug`: kebab-case, 5-6 words, from the Goal line. Create the state file, pass `mode` into `dictation-spec-writer` — its `brainstorming` loop/gate stay unaffected; its confirmation step is mode-conditional.

### Stage 2 — Architecture & Decisions

See Quick Reference for the skill order.

### Stage 3 — Isolated TDD Implementation

`using-git-worktrees` reuses the recorded `worktree`, never re-provisions; `executing-plans` implements `PLAN.md` there.

### Stage 4 — Hard Verification Gate

`hard-verification-gate` gates the worktree: PASS stages the diff (`git add -A`; see Common Mistakes) and advances to `stage: 5`; Paused means STOP — leave `stage: 4`, present the report, never touch the worktree.

### Stage 5 — Triple Parallel Audit

`dispatching-parallel-agents` runs `reviewer-micro`/`macro`/`ops` against `git diff --staged` plus the run's `SPEC.md`, writing `docs/dotnet-solution-architect/<date>-<slug>-REVIEW_{MICRO,MACRO,OPS}.json`.

### Stage 6 — Human Arbitration

`receiving-code-review` builds a triage matrix (finding, reviewer, severity).

- With-user mode, or any `concerns`/`fail` verdict: ask per finding, **Approve & Merge** or **Fix & Loop**; automatic mode with all `pass`: skip that ask, go straight to Approve & Merge — but still ask `finishing-a-development-branch`'s merge-method question (mechanics, not safety).
- **Approve & Merge:** `finishing-a-development-branch` merges; delete `<date>-<slug>-.das-state.json` — pipeline complete.
- **Fix & Loop:** set `stage: 3` (reuse `worktree`), pass findings in as refactor instructions, re-enter Stage 3.

## Quick Reference

| Stage | Entry skill(s) | Output |
|---|---|---|
| 1 | `core-agentic:dictation-spec-writer` → `superpowers:brainstorming` | `docs/dotnet-solution-architect/<date>-<slug>-SPEC.md` |
| 2 | `superpowers:writing-plans` → `core-agentic:decision-recorder` → `core-agentic:replan` | run's `PLAN.md`, `DECISIONS.md` (run-prefixed) |
| 3 | `superpowers:using-git-worktrees` → `superpowers:executing-plans` (+ `superpowers:test-driven-development`) | passing tests in worktree |
| 4 | `core-agentic:hard-verification-gate` (→ `superpowers:systematic-debugging`) | PASS + staged diff, or paused report |
| 5 | `superpowers:dispatching-parallel-agents` (`core-agentic:reviewer-micro`, `core-agentic:reviewer-macro`, `core-agentic:reviewer-ops`) | 3× run-prefixed `REVIEW_*.json` |
| 6 | `superpowers:receiving-code-review` → `superpowers:finishing-a-development-branch` | merge, or loop to Stage 3 |

## Common Mistakes

- **Skipping the state file.** An interrupted session restarts from Stage 1.
- **Re-provisioning the worktree on Fix & Loop re-entry.** Reuse the recorded one.
- **Running Stage 5 against a failing build.** Stage 4 PASS is required.
- **Forgetting to stage the diff after Stage 4 PASS.** No sub-skill runs `git add`; the orchestrator does it.
- **Auto-resolving Stage 6.** Only when mode is `"automatic"` and all verdicts pass; else the user decides.
