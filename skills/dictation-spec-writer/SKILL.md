---
name: dictation-spec-writer
type: atomic
description: Use when a raw stream-of-consciousness or speech-to-text markdown dump describing a feature needs to be restructured into a clean draft requirements document before refinement
---

# Dictation Spec Writer

## Overview

Takes an unstructured dictation dump (spoken stream-of-consciousness, transcribed to markdown) and restructures it into a clean, scannable first draft. Produces no new requirements — only reorganizes what was said. Refinement and gap-filling happen afterward via `superpowers:brainstorming`.

## When to Use

- Input is a raw `.md` file of dictated, unstructured text describing a feature or change.
- Not for already-structured requirements documents — skip straight to `superpowers:brainstorming` for those.

## Process

1. Read the raw dictation file completely.
2. Extract and group content under exactly these five headings, in this order:
   - **Goal** — one or two sentences: what outcome is wanted and why.
   - **Requirements** — bullet list of concrete things the feature must do, phrased as plain statements (no jargon, no Given/When/Then).
   - **Edge Cases** — bullet list of unusual situations, error conditions, or boundary behavior mentioned or implied.
   - **Acceptance Criteria** — bullet list of checkable statements that would prove the feature works.
   - **Open Questions** — bullet list of anything ambiguous, contradictory, or left unsaid in the dictation.
3. Do not invent requirements that weren't stated or clearly implied. If something is unclear, put it in Open Questions rather than guessing.
4. Present the restructured draft to the user and ask them to confirm it captures their intent before proceeding.
5. Once confirmed, **REQUIRED SUB-SKILL:** use `superpowers:brainstorming` with the restructured draft as its starting context, letting it run its normal one-question-at-a-time clarification loop against the Open Questions and any gaps it finds. `brainstorming`'s output becomes the final `SPEC.md`.

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
- **Skipping the confirmation step.** The restructured draft is a translation, not a design — the user must confirm it before `brainstorming` builds on it.
- **Formatting as Given/When/Then.** This skill deliberately produces plain-language sections, not BDD scenarios.
