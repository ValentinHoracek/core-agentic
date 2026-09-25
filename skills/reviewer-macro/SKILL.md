---
name: reviewer-macro
type: atomic
description: Use when a staged C# diff needs a focused review for Clean Architecture boundary violations, layer isolation, and ArchUnitNET-style structural rules
---

# Reviewer: Macro

## Overview

This skill reviews a staged C# diff for architectural and structural concerns only. Line-level code quality, security, and operations are out of scope. The skill is read-only. It never edits code.

## Contract

- **Inputs:**
  - `directory` (path of the git repository or worktree that has the staged diff, required)
  - `spec` (file path or the content of the spec, required)
  - `output` (file path, optional, default `REVIEW_MACRO.json`)
- **Output:** `REVIEW_MACRO.json` at the caller-given path. It contains the findings and a verdict in the JSON shape below.
- **Asks the user:** Never.

## When to Use

- A staged C# diff needs an architecture and layering review.
- The scope is narrow on purpose. Do not write findings about null safety, async patterns, security, or licensing, even if you see a problem.

## Review Scope

- Layer boundary violations. Examples: the domain layer references infrastructure, or presentation references data access directly.
- Dependency direction. Does the change add a dependency in the wrong direction (an inner layer that depends on an outer layer)?
- Responsibility placement. Is the new logic in the layer or class where it belongs? Or is it in a class that has a different responsibility?
- Consistency with the existing project structure. Compare with neighboring code in the same layer that was already reviewed.

## Process

1. Read the staged diff (`git diff --staged` in `directory`). Read enough of `spec` to understand the intent. Keep the total context below approximately 2,000 tokens.
2. For each changed file, find its architectural layer. Compare its new dependencies and references with the direction that its layer allows.
3. Do not propose fixes for issues outside this scope.
4. Set the verdict:
   - `pass`: no findings, or only `minor` findings.
   - `concerns`: one or more `major` findings, and no `blocker` finding.
   - `fail`: one or more `blocker` findings (for example, the domain layer calls a database client directly).
5. Write `output`. It must match this shape exactly:

```json
{
  "reviewer": "macro",
  "findings": [
    {
      "severity": "blocker | major | minor",
      "file": "relative/path.cs",
      "line": 42,
      "issue": "one-sentence description of the problem",
      "recommendation": "one-sentence description of the fix"
    }
  ],
  "verdict": "pass | concerns | fail"
}
```

An empty `findings` array with `"verdict": "pass"` is a valid, complete result.

## Common Mistakes

- **Findings about syntax or security.** These are out of scope. Report only architecture and layering.
- **A layer violation without its direction.** "Wrong layer" is not usable. "line 10: `Domain/Order.cs` references `Infrastructure.SqlClient` directly — domain must not depend on infrastructure" is usable.
- **A missing verdict field, or `fail` for only minor issues.**
