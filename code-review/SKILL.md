---
name: code-review
description: Perform a comprehensive code review across four dimensions — architecture, semantic quality, module ownership, and error handling — producing a unified scored report with actionable findings.
---

# Code Review

You are now in **Code Review mode**. Conduct a thorough review across four dimensions: architecture design, semantic code quality, module ownership health, and error handling patterns. This is a read-only analysis — produce a report with findings and recommendations, do not modify any files.

## Review Scope

If the user specifies a target (package, module, directory, or file set), focus there. Otherwise review the current working directory as a whole.

## What to Read First

Before reviewing, always read in this order:
1. `README.md` and `CLAUDE.md` in scope — understand stated conventions before flagging violations
2. Key entry points (`index.ts`, `main.ts`, etc.) and type definitions
3. `package.json` — dependencies, scripts, package boundaries
4. Test files co-located with source
5. All interfaces and types consumed or exported by the target

---

## Review Dimensions

### Dimension 1 — Architecture

Evaluate system-level design decisions.

- **Separation of concerns**: Are layers (transport, business logic, persistence, scheduling) clearly separated?
- **Type safety**: Is `unknown` preferred over `any`? Are interfaces well-defined? Do type unions or enums contain variants that are never produced at runtime (declared intent vs. implemented reality)?
- **Testability**: Can units be tested in isolation? Are dependencies injectable?
- **Scalability**: Are there N+1 patterns, unbounded lists, missing caching, or synchronous blocking calls?
- **Observability**: Is there adequate logging, tracing, or metrics instrumentation?
- **Security posture**: Auth gaps, secret handling, input validation, data exposure risks (reference OWASP Top 10).
- **Developer experience**: Is the API surface intuitive? Are entry points easy to discover?

### Dimension 2 — Semantic Quality

Evaluate whether code means what it says at the method and module level.

- **Semantic accuracy**: Does each method name describe ALL its side effects? Flag `get*` / `build*` methods that mutate state, and `initialize()` / `setup()` with observable side effects beyond the object itself. Flag file or module names that imply a pattern or protocol (e.g. "machine", "factory", "registry") without implementing it — names are contracts with the reader.
- **Single responsibility**: Does each method have exactly one reason to change? Flag AND-descriptions ("persists state AND emits checkpoint"). Flag abstraction layers that wrap a single call without adding behavior, validation, or semantic narrowing — a wrapper that does nothing except add an extra name is noise, not structure.
- **Caller-perspective predictability**: Can a caller predict all behavior from the call site alone? Flag hidden async work, debounced side effects buried in unrelated methods, and implicit ordering requirements. Flag mixed mutability styles within a single function — creating an immutable copy then mutating it directly destroys the reader's mental model of data flow.
- **Coupling**: Does each method touch exactly one layer? Flag storage persistence calling checkpoint notification, business logic inside infrastructure code.
- **Async / side-effect transparency**: Are all async side effects explicitly awaited or registered? Flag fire-and-forget calls, closures capturing unassigned variables, and silently dropped work on early termination.
- **Idempotency guards**: Are expensive operations protected against redundant execution? Flag O(n) idempotency checks (e.g., `JSON.stringify` comparisons) where O(1) counters suffice.

### Dimension 3 — Module Ownership

Audit whether code lives where it is owned, not where it was convenient to put it.

**Classify every export / file in the target as one of:**
- `single-owner` — imported by exactly one module
- `multi-owner` — imported by two or more distinct modules
- `dead` — imported by nobody

**Identify these patterns:**
- Pure re-export files (entire body is `export { ... } from '...'`)
- Exports that are never imported anywhere (dead code)
- A `common/` or `shared/` layer co-mingling truly shared code with module-specific code
- Identity functions (return input unchanged, no validation) and single-use one-liners
- Alias re-exports (`export const newName = originalName`) with no additional value
- Files whose existence cost (import path indirection, reader context-switch) exceeds their content value — a 5-line class or a single factory function in isolation rarely justifies a dedicated file unless it marks a true semantic boundary
- Circular or overly tight coupling between packages

### Dimension 4 — Error Handling

Evaluate how errors are surfaced, propagated, and communicated.

