---
name: decision-recorder
type: atomic
description: Use when a significant architecture or technical decision has just been made during planning and needs to be captured with its rationale before implementation begins
---

# Decision Recorder

## Overview

Captures one technical decision as a short, structured entry in `DECISIONS.md`, so future readers (including a future you) understand why the code looks the way it does, not just what it does. One entry per decision — don't batch unrelated decisions into one entry.

## When to Use

- A choice was just made during Stage 2 planning that isn't obvious from the code alone (e.g. picking one library, pattern, or approach over another; a non-default configuration choice; a deliberate constraint).
- Not for decisions with only one reasonable option — record only choices that involved a real trade-off.

## Process

1. Identify the decision in one sentence.
2. Write an entry using this exact structure, appended to `DECISIONS.md` (create the file with a `# Decisions` heading if it doesn't exist yet):

```markdown
## D-00N: <short decision title>

**Decision:** <what was decided, one or two sentences>

**Context:** <what problem or question prompted this decision>

**Rationale:** <why this option was chosen>

**Alternatives Considered:** <other options and why they were rejected — at least one, even if "do nothing" or "the obvious default">

**Consequences:** <what this decision commits future work to, or rules out>
```

3. Number entries sequentially (`D-001`, `D-002`, ...) by scanning existing entries in `DECISIONS.md` and incrementing.
4. Keep each field to 1–3 sentences. This is a record, not an essay.

## Quick Reference

| Field | One-line test |
|---|---|
| Decision | Could someone quote this back accurately in one sentence? |
| Context | Does it explain what prompted the decision, not just restate it? |
| Rationale | Does it say *why*, not just *what*? |
| Alternatives Considered | Is there at least one real alternative named? |
| Consequences | Does it say what this locks in or rules out going forward? |

## Common Mistakes

- **Recording the obvious.** If there was no real alternative, it's not a decision worth recording — it's just how the tech works.
- **Vague rationale.** "It's better" is not a rationale. "It avoids the N+1 query pattern the current ORM would otherwise produce" is.
- **Skipping Consequences.** The point of an ADR-lite entry is to save a future reader from re-litigating the choice — Consequences is what tells them whether that's still necessary.
