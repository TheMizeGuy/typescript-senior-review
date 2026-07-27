# Changelog

## [Unreleased]

### Changed

- Retargeted the reviewer's TypeScript 7 standard from the retired preview channel to the GA release.
  TypeScript 7.0 shipped 2026-07-08 as `typescript@7` with `tsc` as its only binary; the
  `@typescript/native-preview` package and its `tsgo` binary were abandoned after
  `7.0.0-dev.20260707.2` (2026-07-07). The reviewer previously ran `npx tsgo --noEmit` and recorded a
  MEDIUM finding urging every reviewed project to install that dead package.
  - Gate is now `node node_modules/ts7/bin/tsc --noEmit` (GA TS7 under the `ts7` npm alias,
    `"ts7": "npm:typescript@~7.0.2"`); the fallback is `node node_modules/typescript/bin/tsc --noEmit`.
    Both by explicit path -- each package declares a `tsc` bin and npm's link order on the collision is
    not guaranteed, so a bare `tsc` can run the wrong compiler.
  - The dual-compiler rationale is corrected: `typescript` 6.x stays because TS 7.0 ships no
    programmatic compiler API (7.1 is expected to), so ts-jest / typescript-eslint / Stryker / tsserver
    still need it -- not because the gate has a different binary name.
  - Finding `tsgo` or `@typescript/native-preview` in a reviewed project is now itself a HIGH finding.
  - Report tooling line is `typecheck(ts7|ts6)=...`; skill pre-flight gate detection gains a
    "retired tsgo channel" state; README section retitled "TypeScript 7 standard"; agent, skill, and
    manifests (keyword `tsgo` -> `typescript-7`) updated to match.
  - The 0.2.0 entry below is left as written and is superseded by this entry.

## [0.2.0] - 2026-07-06

### Changed

- Adopted the tsgo / TypeScript 7 standard: `tsgo --noEmit` (`@typescript/native-preview`) is the one typecheck gate the reviewer runs and the gate suggested reworks must pass; `tsc --noEmit` demoted to a per-review fallback on projects without tsgo, where the reviewer now records a MEDIUM adoption finding. tsc stays only in the emit/tooling lane (`.d.ts` builds, ts-jest, Stryker) -- dual-compiler, not a swap.
- Report tooling line renamed to `typecheck(tsgo|tsc)=...` in the agent template, skill acceptance criteria, and README example.
- Skill pre-flight now detects the project's typecheck gate (tsgo / tsc-only / none) and passes it in PROJECT CONTEXT.
- Knowledge-base tables now list reference files 17 and 18 (TS 6/7 Migration Playbook; TS7 Native Compiler and Tooling Compatibility); added a "Typecheck gate / TS7 readiness" scope row.
- README: added a "TypeScript 7 / tsgo standard" section and a tsgo-adoption troubleshooting row.

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
