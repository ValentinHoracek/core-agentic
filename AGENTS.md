# core-agentic — agent guide

Source repository of a Claude Code plugin (manifest: `.claude-plugin/plugin.json`).
This repo is where skills are written and edited. Skills are invoked as `core-agentic:<name>`.

## Layout

- `skills/<name>/SKILL.md` — one folder per skill, flat. No nesting, no shared folders.
- `.claude-plugin/plugin.json` — plugin name, description, version.

## Skill conventions

- Frontmatter has exactly three fields:
  - `name` — equal to the folder name.
  - `type` — `hero` or `atomic`.
  - `description` — starts with "Use when…" and states the trigger condition, not the workflow.
- Skill text is written in ASD-STE100 Strict (see the Check section).
- `hero` skills are the only place that chains skills, runs loops (fail → debug → re-check), interprets `mode`, and chooses artifact paths. Heroes reference other skills by qualified name (`core-agentic:replan`, `superpowers:writing-plans`), never by file path.
- `atomic` skills:
  - do one job;
  - may ask the user questions only about their own job;
  - never invoke or name another skill (no `superpowers:`, no `core-agentic:`, no skill names — not even their own, outside the frontmatter);
  - never mention a pipeline, stage, hero, orchestrator or `mode`;
  - write every output to a file at a caller-given path, else a default name — Markdown for documents a person reads, JSON for results a hero processes, other formats when a future skill needs them;
  - take every input as a file path or a plain value.

## Atomic contract

Every atomic skill has this section directly after `## Overview`:

```markdown
## Contract

- **Inputs:**
  - `<name>` (<file path | value>, <required | optional, default X>)
- **Output:** `<default file name>` at the caller-given path. <What it contains.>
- **Asks the user:** <Never. | When and what, own job only.>
```

A hero passes every input by its contract name and reads the output file, not the conversation, to decide the next step.

## Skills

| Skill | Type | Purpose |
|---|---|---|
| `dotnet-solution-architect` | hero | Raw .NET requirement → spec, plan, TDD implementation, verification, review, merge |
| `dictation-spec-writer` | atomic | Restructure a dictated/unstructured dump into a draft requirements doc |
| `decision-recorder` | atomic | Record an architecture/technical decision with its rationale |
| `replan` | atomic | Final review pass of a drafted plan against spec and decisions |
| `hard-verification-gate` | atomic | Run build, tests, architecture rules, container check; write a PASS/FAIL report |
| `reviewer-micro` | atomic | Staged C# diff: syntax, null safety, async/await, allocations |
| `reviewer-macro` | atomic | Staged C# diff: layer boundaries and structural rules |
| `reviewer-ops` | atomic | Staged diff: OWASP-class issues, Docker non-root, NuGet dependency/license risk |

## Check

Run from the repo root after changing any atomic skill. The output must be empty:

```bash
P='superpowers:|core-agentic:|Stage [0-9]|orchestrator|pipeline|\bmode\b|dotnet-solution-architect|dictation-spec-writer|hard-verification-gate|decision-recorder|replan|reviewer-(micro|macro|ops)'
for f in $(grep -l '^type: atomic' skills/*/SKILL.md); do grep -nHE "$P" "$f" | grep -vE ':[0-9]+:name: '; done
```

Every `SKILL.md` must also have 0 hard ASD-STE100 violations:

```bash
for f in skills/*/SKILL.md; do python3 ~/.claude/skills/asd-ste100/scripts/ste-lint.py --json "$f" | python3 -c "import json,sys; h=json.load(sys.stdin)['hard_count']; print('$f', h) if h else None"; done
```

Extend `P` with the name of every new skill.

## External dependencies

- `asd-ste100` — user skill in `~/.claude/skills`. `dotnet-solution-architect` uses it in Stage 1 to rewrite the requirements draft in Simplified Technical English. Without it, the pipeline skips that step.

## Editing rules

- Create or change skills with `superpowers:writing-skills`.
- Keep this table in sync when adding, renaming or removing a skill.
- Bump `version` in `.claude-plugin/plugin.json` for every release.
