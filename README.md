# typescript-senior-review

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Plugin Version](https://img.shields.io/badge/version-0.1.1-blue.svg)](https://github.com/TheMizeGuy/typescript-senior-review/releases)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-8A2BE2.svg)](https://claude.com/claude-code)

A [Claude Code](https://claude.com/claude-code) plugin that dispatches a senior TypeScript developer agent — running on the session model, always the strongest available Claude — to review your code across **18 angles** — quality, type-system correctness, architecture, maintainability, security, performance, error handling, testing, modern feature adoption, and ecosystem fit.

The reviewer is a fresh-context subagent with strict **read-only** tool access. Findings come back severity-tagged with concrete code rewrites, file:line locations, and citations. The orchestrator presents the report and asks which findings to apply — nothing is auto-fixed without your explicit selection.

## What it does

When you invoke the `review-typescript` skill (or ask Claude to review your TypeScript):

1. **Scope resolution** — single file, directory, git diff, staged, PR diff, or whole project
2. **Project context gathering** — `tsconfig.json` strictness flags, configured linter, framework, key deps
3. **Agent dispatch** — fresh-context subagent, running on the session model, with read-only tools (`Read`, `Grep`, `Glob`, `Bash`, optional MCP tools)
4. **The agent** reads your code, runs `tsc --noEmit` and your linter, reviews against the 18 angles, and returns findings in a strict format
5. **Present results** — the orchestrator displays the verbatim report and asks which findings you want applied

## Installation

```bash
# 1. Add this repo as a marketplace
claude plugin marketplace add https://github.com/TheMizeGuy/typescript-senior-review.git

# 2. Install the plugin
claude plugin install typescript-senior-review@typescript-senior-review

# 3. Restart Claude Code for the plugin to load
```

After restart, verify with `claude plugin list` and look for `typescript-senior-review@typescript-senior-review`.

## Usage

| Invocation | What it reviews |
|---|---|
| `/typescript-senior-review:review-typescript` | Uncommitted + staged TS changes (default) |
| `/typescript-senior-review:review-typescript staged` | Only staged changes |
| `/typescript-senior-review:review-typescript pr` | Diff vs `main`/`master` |
| `/typescript-senior-review:review-typescript src/` | All TS files in `src/` |
| `/typescript-senior-review:review-typescript src/auth/login.ts` | Single file |
| `/typescript-senior-review:review-typescript all` | Entire project (excluding `node_modules`, `dist`, `build`, `.next`, `out`, `coverage`) |

You can also ask Claude in plain English: "review my TypeScript", "check this file for issues", "audit my TS code", "pre-PR TypeScript review". The skill description triggers automatically.

## Walkthrough

A concrete run, start to finish:

1. Install the plugin (see Installation above) and restart Claude Code.
2. Finish a TypeScript change, then ask "review my auth changes before I open the PR" — or invoke directly: `/typescript-senior-review:review-typescript diff`.
3. The `review-typescript` skill resolves scope (for `diff`, that's `git diff --name-only HEAD` filtered to `.ts`/`.tsx`/`.cts`/`.mts`), reads your `tsconfig.json`, linter config, and `package.json`, then dispatches `senior-typescript-reviewer` with all of that context in one self-contained prompt.
4. The agent reads every file in scope, consults its knowledge base (if configured) and any GoodMem Learnings, runs `tsc --noEmit` and your linter when available, and returns a report shaped like:

   ```
   ## TypeScript Senior Review

   **Scope:** 3 files reviewed
   **Tooling run:** tsc=FAIL, eslint=PASS, biome=N/A
   **Findings:** 1 CRITICAL, 2 HIGH, 3 MEDIUM, 1 LOW, 0 NIT
   **Verdict:** fix HIGH+ before merge

   ### [CRITICAL] Security: webhook payload trusted via `as` cast, no runtime validation
   **File:** `src/webhooks/stripe.ts:31-38`
   **Issue:** ...
   **Why it matters:** ...
   **Current code:** ```ts ... ```
   **Suggested rework:** ```ts ... ```
   ```

5. Claude shows you the full report verbatim, then asks which findings to apply (`all CRITICAL`, `finding 3`, `everything in stripe.ts`, `skip`).
6. If you request fixes, Claude — not the reviewer agent — edits the named files, then re-runs `tsc --noEmit` and the linter on the touched files before confirming anything is fixed.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Skill doesn't trigger from a plain-English request | Phrasing didn't match the skill's trigger description | Invoke explicitly: `/typescript-senior-review:review-typescript [scope]` |
| "resolved file list is empty" | Scope argument resolved to nothing — no staged/uncommitted TS changes, or wrong path | Pass an explicit file or directory, or use `all` |
| Findings never cite a knowledge-base file | No local TypeScript reference docs at the path given to the agent | Point the agent at your own notes directory in the dispatch prompt, or ignore it — reviews still run without one |
| `Tooling run: tsc=N/A` or `eslint=N/A` | No `tsconfig.json` or linter config found from the project root | Add the missing config, or accept a static-only review |
| GoodMem-sourced context never appears in findings | GoodMem MCP isn't installed or configured for this session | Optional dependency — configure it, or ignore; the agent still runs without it |
| Plugin doesn't show up after cloning | Claude Code wasn't restarted, or marketplace/plugin weren't added in order | Re-run the install steps above in order, restart Claude Code, confirm with `claude plugin list` |

## The 18 review angles

| # | Angle | Examples |
|---|---|---|
| 1 | Type-system correctness | `any`, unsafe casts (`as unknown as`), `!` non-null, missed narrowing, weak generic constraints |
| 2 | Strict mode adherence | `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `useUnknownInCatchVariables` |
| 3 | Best practices | `interface` vs `type`, discriminated unions, `as const` + `satisfies`, branded types, exhaustiveness |
| 4 | Anti-patterns | Enums (vs `as const` objects), namespaces, `React.FC`, `Array.includes` narrowing failure, barrel files, `try/catch` swallowing, `@ts-ignore` over `@ts-expect-error` |
| 5 | Architecture | Module boundaries, circular imports, abstraction layers, dependency direction |
| 6 | Maintainability | Naming, complexity, dead code, comment quality |
| 7 | Compile-time performance | Wide unions, deep generics, recursive conditional types, missing return-type annotations on exports |
| 8 | Runtime performance | V8 anti-patterns, hidden class instability, allocation in loops, hot-path validation costs |
| 9 | Error handling | `throw` vs Result, catch narrowing of `unknown`, `Error.cause` chaining, swallowed errors, AbortSignal handling |
| 10 | Security | Unvalidated external input, prototype pollution, ReDoS, injection, secrets in source/logs, supply chain |
| 11 | Testing | Edge case coverage, mocks at boundaries only, type tests for public APIs, assertion strength |
| 12 | Modern feature adoption | `using` / `await using`, `satisfies`, `const T`, `NoInfer`, inferred type predicates, stage-3 decorators |
| 13 | Module system | `import type` discipline, `verbatimModuleSyntax`, ESM/CJS boundaries, package.json exports |
| 14 | Lint / format consistency | Disabled lint rules, suppression comments, inconsistent style |
| 15 | API design (libraries) | Public surface, breaking-change risk, brand types at boundaries, error types in return signatures |
| 16 | Documentation | JSDoc on public APIs, `@example`, `@deprecated`, README accuracy |
| 17 | Concurrency / async | `Promise.all` correctness (loses concurrent failures vs `allSettled`), race conditions |
| 18 | Resource lifecycle | DB connections, file handles, timers, listeners — leaked or properly disposed (`using`) |

## Output format

Each finding follows this exact template:

```
### [SEVERITY] [Category]: <one-line title>

**File:** `path/to/file.ts:42-58`

**Issue:** Plain-English explanation of what is wrong.

**Why it matters:** Concrete consequences — bug, security risk, performance hit, future maintenance pain.

**Current code:**
```ts
// minimal extract showing the problem
```

**Suggested rework:**
```ts
// concrete rewrite that fixes it, complete enough to apply verbatim
```
```

## Severity scale

| Severity | Meaning | Examples |
|---|---|---|
| **CRITICAL** | Bug, data loss, security vulnerability, will break in production | Unvalidated user input parsed as `any`, race condition, prototype pollution, SQL injection |
| **HIGH** | Will cause real problems soon | Type unsoundness in hot path, missing error handling on async, broken module boundary |
| **MEDIUM** | Should fix; quality / maintainability cost | `any` where `unknown` works, missing exhaustiveness, weak test |
| **LOW** | Nice to fix; consistency or polish | Naming inconsistency, missing JSDoc, could use newer TS feature |
| **NIT** | Personal preference / micro-optimization | Style choices, ordering, idiom variations |

## Components

| Type | Name | Purpose |
|---|---|---|
| Skill | `review-typescript` | User-invoked entry point; gathers scope and dispatches the agent |
| Agent | `senior-typescript-reviewer` | Reviewer, running on the session model, that reads code, runs tooling, returns findings |

## Tool access

The agent is **read-only by design**. It has:

| Tool | Purpose |
|---|---|
| `Read`, `Grep`, `Glob` | Read source files, search patterns |
| `Bash` | Run `tsc --noEmit`, `eslint`, `biome check` |
| `TodoWrite` | Track findings during long reviews |
| `WebSearch`, `WebFetch` | Verify against latest ecosystem state |
| `mcp__context7__resolve-library-id`, `mcp__context7__query-docs` (optional) | Live library docs if a [Context7](https://context7.com/) MCP server is configured |
| `mcp__goodmem__goodmem_memories_retrieve`, `mcp__goodmem__goodmem_memories_get` (optional) | Semantic memory search/fetch if a [GoodMem](https://goodmem.ai/) MCP server is configured |
| `mcp__plugin_serena_serena__*` (optional) | Symbol-aware code navigation (`find_symbol`, `find_referencing_symbols`, project memories) if a [Serena](https://github.com/oraios/serena) MCP server is configured |

It does **not** have `Edit`, `Write`, or `Agent` access. Findings are advisory — the orchestrator (your main Claude session) applies them based on your selection.

## Optional enhancements

The plugin works fine with just the built-in tools, but findings get better with:

- **[Context7](https://context7.com/) MCP** — live library docs for library-specific findings (zod, fastify, React, etc.).
- **[GoodMem](https://goodmem.ai/) MCP** — semantic memory search for cross-session learnings. If you have a Learnings space, pass its space ID in the dispatch prompt and the agent will query it.
- **[Serena](https://github.com/oraios/serena) MCP** — symbol-level navigation so the agent can trace callers/references instead of relying on grep alone.
- **A local TypeScript knowledge base** — e.g., a notes directory with reference docs the agent can cite. Any path works — just tell the agent where it is in the dispatch prompt.

None are required. The plugin is tested to work without them.

## Why a plugin instead of a skill?

This used to be tempting to build as a standalone skill (469-line instructions loaded inline into your conversation). Three reasons for the plugin / orchestrator + subagent pattern instead:

1. **Fresh context** — the reviewer runs in an isolated subagent that has never seen the conversation that wrote the code. No pattern blindness.
2. **Consistent behavior** — the reviewer inherits the session model rather than pinning to a specific one, so it always runs on whatever Claude tier is powering your session — no separate model configuration to maintain, and no risk of the reviewer stalling because a pinned model is unavailable.
3. **Read-only enforcement** — the agent has Read but not Edit/Write. Impossible to "accidentally fix" code mid-review.

## License

MIT. See [LICENSE](LICENSE).

## Credits

Built by [TheMizeGuy](https://github.com/TheMizeGuy). Backed by the [Claude Code](https://claude.com/claude-code) plugin system and the session model — always the strongest available Claude.
