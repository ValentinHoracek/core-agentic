---
name: dictation-spec-writer
type: atomic
description: Use when a raw stream-of-consciousness or speech-to-text markdown dump describes a feature and needs restructuring into a clean draft requirements document before refinement
---

# Dictation Spec Writer

## Overview

This skill takes an unstructured dictation dump (spoken stream-of-consciousness, transcribed to markdown). It puts the content into a clean first draft that is easy to scan. It adds no new requirements. It only puts what the speaker said into a new order. The draft is a start point for later refinement. It is not a final spec.

## Contract

- **Inputs:**
  - `dictation` (file path, required)
  - `confirm` (`yes | no`, optional, default `yes`)
  - `output` (file path, optional, default `DRAFT.md`)
- **Output:** `DRAFT.md` at the caller-given path. It contains the five sections below. When `confirm = no`, the auto-approval note comes before them.
- **Asks the user:** One question only: does the draft show the intent of the user? The skill asks it only when `confirm = yes`.

## When to Use

- The input is a raw `.md` file of dictated, unstructured text about a feature or change.
- Do not use it for a requirements document that already has a structure. That document needs no restructuring.

## Process

1. Read all of `dictation`.
2. Put the content under exactly these five headings, in this order:
   - **Goal** — one or two sentences: the outcome that the speaker wants, and why.
   - **Requirements** — a bullet list of concrete things that the feature must do. Write them as plain statements (no jargon, no Given/When/Then).
   - **Edge Cases** — a bullet list of unusual situations, error conditions, or boundary behavior that the dictation mentions or implies.
   - **Acceptance Criteria** — a bullet list of statements that a person can check to prove that the feature works.
   - **Open Questions** — a bullet list of each item that is ambiguous, contradictory, or missing in the dictation.
3. Do not add requirements that the dictation did not state or clearly imply. If an item is not clear, put it in Open Questions. Do not guess.
4. If `confirm = yes`: show the draft to the user. Ask the user if the draft shows their intent. Apply their corrections. Ask again until the user says yes.
5. If `confirm = no`: do not ask the question. Make this the first line of the file: `> Auto-approved without user confirmation — YYYY-MM-DD` (the date of today). This line is a visible record that no person approved the draft.
6. Write the draft to `output`. Then stop. The job ends at the file.

## Quick Reference

| Section | Contains |
|---|---|
| Goal | Outcome + why |
| Requirements | Concrete must-do statements |
| Edge Cases | Unusual/error/boundary situations |
| Acceptance Criteria | Checkable proof-of-done statements |
| Open Questions | Ambiguity, contradictions, gaps |

## Common Mistakes

- **Structure that the dictation did not have.** If the dictation never mentions error handling, do not add an edge case that you assume. Put the item in Open Questions.
- **No approval question when `confirm = yes`.** The draft is a translation, not a design. The user must approve it. Only `confirm = no` removes the question, and only with the auto-approval note in place.
- **Refining the draft.** It is not the job of this skill to answer Open Questions or to fill gaps. Keep them in the list.
- **Given/When/Then format.** This skill writes plain-language sections on purpose, not BDD scenarios.
