---
name: reviewer-micro
type: atomic
description: Use when a staged C# diff needs a focused review for Roslyn-level syntax issues, null safety, async/await correctness, and unnecessary allocations
---

# Reviewer: Micro

## Overview

Reviews a staged C# diff for line-level code quality only — not architecture and layering, and not security and operations. Read-only: never edits code.

## Contract

- **Inputs:**
  - `directory` — path of the git repository or worktree whose staged diff is reviewed; required
  - `spec` — file path or the spec's content; required
  - `output` — file path; optional, default `REVIEW_MICRO.json`
- **Output:** `REVIEW_MICRO.json` at the caller-given path — findings and a verdict, in the JSON shape below
- **Asks the user:** never

## When to Use

- A staged C# diff needs a line-level code quality review.
- Scope is deliberately narrow — do not comment on architecture, layering, security, or licensing here even if noticed.

## Review Scope

- Null safety: missing null checks, incorrect nullable annotations, unguarded dereferences.
- Async/await: `async void`, missing `ConfigureAwait` where relevant, blocking calls (`.Result`, `.Wait()`) on async code, unobserved tasks.
- Allocations: unnecessary boxing, LINQ in hot paths, avoidable large object allocations.
- General Roslyn-catchable issues: unused variables/usings, obvious analyzer-flaggable patterns.

## Process

1. Read the staged diff (`git diff --staged` in `directory`) and enough of `spec` to understand intent — keep total context under roughly 2,000 tokens.
2. Walk every changed line in scope (see Review Scope above). For each issue found, note the exact file and line number from the diff.
3. Do not propose fixes outside this scope, even if noticed.
4. Assign a verdict:
   - `pass`: no findings, or only `minor` findings.
   - `concerns`: at least one `major` finding, no `blocker`.
   - `fail`: at least one `blocker` finding (e.g. a null-safety bug that will crash at runtime).
5. Write `output`, matching exactly:

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

- **Commenting on architecture or security.** Out of scope — report only line-level code quality.
- **Vague findings.** "Async could be better" is not usable; "line 42: `.Result` on an async call blocks the thread pool, use `await` instead" is.
- **Skipping the verdict field, or picking `fail` for only minor issues.** `fail` is reserved for `blocker`-severity findings.
