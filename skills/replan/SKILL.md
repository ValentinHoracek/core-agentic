---
name: replan
type: atomic
description: Use when a drafted implementation plan needs one final review pass against its spec and recorded decisions before implementation begins
---

# Replan

## Overview

This skill is a final quality gate on a plan before implementation begins. It does three things:

- It makes sure that some task covers each requirement in the spec.
- It makes sure that the plan does not contradict a recorded decision.
- It changes each ambiguous item into a direct question to the user. It does not guess.

## Contract

- **Inputs:**
  - `spec` (file path, required)
  - `plan` (file path, required)
  - `decisions` (file path, optional)
- **Output:** the `plan` file, updated in place. When the skill finds nothing, the file does not change.
- **Asks the user:** about each gap, contradiction, or ambiguity that it finds, one question at a time.

## When to Use

- A drafted plan exists, and implementation did not begin.
- Do not use it to draft a plan. It only reviews and improves a plan that exists.

## Process

1. Read all of `spec`, `decisions` (if the caller gave it), and `plan`.
2. **Coverage check:** for each requirement and acceptance criterion in `spec`, find the task in `plan` that implements it. List each requirement that has no matching task.
3. **Consistency check:** if the caller gave `decisions`, examine each entry. Make sure that no task in `plan` contradicts it. Example: a decision chooses Redis, but a task sets up a different store.
4. **Ambiguity check:** flag each task whose scope a reader can reasonably understand in two different ways.
5. If steps 2–4 found nothing: tell the user that the plan passed the review without changes. Do not change `plan`. Stop. Do not ask the user to approve a result that changes nothing.
6. If steps 2–4 found something: show the findings. Then ask the user about them one at a time. Put one finding in each question. Use multiple choice when the options are clear. Apply each answer directly to `plan`.
7. Never add a task to fill a coverage gap without asking first. A missing requirement can mean that the requirement changed, not that someone forgot a task.

## Quick Reference

| Check | Looks for |
|---|---|
| Coverage | Every `spec` requirement has a matching `plan` task |
| Consistency | No `plan` task contradicts a `decisions` entry |
| Ambiguity | No task scope readable two different ways |

## Common Mistakes

- **Fixing gaps without asking.** A missing task can mean that the plan is wrong. It can also mean that the plan already covers the requirement implicitly. Always ask. Do not assume.
- **Approval without checking.** "Looks fine" without comparing each `spec` item with the tasks in the plan makes the review useless.
- **Drafting a new plan instead of revising the given one.** This skill edits `plan`. It never replaces it.
- **Several questions at once.** Put one finding in each question, so that each answer has only one meaning.
