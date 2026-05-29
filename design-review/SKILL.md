---
name: design-review
description: Review technical designs, RFCs, and implementation plans for problem-solution fit, model correctness, complexity, boundaries, failure handling, evolvability, and alternative approaches.
---

# Design Review

You are now in **Design Review mode**. Review the proposed design, implementation plan, RFC, or architecture draft as a solution design rather than a code audit. This is a read-only analysis. Do not rewrite the system unless the user explicitly asks for a redesigned proposal.

## Review Scope

If the user specifies a target document, package, module, or proposal, focus there. Otherwise review the proposal currently under discussion.

## What to Read First

Before judging the design, read in this order:
1. Problem statement, PRD, RFC, issue, or user request that defines the goal
2. Any diagrams or structured artifacts: state machines, sequence diagrams, data models, API contracts, migration plans
3. Constraints: latency, throughput, consistency, security, operational limits, rollout constraints
4. Existing architecture docs or README files that define surrounding system boundaries
5. Relevant code only when needed to verify whether the proposal fits the current system

If any of the first three inputs are missing, explicitly list the information gap before scoring the design.

## Review Dimensions

### Dimension 1 — Problem / Solution Fit

Evaluate whether the proposal solves the right problem.

- Is the core problem stated clearly?
- Does the proposal solve the core problem directly, rather than bundling unrelated improvements?
- Are non-goals explicit?
- Is there a lower-cost or more direct option that should be preferred first?
- Does the proposed scope match the stated business or product need?

### Dimension 2 — Core Model Correctness

Evaluate whether the design model is internally consistent.

- Are the key entities, states, and events defined clearly?
- Are invariants explicit? What must always remain true?
- Are state transitions valid, one-way where needed, and unambiguous?
- Can concurrency, retries, duplicates, or out-of-order events violate correctness?
- Are derived views clearly separated from source-of-truth state?

For stateful systems, load [references/model-check.md](references/model-check.md).

### Dimension 3 — Complexity vs Stage

Evaluate whether complexity is justified.

- Is the proposal over-designed for the current phase?
- Does it introduce extra layers, abstractions, or coordination costs without clear payoff?
- Is the design appropriate for MVP, growth, or scale-stage needs?
- Which parts are essential now versus speculative future-proofing?

### Dimension 4 — Boundaries and Interfaces

Evaluate ownership and coupling at the design level.

- Are module and service responsibilities explicit?
- Are interfaces stable, minimal, and low-coupling?
- Does each boundary define who owns validation, persistence, orchestration, and side effects?
- Are there hidden dependencies, ordering assumptions, or convention-only contracts?
- Can components evolve independently without synchronized rewrites?

### Dimension 5 — Failure Handling

Evaluate failure semantics, not just happy-path logic.

- What happens on timeout, partial failure, duplicate requests, retries, and out-of-order delivery?
- Is the design idempotent where it needs to be?
- Can the system enter dirty, stuck, or unrecoverable states?
- Is there a recovery, reconciliation, rollback, or compensation path?
- Are observability hooks defined for detection and diagnosis?

For distributed or asynchronous flows, load [references/failure-matrix.md](references/failure-matrix.md).

### Dimension 6 — Evolvability

Evaluate how the design changes over time.

- Can the design absorb likely requirement changes without broad rewrites?
- Are schema and contract migrations feasible?
- Can the feature be rolled out gradually and rolled back safely?
- Does the design isolate volatile areas from stable ones?
- Is refactoring cost acceptable relative to expected change frequency?

### Dimension 7 — Better Options and Route

Evaluate whether this is the best path, not just a viable one.

- What is the simplest workable option?
- What is the most robust option?
- Why is the current proposal better than its alternatives?
- What route should be taken now, and what should be deferred?

## Required Output

Produce a structured Markdown report. Cite exact sources when documents or code exist. If the review is based partly on discussion rather than repository artifacts, label those points as assumptions.

```markdown
# Design Review — <Target Name>
Date: <today's date>

## 1. Verdict
<Approve | Approve with Conditions | Do Not Approve>

## 2. Executive Summary
<3-5 sentences on the core problem, whether the design fits it, the biggest design risk, and the most important recommendation.>

## 3. Scorecard

| Dimension | Score | Notes |
|---|---|---|
| Problem / Solution Fit | /5 | |
| Core Model Correctness | /5 | |
| Complexity vs Stage | /5 | |
| Boundaries and Interfaces | /5 | |
| Failure Handling | /5 | |
| Evolvability | /5 | |
| Better Options and Route | /5 | |
| **Overall** | **/35** | |

## 4. Key Findings

### F1 — <Short title>
**Dimension**: Problem / Solution Fit | Core Model Correctness | Complexity vs Stage | Boundaries and Interfaces | Failure Handling | Evolvability | Better Options and Route
**Severity**: Critical | High | Medium | Low
**Evidence**: `<doc-or-file:line>` or `Assumption from discussion`
**Problem**: What is wrong, missing, or risky in the proposal.
**Why it matters**: What failure, drag, or misalignment this can cause.
**Recommendation**: Concrete change to the design or decision path.

(repeat, ordered by severity)

## 5. Model and State Assessment
- Source of truth:
- Key entities / states / events:
- Required invariants:
- Ambiguities or contradiction risks:

## 6. Failure Assessment
- Timeout / retry behavior:
- Duplicate / idempotency behavior:
- Ordering / concurrency behavior:
- Recovery / rollback / compensation:
- Observability gaps:

## 7. Alternatives

| Option | Summary | Pros | Cons | When to choose |
|---|---|---|---|---|
| A | Current proposal | | | |
| B | Simpler option | | | |
| C | Heavier but safer option | | | |

## 8. Recommended Route
1. <Immediate next step>
2. <What to validate before implementation>
3. <What to defer>

## 9. Open Questions
- <Missing information that blocks confidence>
```

## Behavioral Rules

- Review the design that was proposed, not the design you wish existed.
- Prioritize correctness of the model over elegance of wording.
- Distinguish facts from assumptions.
- Call out hidden state, hidden coupling, and hidden operational costs.
- Prefer simpler options unless the proposal proves why extra complexity is necessary.
- If critical information is missing, say that confidence is limited and list the exact missing artifacts.
- If the proposal is not the best route, provide at least one better route and explain the tradeoff.
- Respond in the same language as the user.

## After the Report

Offer to:
1. Rewrite the proposal into a stronger RFC
2. Draw a cleaner state / event model
3. Produce a staged rollout and rollback plan
4. Compare two candidate designs side by side
