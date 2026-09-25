---
name: replan
type: atomic
description: Use when a drafted implementation plan needs one final review pass against its spec and recorded decisions before implementation begins
---

# Replan

## Overview

A final quality gate on a plan before implementation starts: checks that every requirement in the spec is covered by some task, that the plan doesn't contradict any recorded decision, and turns anything ambiguous into a direct question instead of a guess.

## Contract

- **Inputs:**
  - `spec` — file path; required
  - `plan` — file path; required
  - `decisions` — file path; optional
- **Output:** the `plan` file, updated in place — unchanged when nothing was found
- **Asks the user:** about each gap, contradiction or ambiguity it found, one question at a time

## When to Use

- A drafted plan exists and implementation has not started.
- Not for drafting a plan — this only reviews and refines an existing one.

## Process

1. Read `spec`, `decisions` (if given), and `plan` in full.
2. **Coverage check:** for each requirement and acceptance criterion in `spec`, find the task in `plan` that implements it. List any requirement with no matching task.
3. **Consistency check:** if `decisions` was given, confirm for each entry that no task in `plan` contradicts it (e.g. a decision to use Redis but a task that provisions a different store).
4. **Ambiguity check:** flag any task whose scope could reasonably be read two different ways.
5. If steps 2–4 found nothing: state that the plan passed review as-is, leave `plan` unchanged, and stop — no need to bother the user with a no-op confirmation.
6. If they found something: present the findings, then ask the user about them one at a time — one finding per question, multiple choice when the options are clear — and apply each answer directly to `plan`.
7. Never invent a task to fill a coverage gap without asking first — a missing requirement might mean the requirement changed, not that a task was forgotten.

## Quick Reference

| Check | Looks for |
|---|---|
| Coverage | Every `spec` requirement has a matching `plan` task |
| Consistency | No `plan` task contradicts a `decisions` entry |
| Ambiguity | No task scope readable two different ways |

## Common Mistakes

- **Silently patching gaps.** A missing task might mean the plan is wrong, or it might mean the requirement is already covered implicitly — always ask, don't assume.
- **Rubber-stamping.** "Looks fine" without actually walking every `spec` line item against the plan's tasks defeats the purpose.
- **Drafting a new plan instead of revising the given one.** This skill edits `plan`; it never replaces it.
- **Asking several questions at once.** One finding per question keeps each answer unambiguous.