- **Swallowed errors**: Empty `catch` blocks or `catch` that only logs without re-throwing or recovering.
- **Error classification**: Are user errors (validation, 4xx) distinguished from system errors (infrastructure, 5xx)? Are error types/classes used or is everything a generic `Error`?
- **Async error propagation**: Are all promise rejections handled? Are there floating promises with no `.catch()`?
- **Boundary handling**: At system boundaries (HTTP handlers, queue consumers, scheduled jobs), is there a top-level error boundary that prevents silent failures?
- **Error messages**: Do error messages include enough context (what operation failed, with what input) to be actionable in production?
- **Cleanup on failure**: Do error paths clean up resources (connections, locks, temp files) or leave them dangling?

---

## Report Format

Produce the review as a structured Markdown report. Every section is required. Cite `file:line` for every claim.

```
# Code Review — <Target Name>
Date: <today's date>

## 1. Executive Summary
<3–5 sentences: what this system does, overall health across all four dimensions, and the single most important thing to fix.>

## 2. Scorecard

| Dimension                  | Score | Notes |
|----------------------------|-------|-------|
| Separation of Concerns     | /5    | |
| Type Safety                | /5    | |
| Testability                | /5    | |
| Scalability                | /5    | |
| Observability              | /5    | |
| Security Posture           | /5    | |
| Developer Experience       | /5    | |
| Semantic Accuracy          | /5    | |
| Single Responsibility      | /5    | |
| Caller Predictability      | /5    | |
| Module Ownership Clarity   | /5    | |
| Error Handling             | /5    | |
| **Overall**                | **/60** | |

## 3. Findings

Each finding uses this template:

### F1 — <Short title>
**Dimension**: Architecture | Semantic | Module Ownership | Error Handling
**Severity**: Critical | High | Medium | Low
**Location**: `file:line`
**Problem**: What the code says vs. what it actually does, or why it is a risk.
**Recommendation**: Concrete fix, rename, decomposition, or migration path.

(repeat for each finding, ordered by severity)

## 4. Module Ownership Map

| Export / File | Consumers | Classification |
|---|---|---|
| `SymbolName` | `module-a` only | single-owner |
| `SharedUtil` | `module-a`, `module-b` | multi-owner |
| `UnusedType` | none | dead |

Only include entries where the classification reveals a problem or a notable pattern.

## 5. Decomposition Candidates
List methods or modules that should be split:
- `oldMethod()` → `newMethodA()` + `newMethodB()`

Omit this section if none found.

## 6. Positive Patterns
Bullet list of genuine design wins, citing file:line. Be specific — not "good types" but "IFoo interface at foo.ts:12 makes the contract explicit and prevents null misuse."

## 7. Recommended Action Plan

| Priority | Finding | Action | Effort |
|----------|---------|--------|--------|
| 1 | F1 | | Quick Win / Medium / Large |

Group by effort tier, order by impact within each tier.

## 8. Open Questions
Assumptions made during review, or clarifications needed before implementing recommendations.
```

---

## Scoring Guide

| Score | Meaning |
|-------|---------|
| 5 | Exemplary — sets a high bar |
| 4 | Good — minor gaps |
| 3 | Adequate — notable issues present |
| 2 | Weak — significant problems |
| 1 | Poor — needs immediate attention |
| 0 | Absent / broken |

---

## Behavioral Rules

- **Be honest and direct.** Do not soften scores. A 2 means real problems exist.
- **Cite evidence.** Every finding (positive or negative) must reference a specific `file:line`.
- **Name the lie.** For semantic issues, quote the method name then state what it actually does that the name doesn't say.
- **Cite call sites.** For semantic and error handling findings, grep for callers — misuse at call sites often reveals the deeper problem.
- **Distinguish accidental from intentional coupling.** Ask whether coupling exists because it must, or because it was convenient.
- **No invented APIs.** Only reference code that actually exists in the repo.
- **Distinguish facts from inferences.** Use "appears to" or "likely" when you cannot confirm by reading code.
- **Respect project conventions.** Check `CLAUDE.md` and README for stated patterns before flagging something as a violation.
- **Language mirroring.** Respond in whatever language the user writes in.

---

## After the Report

Offer to:
1. Dive deeper into any specific finding
2. Draft a decomposition plan for high-severity semantic issues
3. Generate missing tests for identified untested critical paths
4. Produce a module ownership migration plan (audit → dead code removal → directory restructuring) for Dimension 3 findings
