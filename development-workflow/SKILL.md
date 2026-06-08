---
name: development-workflow
description: Use when planning or executing non-trivial development work that needs requirements clarification, architecture boundaries, TDD planning, milestones, or implementation gates. Scale the process to the task size and avoid heavy RFCs for small changes.
---

# Development Workflow

Use this skill for non-trivial development tasks before implementation or when the user asks for planning, architecture, TDD, milestones, or a development process. The goal is to keep work grounded in clear behavior and verifiable delivery without turning every task into a large RFC.

## Core Rule

Scale the process to the task.

- **Small**: one-file or obvious changes. Confirm intent, identify validation, then implement.
- **Medium**: new behavior, cross-file changes, or meaningful tests. Produce a compact plan before implementation.
- **Large**: new modules, systems, public APIs, data model changes, or risky refactors. Produce a fuller planning artifact and get alignment before implementation.

Do not force every section below into the final answer. Use only what the task needs.

## Workflow

1. **Classify**
   - Decide whether the task is small, medium, or large.
   - If the user is explicitly brainstorming or planning, do not implement yet.

2. **Clarify Intent**
   - State the problem being solved.
   - Identify the user or caller affected by the change.
   - Define success criteria in observable terms.
   - Ask only for blockers that cannot be reasonably inferred.

3. **Define Scope**
   - List what is in scope.
   - List explicit non-goals.
   - Record assumptions that affect behavior or architecture.
   - Surface open questions only when they materially change the route.

4. **Behavior First**
   - Describe expected behavior before designing files or classes.
   - Prefer scenario form: given current state, when action occurs, then result.
   - Include important edge cases and failure paths.

5. **Minimal Architecture**
   - Define source of truth.
   - Define module or package boundaries.
   - Define public interfaces and ownership.
   - Define dependency rules and forbidden coupling.
   - Call out risks that influence the implementation route.

6. **TDD Matrix**
   - Convert important behavior into tests.
   - Prioritize core rules, state transitions, edge cases, failure paths, and cross-module scenarios.
   - Avoid low-value tests that only mirror implementation details.

7. **Milestones**
   - Split work into small, verifiable stages.
   - Each milestone should have a goal, deliverables, tests, and done criteria.
   - Prefer milestones that leave the codebase in a valid state.

8. **Code Guide**
   - For medium/large planning artifacts intended for maintainers or new contributors, include a final code guide section.
   - List the key files and packages in recommended reading order.
   - Explain what each file owns, not just its path.
   - Call out current risk points or behavior hotspots that the plan intends to change.
   - Keep it practical: entrypoints, public types, main flows, tests, and integration points.

9. **Implementation Loop**
   - For each milestone: write or update tests, implement the minimal code, refactor, then validate.
   - If implementation reveals a wrong assumption, update the plan or call it out before continuing.

## Compact Planning Template

Use this for medium and large tasks, trimming sections as needed.

```markdown
## Intent
- Problem:
- User / caller:
- Success criteria:

## Scope
- In:
- Out:
- Assumptions:
- Open questions:

## Behavior Contract
- Scenario:
- Edge cases:
- Failure behavior:

## Architecture
- Source of truth:
- Boundaries:
- Public interfaces:
- Dependency rules:
- Risks:

## TDD Matrix
| Behavior | Test Type | Priority |
|---|---|---|

## Milestones
### M1
- Goal:
- Deliverables:
- Tests:
- Done when:

## Code Guide
- Reading order:
- Entrypoints:
- Public contracts:
- Main flow:
- Tests:
- Integration points:
- Known risk hotspots:
```

## Practical Guardrails

- Requirements come before architecture.
- Behavior comes before file structure.
- Architecture should define only necessary boundaries.
- TDD should lock behavior, not implementation trivia.
- Milestones must be verifiable, not just task lists.
- Code guides are for orientation, not exhaustive file inventories.
- Keep the smallest useful artifact for the task size.
