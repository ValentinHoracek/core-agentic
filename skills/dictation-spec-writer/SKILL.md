---
name: dictation-spec-writer
type: atomic
description: Use when a raw stream-of-consciousness or speech-to-text markdown dump describing a feature needs to be restructured into a clean draft requirements document before refinement
---

# Dictation Spec Writer

## Overview

Takes an unstructured dictation dump (spoken stream-of-consciousness, transcribed to markdown) and restructures it into a clean, scannable first draft. Produces no new requirements — only reorganizes what was said. The draft is a starting point for later refinement, not a final spec.

## Contract

- **Inputs:**
  - `dictation` — file path; required
  - `confirm` — `yes | no`; optional, default `yes`
  - `output` — file path; optional, default `DRAFT.md`
- **Output:** `DRAFT.md` at the caller-given path — the five sections below, preceded by the auto-approval note when `confirm = no`
- **Asks the user:** only whether the draft captures their intent, and only when `confirm = yes`

## When to Use

- Input is a raw `.md` file of dictated, unstructured text describing a feature or change.
- Not for already-structured requirements documents — those need no restructuring.

## Process

1. Read `dictation` completely.
2. Extract and group content under exactly these five headings, in this order:
   - **Goal** — one or two sentences: what outcome is wanted and why.
   - **Requirements** — bullet list of concrete things the feature must do, phrased as plain statements (no jargon, no Given/When/Then).
   - **Edge Cases** — bullet list of unusual situations, error conditions, or boundary behavior mentioned or implied.
   - **Acceptance Criteria** — bullet list of checkable statements that would prove the feature works.
   - **Open Questions** — bullet list of anything ambiguous, contradictory, or left unsaid in the dictation.
3. Do not invent requirements that weren't stated or clearly implied. If something is unclear, put it in Open Questions rather than guessing.
4. If `confirm = yes`: present the draft to the user and ask them to confirm it captures their intent. Apply their corrections and ask again until they confirm. If `confirm = no`: skip the question and make this the first line of the file: `> Auto-approved without user confirmation — YYYY-MM-DD` (today's date), so there is a visible record that no person confirmed the draft.
5. Write the draft to `output`, then stop. The job ends at the file.

## Quick Reference

| Section | Contains |
|---|---|
| Goal | Outcome + why |
| Requirements | Concrete must-do statements |
| Edge Cases | Unusual/error/boundary situations |
| Acceptance Criteria | Checkable proof-of-done statements |
| Open Questions | Ambiguity, contradictions, gaps |

## Common Mistakes

- **Inventing structure that wasn't there.** If the dictation never mentions error handling, don't add an assumed edge case — leave it as an Open Question instead.
- **Skipping the confirmation when `confirm = yes`.** The draft is a translation, not a design — the user must confirm it. Only `confirm = no` skips this, and only with the auto-approval note in place.
- **Refining the draft.** Answering Open Questions or filling gaps is not this job — leave them listed.
- **Formatting as Given/When/Then.** This skill deliberately produces plain-language sections, not BDD scenarios.
