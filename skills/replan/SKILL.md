---
name: replan
type: atomic
description: Use when an initial implementation plan has just been drafted and needs one final review pass against the spec and recorded decisions before implementation begins
---

# Replan

## Overview

A final quality gate on `PLAN.md` before implementation starts: checks that every requirement in `SPEC.md` is covered by some task, that the plan doesn't contradict anything in `DECISIONS.md`, and surfaces anything ambiguous as a direct question rather than silently guessing.

## When to Use

- Immediately after `superpowers:writing-plans` has produced a draft `PLAN.md`, before any implementation work starts.
- Not a substitute for `writing-plans` — this only reviews and refines an existing plan, it doesn't draft one from scratch.

## Process

1. Read `SPEC.md`, `DECISIONS.md`, and the draft `PLAN.md` in full.
2. **Coverage check:** for each requirement and acceptance criterion in `SPEC.md`, find the task in `PLAN.md` that implements it. List any requirement with no matching task.
3. **Consistency check:** for each entry in `DECISIONS.md`, confirm `PLAN.md`'s tasks don't contradict it (e.g. a decision to use Redis but a task that provisions a different store).
4. **Ambiguity check:** flag any task whose scope could reasonably be read two different ways.
5. If step 2–4 found nothing: state that the plan passed review as-is, and stop — no need to bother the user with a no-op confirmation.
6. If they found something: present the specific gaps/contradictions/ambiguities found, ask the user targeted questions to resolve each one (one at a time, per `superpowers:brainstorming`'s questioning style), then apply the resulting fixes directly to `PLAN.md`.
7. Never invent a task to fill a coverage gap without asking first — a missing requirement might mean the requirement changed, not that a task was forgotten.

## Quick Reference

| Check | Looks for |
|---|---|
| Coverage | Every SPEC.md requirement has a matching PLAN.md task |
| Consistency | No PLAN.md task contradicts a DECISIONS.md entry |
| Ambiguity | No task scope readable two different ways |

## Common Mistakes

- **Silently patching gaps.** A missing task might mean the plan is wrong, or it might mean the requirement is already covered implicitly — always ask, don't assume.
- **Rubber-stamping.** "Looks fine" without actually walking every SPEC.md line item against PLAN.md's tasks defeats the purpose of this stage.
- **Re-running writing-plans instead.** This skill revises an existing plan; it doesn't replace `superpowers:writing-plans`' job of drafting the first version.
