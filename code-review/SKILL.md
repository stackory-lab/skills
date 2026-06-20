---
name: code-review
description: Review code for actionable architecture, semantic, module ownership, and error-handling defects. Use when Codex needs to review a package, module, directory, file set, or change and produce a concise findings-only repair list that a code implementer can execute and validate directly.
---

# Code Review

Perform a read-only code review. Investigate thoroughly, but report only confirmed, actionable problems. Optimize the report for the code implementer who will apply the fixes.

## Review Scope

Focus on the target specified by the user. Otherwise, review the current working directory.

## Investigation

Read the following before judging the code:

1. Read the `README.md`, `CLAUDE.md`, and architecture documentation in scope.
2. Read entry points, public types, `package.json`, and co-located tests.
3. Trace relevant production call paths and side effects.
4. Search workspace consumers before judging exports or public APIs.
5. Derive validation commands from the affected Nx projects and repository instructions.

Use the four dimensions below as an internal checklist. Do not create scores, dimension summaries, or positive-pattern sections from this checklist.

## Review Dimensions

### Architecture

- Verify separation between transport, business logic, persistence, and lifecycle concerns.
- Check type safety, testability, scalability, observability, security, and public API boundaries.
- Flag only problems with a concrete behavioral, maintenance, security, or operability impact.
- Prefer domain subpath exports over a flattened root barrel when real consumers follow domain boundaries.
- Do not recommend runtime namespace objects solely to organize TypeScript package exports.

### Semantic Quality

- Verify names describe all observable behavior and side effects.
- Flag methods with multiple reasons to change or wrappers that add no behavior, validation, or semantic narrowing.
- Check caller predictability, async transparency, mutation style, ordering requirements, and idempotency.
- Trace callers before concluding that a semantic mismatch is harmful.

### Module Ownership

- Investigate suspicious barrels, dead exports, misplaced shared code, circular dependencies, and cross-domain imports.
- Start from real consumers rather than assuming the current `index.ts` defines the correct public API.
- Treat import counts as evidence, not conclusions. A single-owner or multi-owner symbol is not inherently misplaced.
- Report only ownership problems that require a concrete move, privatization, deletion, or boundary change.
- Include relevant consumers directly in the finding instead of producing a separate ownership map.

### Error Handling

- Check swallowed errors, floating promises, error classification, boundary handling, actionable context, and cleanup.
- Verify every failure and fallback path produces the intended observable result.
- Confirm cancellation and timeout signals reach the real awaited operation.
- Trace callers and boundary behavior before reporting propagation problems.

## Finding Standard

Report a finding only when all of the following are available:

- A confirmed defect or material risk, not a preference or speculative improvement.
- A precise `file:line` location and supporting call-site evidence when relevant.
- A single required change that is specific enough to implement without design discovery.
- Observable acceptance criteria, including required regression coverage.

Merge locations that share one root cause into one finding. Split findings that require independent fixes or acceptance checks.

Do not report:

- Positive patterns, scores, or general health assessments.
- Style-only nits without material impact.
- Raw ownership inventories or analysis artifacts.
- Optional refactors presented as defects.
- Open questions that further repository investigation can answer.

If a missing fact makes implementation unsafe, continue investigating. If the fact cannot be derived from the repository, list it under `Blockers` and do not present the uncertain recommendation as a finding.

## Severity

- **Critical**: Exploitable security issue, data loss, or system-wide failure requiring immediate action.
- **High**: Incorrect production behavior, broken critical path, serious reliability issue, or major boundary violation.
- **Medium**: Real defect or maintainability problem with bounded impact that should be scheduled.

Omit low-value findings and style-only nits.

## Report Format

Order findings by severity, then by implementation dependency. Use this exact structure:

```markdown
# Code Review — <Target>

## Findings

### F1 — <Problem title>
**Severity**: Critical | High | Medium
**Location**: `file:line`

**Problem**
Describe the current behavior, trigger, impact, and relevant call-site evidence.

**Required Change**
State the required implementation and affected boundary. Do not give a menu of vague options.

**Acceptance Criteria**
- State observable behavior that must hold after the fix.
- Name the regression test or scenario that must be added or updated.
- State the important behavior that must not regress.

## Blockers

- Include only facts that cannot be derived from the repository and prevent a safe fix.
- Omit this section when there are no blockers.

## Validation

- `<targeted commands for the affected findings>`
- `pnpm nx run <project>:check:lint`
- `pnpm nx run <project>:check:type`
- `pnpm nx run <project>:check:dep`
- `pnpm nx run <project>:check:test`
- `pnpm nx run <project>:build`
```

Use one `Validation` section for the whole report. Do not repeat commands under individual findings.

Choose validation commands based on scope:

- One project: list the project's `check:lint`, `check:type`, `check:dep`, `check:test`, and `build` targets.
- Multiple known projects: use `pnpm nx run-many -t check:lint,check:type,check:dep,check:test,build -p <project-a> <project-b>`.
- Uncertain or broad affected scope: use `pnpm run check:affected`.
- Package metadata changes: also use `pnpm run monosync`.
- Add the narrowest targeted regression-test command when the repository exposes one.

If no actionable findings exist, output only:

```markdown
# Code Review — <Target>

No actionable findings.
```

## Behavioral Rules

- Cite `file:line` for every claim and cite relevant call sites for semantic and error-handling findings.
- State facts directly and label unavoidable inference explicitly.
- Describe what the code does, not just which guideline it violates.
- Make `Required Change` imperative and implementation-ready.
- Keep acceptance criteria observable and verifiable.
- Do not invent APIs, consumers, commands, or repository conventions.
- Mirror the user's language.
