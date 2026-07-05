# Changelog

## [0.1.1] - 2026-07-05

### Changed

- Removed the `model: fable` pin from the `senior-typescript-reviewer` agent. Agents in this plugin no longer pin a model — they inherit the session model (always the strongest available Claude), so the reviewer never stalls if a specific model becomes unavailable.
- Added a per-angle evidence bar to the agent's system prompt: a finding must clear a stated evidence threshold for its angle (compiler/linter output, a traced attacker-controlled input, a named throwing call, etc.) or it gets downgraded or dropped.
- Added a worked example finding to the agent's system prompt showing the exact bar every reported finding must clear.
- Added an 8-point self-verification checklist the agent runs before emitting its report (ground truth re-read, failure-scenario check, rework type-check, evidence bar, severity audit, citation check, count arithmetic, tone check).
- Added explicit acceptance criteria to the `review-typescript` skill's dispatch prompt, plus a gate step where the orchestrator checks the returned report against those criteria and re-dispatches once on failure before presenting results.
- Added a post-fix verification step: after applying findings, the orchestrator now runs `tsc --noEmit` and the configured linter on the touched files and shows the output before reporting fixes as applied.
- Added an "Execution mode" note: the reviewer always inherits the session model; if the session model is already the strongest tier and the review is small or time-critical, the orchestrator may run it inline instead of dispatching a subagent.
- Refreshed README: accurate tool-access table (previous table referenced non-existent MCP tool names and omitted Serena), added a worked Walkthrough section and a Troubleshooting table, removed the fixed-model badge and all "Fable 5" references.

## 2026-07-02

- Synced plugin content from the maintained source: Fable 5 model line throughout, agent fan-out bounded by the dispatch budget (≤10/wave), refreshed knowledge-base counts, removed stale references.

All notable changes to the `typescript-senior-review` plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-04-14

### Added

- Initial public release of the `typescript-senior-review` plugin
- `review-typescript` skill: scope resolution (file / directory / `staged` / `diff` / `pr` / `all`), project context gathering (`tsconfig.json` strictness flags, configured linter, framework, key dependencies), and dispatch of the reviewer subagent
- `senior-typescript-reviewer` agent: Opus 4.6, read-only tools (Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, optional Context7 MCP, optional GoodMem MCP), strict 18-angle review process
- 18 review angles spanning type-system correctness, strict mode adherence, best practices, anti-patterns, architecture, maintainability, compile-time and runtime performance, error handling, security, testing, modern feature adoption, module system, lint consistency, API design, documentation, concurrency, and resource lifecycle
- 5-level severity scale (CRITICAL / HIGH / MEDIUM / LOW / NIT) with concrete code rewrite blocks on every finding
- Marketplace manifest at `.claude-plugin/marketplace.json` for one-step `claude plugin marketplace add` install
- MIT license

[0.1.1]: https://github.com/TheMizeGuy/typescript-senior-review/releases/tag/v0.1.1
[0.1.0]: https://github.com/TheMizeGuy/typescript-senior-review/releases/tag/v0.1.0
