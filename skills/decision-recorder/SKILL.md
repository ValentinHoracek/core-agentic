---
name: decision-recorder
type: atomic
description: Use when planning just produced a significant architecture or technical decision that needs a record of its rationale before implementation begins
---

# Decision Recorder

## Overview

This skill records one technical decision as a short, structured entry in a decisions file. Future readers can then understand why the code has its current form, not only what the code does. Write one entry for each decision. Do not put unrelated decisions into one entry.

## Contract

- **Inputs:**
  - `decision` (value: the decision and its context, required)
  - `decisions` (file path, optional, default `DECISIONS.md`)
- **Output:** one entry added at the end of the `decisions` file. If the file does not exist, the skill creates it with a `# Decisions` heading.
- **Asks the user:** Never.

## When to Use

- Planning just made a choice that the code alone does not make clear. Examples: one library, pattern, or approach instead of another, a configuration value that is not the default, or a constraint that someone chose on purpose.
- Do not use it for a decision that had only one reasonable option. Record only choices that had a real trade-off.

## Process

1. Write the decision in one sentence.
2. Add an entry with this exact structure at the end of `decisions`. If the file does not exist, create it with a `# Decisions` heading.

```markdown
## D-00N: <short decision title>

**Decision:** <what was decided, one or two sentences>

**Context:** <what problem or question prompted this decision>

**Rationale:** <why this option was chosen>

**Alternatives Considered:** <other options and why they were rejected — at least one, even if "do nothing" or "the obvious default">

**Consequences:** <what this decision commits future work to, or rules out>
```

3. Give each entry the next number in sequence (`D-001`, `D-002`, ...). To find the next number, read the existing entries in `decisions` and add 1 to the highest number.
4. Write 1 to 3 sentences in each field. The entry is a record, not an essay.

## Quick Reference

| Field | One-line test |
|---|---|
| Decision | Can a reader repeat the decision correctly in one sentence? |
| Context | Does the field tell what caused the decision, not only repeat it? |
| Rationale | Does the field tell *why*, not only *what*? |
| Alternatives Considered | Does the field name one or more real alternatives? |
| Consequences | Does the field tell what the decision makes necessary or prevents in the future? |

## Common Mistakes

- **Recording the obvious.** If no real alternative existed, the choice is not a decision to record. It is only how the technology works.
- **Vague rationale.** "It's better" is not a rationale. "It avoids the N+1 query pattern the current ORM would otherwise produce" is a rationale.
- **No Consequences field.** An ADR-lite entry prevents a future reader from arguing the choice again. The Consequences field tells the reader if that argument is still necessary.
