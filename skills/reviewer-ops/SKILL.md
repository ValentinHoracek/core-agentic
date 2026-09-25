---
name: reviewer-ops
type: atomic
description: Use when a staged diff needs a focused review for OWASP-class security issues, Docker non-root safety, and NuGet dependency/license risk
---

# Reviewer: Ops

## Overview

This skill reviews a staged diff for security and operational concerns only. Line-level code quality, architecture, and layering are out of scope. The skill is read-only. It never edits code.

## Contract

- **Inputs:**
  - `directory` (path of the git repository or worktree that has the staged diff, required)
  - `spec` (file path or the content of the spec, required)
  - `output` (file path, optional, default `REVIEW_OPS.json`)
- **Output:** `REVIEW_OPS.json` at the caller-given path. It contains the findings and a verdict in the JSON shape below.
- **Asks the user:** Never.

## When to Use

- A staged diff needs a security and operations review.
- The scope is narrow on purpose. Do not write findings about null safety, async patterns, or layering, even if you see a problem.

## Review Scope

- OWASP-class issues:
  - injection (SQL or command)
  - missing input validation on input that is not trusted
  - secrets or credentials in code
  - insecure deserialization
  - missing auth checks on new endpoints
- Docker: a new or changed `Dockerfile` that does one of these things:
  - It runs as root and not as a non-root user.
  - It exposes ports that are not necessary.
  - It puts secrets into image layers.
- NuGet dependencies: flag each new package reference in these cases:
  - The package is not familiar.
  - Its license is restrictive or viral for this codebase (for example, AGPL).
  - It does the same job as an existing dependency.

## Process

1. Read the staged diff (`git diff --staged` in `directory`). Read enough of `spec` to understand the intent. Keep the total context below approximately 2,000 tokens.
2. Examine each changed file against the Review Scope categories. Look carefully at new handling of external input, at changes to `Dockerfile` or `docker-compose.yml`, and at new package references in `.csproj` files.
3. Do not propose fixes for issues outside this scope.
4. Set the verdict:
   - `pass`: no findings, or only `minor` findings.
   - `concerns`: one or more `major` findings, and no `blocker` finding.
   - `fail`: one or more `blocker` findings (for example, SQL that the code builds by string concatenation from user input).
5. Write `output`. It must match this shape exactly:

```json
{
  "reviewer": "ops",
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

- **Findings about syntax or architecture.** These are out of scope. Report only security and operations.
- **No Dockerfile review when a Dockerfile changed.** When a change touches a Dockerfile, always look for a root user and for the `USER` directive.
- **A missing verdict field, or `fail` for only minor issues.**
