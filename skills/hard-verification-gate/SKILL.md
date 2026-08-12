---
name: hard-verification-gate
type: atomic
description: Use when implementation work in a worktree needs a binary pass/fail verification check (build, tests, architecture rules, container run) before it can proceed to code review
---

# Hard Verification Gate

## Overview

Runs the project's binary terminal checks and only lets work proceed to review if they all pass. On failure, hands off to `superpowers:systematic-debugging` rather than retrying blindly — and relies entirely on that skill's own stopping point rather than keeping a second counter. Never destroys the worktree automatically.

## When to Use

- Implementation work (Stage 3 of the `dotnet-solution-architect` pipeline) claims to be done and needs a go/no-go check before Stage 5 review.
- Not a substitute for `superpowers:verification-before-completion`'s evidence-before-assertions discipline — this skill adds the specific "what happens on failure" handoff on top of it.

## Process

1. Run every configured binary check for the solution: `dotnet build`, the test suite, architecture rules (e.g. ArchUnitNET), and any container/integration checks (e.g. `docker compose up --build` if the feature involves one). Capture full output of each.
2. If every check passes: report PASS and proceed — this stage is done.
3. If any check fails: **REQUIRED SUB-SKILL:** use `superpowers:systematic-debugging` on the failure, starting from Phase 1 (root cause investigation) with the captured output as evidence. Do not attempt a fix before it completes Phase 1.
4. Let `systematic-debugging` run its own cycle (investigate → hypothesize → fix → verify) exactly as that skill defines it. Do not add a separate attempt counter here — `systematic-debugging` already stops itself and treats 3 failed fixes as a sign of an architectural problem rather than continuing.
5. After each fix `systematic-debugging` proposes, re-run the full check suite from step 1 (not just the check that failed) before deciding whether to continue.
6. If all checks now pass: report PASS and proceed.
7. If `systematic-debugging` reaches its own stopping point (3 failed fixes, "question the architecture"): STOP. Report back to the user: what was tried, what's still failing (full output), and where the worktree is. **Do not delete, reset, or modify the worktree.** Wait for the user's decision.
8. Reviewers (Stage 5) are never invoked while any check is failing — PASS is a hard prerequisite for Stage 5.

## Quick Reference

| Outcome | Action |
|---|---|
| All checks pass | Report PASS, proceed to Stage 5 |
| Checks fail, first attempt | Hand off to `systematic-debugging` Phase 1 |
| `systematic-debugging` fixes it | Re-run full check suite, then PASS |
| `systematic-debugging` hits its 3-failed-fix stop | Pause, report to user, worktree untouched |

## Common Mistakes

- **Adding a second retry counter.** `systematic-debugging` already has one (3 failed fixes → question the architecture). A second counter here just creates two conflicting thresholds.
- **Wiping the worktree on failure.** Destroying in-progress work automatically is a hard-to-reverse action — always pause and let the user decide instead.
- **Re-running only the failing check.** A fix for one check can break another; always re-run the full suite before declaring PASS.
- **Letting a reviewer see a broken build.** PASS must be true before Stage 5 starts, no exceptions.
