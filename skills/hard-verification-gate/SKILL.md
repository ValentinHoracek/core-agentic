---
name: hard-verification-gate
type: atomic
description: Use when a .NET solution directory needs a binary pass/fail check (build, tests, architecture rules, container run) recorded in a report file
---

# Hard Verification Gate

## Overview

Runs every binary check of a .NET solution and writes one report with a PASS or FAIL verdict. It only observes: it never fixes, retries, or changes source files. What happens after a FAIL is the caller's decision.

## Contract

- **Inputs:**
  - `directory` — path to the solution root; required
  - `output` — file path; optional, default `VERIFY.md`
- **Output:** `VERIFY.md` at the caller-given path — verdict on line 1, a table of checks, the output tail of each failed check
- **Asks the user:** never

## When to Use

- Work in a directory claims to be done and needs a go/no-go answer backed by real command output.
- Not a debugger: it reports failures, it does not investigate them.

## Process

1. In `directory`, determine the checks that apply:
   - **build** — `dotnet build`
   - **tests** — `dotnet test` (includes architecture rules when they are ArchUnitNET tests in the solution)
   - **architecture rules** — any other configured architecture-rule command gets its own row
   - **container** — `docker compose up --build --wait` (starts the services detached and waits until they are running or healthy), then always `docker compose down`; only if `directory` contains a compose file
2. Run every check, even after one fails. Capture the full output of each. A check that cannot run (e.g. no solution file, `dotnet` missing) counts as `fail`; its error message is its output tail.
3. Write `output` in this shape:

   ````markdown
   FAIL

   | Check | Command | Result |
   |---|---|---|
   | build | `dotnet build` | pass |
   | tests | `dotnet test` | fail |

   ## Failures

   ### tests

   ```
   <last ~50 lines of the failed command's output>
   ```
   ````

   Line 1 is exactly `PASS` when every check passed, else exactly `FAIL`. With no failures, `## Failures` contains `(none)`.
4. Stop. Do not attempt a fix.

## Quick Reference

| Outcome | Line 1 of the report | Failures section |
|---|---|---|
| All checks pass | `PASS` | `(none)` |
| Any check fails | `FAIL` | One subsection per failed check with its output tail |

## Common Mistakes

- **Stopping at the first failure.** Run every check; the report lists every failure.
- **Fixing the failure.** This skill reports. Fixing is not its job.
- **Changing files in `directory`.** Build output (`bin/`, `obj/`) is expected; source, test and config files are never modified.
- **Reporting a result without running the command.** Every row in the table comes from a command run in this invocation.
