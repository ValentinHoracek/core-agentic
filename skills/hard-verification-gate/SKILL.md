---
name: hard-verification-gate
type: atomic
description: Use when a .NET solution directory needs a binary pass/fail check (build, tests, architecture rules, container run) recorded in a report file
---

# Hard Verification Gate

## Overview

This skill runs every binary check of a .NET solution. It writes one report with a PASS or FAIL verdict. The skill only observes. It never fixes, retries, or changes source files. The caller decides what to do after a FAIL.

## Contract

- **Inputs:**
  - `directory` (path to the solution root, required)
  - `output` (file path, optional, default `VERIFY.md`)
- **Output:** `VERIFY.md` at the caller-given path. It contains the verdict on line 1, a table of checks, and the output tail of each failed check.
- **Asks the user:** Never.

## When to Use

- Work in a directory is complete, according to its author. The work needs a go/no-go answer that real command output supports.
- The skill is not a debugger. It reports failures. It does not investigate them.

## Process

1. In `directory`, find the checks that apply:
   - **build** — `dotnet build`
   - **tests** — `dotnet test` (this includes architecture rules when they are ArchUnitNET tests in the solution)
   - **architecture rules** — each other configured architecture-rule command gets its own row
   - **container** — only if `directory` contains a compose file:
     1. Run `docker compose up --build --wait`. This command starts the services in the background and waits until they are running or healthy.
     2. Then always run `docker compose down`.
2. Run every check, also after one check fails. Keep the full output of each check.
3. A check that cannot run counts as `fail` (for example, no solution file, or `dotnet` missing). Its error message is its output tail.
4. Write `output` in this shape:

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

   Line 1 is exactly `PASS` when all checks passed. In all other cases, line 1 is exactly `FAIL`. When no check failed, `## Failures` contains `(none)`.
5. Stop. Do not try to fix a failure.

## Quick Reference

| Outcome | Line 1 of the report | Failures section |
|---|---|---|
| All checks pass | `PASS` | `(none)` |
| Any check fails | `FAIL` | One subsection per failed check with its output tail |

## Common Mistakes

- **Stopping at the first failure.** Run every check. The report lists every failure.
- **Fixing the failure.** This skill reports. Fixing is not its job.
- **Changing files in `directory`.** Build output (`bin/`, `obj/`) is expected. The skill never changes source, test, or config files.
- **Reporting a result without running the command.** Every row in the table comes from a command that ran in this invocation.
