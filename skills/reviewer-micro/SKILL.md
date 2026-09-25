---
name: reviewer-micro
type: atomic
description: Use when a staged C# diff needs a focused review for Roslyn-level syntax issues, null safety, async/await correctness, and unnecessary allocations
---

# Reviewer: Micro

## Overview

This skill reviews a staged C# diff for line-level code quality only. Architecture, layering, security, and operations are out of scope. The skill is read-only. It never edits code.

## Contract

- **Inputs:**
  - `directory` (path of the git repository or worktree that has the staged diff, required)
  - `spec` (file path or the content of the spec, required)
  - `output` (file path, optional, default `REVIEW_MICRO.json`)
- **Output:** `REVIEW_MICRO.json` at the caller-given path. It contains the findings and a verdict in the JSON shape below.
- **Asks the user:** Never.

## When to Use

- A staged C# diff needs a line-level code quality review.
- The scope is narrow on purpose. Do not write findings about architecture, layering, security, or licensing, even if you see a problem.

## Review Scope

- Null safety: missing null checks, incorrect nullable annotations, and dereferences without a guard.
- Async/await: `async void`, missing `ConfigureAwait` where it applies, blocking calls (`.Result`, `.Wait()`) on async code, and tasks that nothing observes.
- Allocations: boxing that is not necessary, LINQ in hot paths, and large object allocations that you can avoid.
- Other issues that Roslyn can find: unused variables or usings, and patterns that an analyzer flags.

## Process

1. Read the staged diff (`git diff --staged` in `directory`). Read enough of `spec` to understand the intent. Keep the total context below approximately 2,000 tokens.
2. Examine each changed line that is in scope (see Review Scope). For each issue, record the exact file and line number from the diff.
3. Do not propose fixes for issues outside this scope.
4. Set the verdict:
   - `pass`: no findings, or only `minor` findings.
   - `concerns`: one or more `major` findings, and no `blocker` finding.
   - `fail`: one or more `blocker` findings (for example, a null-safety bug that causes a crash at runtime).
5. Write `output`. It must match this shape exactly:

```json
{
  "reviewer": "micro",
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

- **Findings about architecture or security.** These are out of scope. Report only line-level code quality.
- **Vague findings.** "Async could be better" is not usable. "line 42: `.Result` on an async call blocks the thread pool, use `await` instead" is usable.
- **A missing verdict field, or `fail` for only minor issues.** Use `fail` only for `blocker`-severity findings.
