---
name: reviewer-ops
type: atomic
description: Use when a staged diff needs a focused review for OWASP-class security issues, Docker non-root safety, and NuGet dependency/license risk
---

# Reviewer: Ops

## Overview

Reviews a staged diff for security and operational concerns only — not line-level code quality, and not architecture and layering. Read-only: never edits code.

## Contract

- **Inputs:**
  - `directory` — path of the git repository or worktree whose staged diff is reviewed; required
  - `spec` — file path or the spec's content; required
  - `output` — file path; optional, default `REVIEW_OPS.json`
- **Output:** `REVIEW_OPS.json` at the caller-given path — findings and a verdict, in the JSON shape below
- **Asks the user:** never

## When to Use

- A staged diff needs a security and operations review.
- Scope is deliberately narrow — do not comment on null safety, async patterns, or layering here even if noticed.

## Review Scope

- OWASP-class issues: injection (SQL/command), missing input validation on untrusted input, secrets/credentials in code, insecure deserialization, missing auth checks on new endpoints.
- Docker: any new/changed `Dockerfile` running as root instead of a non-root user, exposing unnecessary ports, or baking secrets into image layers.
- NuGet dependencies: any new package reference — flag if it's unfamiliar, has a restrictive/viral license (e.g. AGPL) for this codebase, or duplicates functionality of an existing dependency.

## Process

1. Read the staged diff (`git diff --staged` in `directory`) and enough of `spec` to understand intent — keep total context under roughly 2,000 tokens.
2. Check every changed file against the Review Scope categories above. Pay particular attention to any new external input handling, new `Dockerfile`/`docker-compose.yml` changes, and any `.csproj` package reference additions.
3. Do not propose fixes outside this scope, even if noticed.
4. Assign a verdict:
   - `pass`: no findings, or only `minor` findings.
   - `concerns`: at least one `major` finding, no `blocker`.
   - `fail`: at least one `blocker` finding (e.g. SQL built via string concatenation from user input).
5. Write `output`, matching exactly:

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

- **Commenting on syntax or architecture.** Out of scope — report only security and operations.
- **Missing Dockerfile review when a Dockerfile changed.** Always check for a root user / `USER` directive when any Dockerfile is touched.
- **Skipping the verdict field, or picking `fail` for only minor issues.**
