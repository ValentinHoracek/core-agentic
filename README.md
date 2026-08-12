# core-agentic

Centralized AI governance, prompt skill directives, and multi-stage execution drivers for the `ValentinHoracek.Core` ecosystem. Ships as a Claude Code plugin — see `.claude-plugin/plugin.json`.

## Directory Taxonomy

* **`skills/`**: Every invokable unit. Ranges from single-purpose atomic capability prompts (e.g. decision recording, Roslyn code analysis, OWASP security checks) to multi-skill orchestrators ("heroes") that chain several skills into an end-to-end pipeline (e.g. `dotnet-solution-architect`). Each lives at `skills/<name>/SKILL.md`. Orchestrators are distinguished from atomic skills by a `type: hero` vs `type: atomic` frontmatter field, not by folder location.
* **`context/`**: Static domain knowledge, governance rules, and environmental constraints.
  * **`academic/`**: VŠE Business Informatics, Game Theory, OSINT, and Due Diligence frameworks.
  * **`enterprise/`**: Skoda DIGITAL, .NET 10, Blazor WASM, Docker, and Jenkins operational standards.

## Execution Mechanics

1. **Hero skills** (`type: hero`) act as execution managers — they hold state, manage Git Worktrees, and evaluate circuit-breaker rules, invoking atomic skills and reused `superpowers` skills via the Skill tool.
2. **Atomic skills** (`type: atomic`) act as pure cognitive functions — they accept inputs (e.g. `git diff` or `SPEC.md`) and produce structured outputs (`PLAN.md`, `REVIEW.json`).
3. **Context** provides domain boundaries — ensuring enterprise C# solutions stay isolated from academic research rules.
