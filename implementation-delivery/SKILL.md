---
name: implementation-delivery
description: Enforce delivery discipline while implementing — prove code runs end-to-end before claiming done, keep side-effects and cancellation honest across branches, respect module boundaries, and gate scope advances on approval. Use during any implementation task. NOT for planning or post-hoc code review.
---

# Implementation Delivery

This skill enforces delivery discipline **while implementing**. It sits between planning and post-hoc code review. Apply it on any task that adds or changes runtime behavior, regardless of language, stack, or project.

## The One Principle

**Written ≠ delivered.** Code that exists, compiles, and even has callers is not yet code that *runs correctly on every path, performs each side-effect the intended number of times, and honors cancellation*. Every rule below is a way of refusing to confuse "the code is there" with "the behavior happens."

The cheapest bugs to ship — and the hardest to see in review — all live in that gap:
- a function is written but never reached on the live path;
- a refactor splits a path and a side-effect now fires twice (or zero times);
- an error/fallback branch silently produces nothing;
- a timeout is constructed but never races the operation it was meant to bound.

Each rule targets one of these. When in doubt, return to the principle, not the checklist.

## Non-negotiables (MUST)

These are hard gates, not advice. Each MUST is bound to a moment of action and **names a concrete artifact that proves it was honored** — a line someone else can check. A "MUST" with no checkable artifact is just a louder adjective; do not add one. If you cannot produce the artifact, the step is not done.

The Rules below explain the *why*; these MUSTs are the *non-skippable how* at the points where, historically, discipline collapses under delivery momentum.

1. **MUST name evidence before marking anything done.** No gate, checkbox, or "done" claim flips without an evidence pointer naming the test (or live trace) **and the boundary it actually exercises**. Evidence borrowed from another milestone, "the types exist", or a test that does not traverse the named boundary → the item stays open.
   *Artifact:* the evidence pointer next to the checkbox / in the PR.

2. **MUST state what a path no longer does when you edit it.** Editing a middleware chain, route, or handler wiring requires writing the list of concerns the new path drops (auth, error boundary, trace, dedup, idempotency …).
   *Artifact:* the "now skipped" list in the PR. An empty list is a claim, not a default.

3. **MUST verify field fidelity across any model conversion.** For any shim between two shapes, list every field the consumer reads and mark each as survives / dropped. Compiling and green tests do not prove semantic fidelity.
   *Artifact:* the field list with survive/drop status.

4. **MUST ship critical-path code with its tests in the same change.** Auth, error boundary, idempotency, and anything security-relevant lands with tests covering at least its allow+deny (or success+failure) branches. Never "tests later".
   *Artifact:* the co-located test file in the same diff.

5. **MUST flag a blocked/hardcoded value, never leave it bare.** A value hardcoded because an upstream is missing gets, in the same change, a gap comment naming the blocker and the milestone that unblocks it, plus a note on any checkbox it affects.
   *Artifact:* the gap comment + the doc note.

## Rules

### 1. Close the loop before claiming done

A step is not done until you have traced the live call chain from entry point to observable effect (handler, write, render, response).

1. For every new export/function in **non-test** code, confirm a real caller exists on the production path — not just a test.
2. Pick one representative input and trace it end-to-end. Record the trace in your working notes / PR description — **not as a comment committed into source** (trace comments rot and mislead).
3. If a unit exists but nothing reaches it, the step is **not done**. Re-open any step that was checked off based on a component *existing* rather than *running*.

`grep`-for-callers catches "never wired." It does **not** catch "wired but wrong" — that's Rules 4–6.

### 2. Match evidence to the claim

A claim is only as strong as the evidence type behind it. Do not let cheap evidence stand in for an expensive claim.

| Claim | Minimum evidence |
|---|---|
| A pure function is correct | Unit test |
| A serialized/contract shape is stable | Round-trip / schema test |
| Two implementations agree | Differential or byte-equivalence test against the real other side |
| A component integrates | Test that actually exercises that component's boundary |
| It works end-to-end | Live trace (Rule 1) **or** a test that calls the real entry point |
| A doc/design is sound | Human review |

A unit test never satisfies an end-to-end claim. A "works on my one manual try" never satisfies a contract claim.

### 3. Respect boundaries; keep a single source of truth

Before adding an import or copy that crosses a module/layer boundary:

- Is it in the allowed dependency direction for *this* project? (Derive the direction from the repo's own conventions; don't assume.)
- If the dependency is blocked, can the shared logic move *down* to a common layer instead of being duplicated?
- If you duplicate for speed, say so explicitly in the PR. Silent duplication is the bug.

**Forbidden:** two implementations of the same algorithm in different places. The moment they exist, any cross-check between them (shadow/diff/parity) is comparing a thing against its own stale copy.

### 4. Enumerate branch paths

Any code that branches on a flag, capability, or condition MUST have every path combination listed with its expected behavior **before** the step is done. Each path is either traced (Rule 1) or covered by a test.

```
flag=on  + normal     → full new path        [traced / tested]
flag=on  + ineligible → fallback path        [traced / tested]
flag=on  + failure    → fallback path        [traced / tested]
flag=off             → legacy path           [exists]
cross-cutting: see Rules 5 and 6
```

An unlisted combination is an unhandled case, i.e. a latent bug.

### 5. Side-effect conservation

This is the rule that catches the bugs Rule 1 cannot see. When you **split, branch, or reorder** a path that performs a side-effect — network I/O, a dedup/idempotency mark, a state mutation, a cache write, an event emit — list every side-effect and assert it fires **exactly the intended number of times on every path**.

- A side-effect hoisted above a branch must not also remain inside the branches.
- A lookup done to *decide* something should be reused by the code that *acts* on it, not repeated.
- "It has a caller" says nothing about "it runs once." Count, don't grep.

**When to make counting an artifact (triggered, not always-on):** when a change
*alters where a side-effect fires* — splitting, hoisting, reordering, or branching
a path that does I/O / a dedup mark / a mutation / an emit — AND that side-effect's
call sites span more than one file, sketch a side-effect ledger in the PR: rows =
side-effects, columns = paths from Rule 4, cells = how many times it fires. Every
cell must equal its intended count. Otherwise, counting in your head is enough.
The ledger exists for exactly the case a per-block comment can't see: a side-effect
that fires in one file and *again* in another. It is scratch — it travels with the
PR, it is not kept or maintained.

```
                 path:normal  path:fallback  path:flag-off
dedup mark            1            1              1
fetch parent          1            0              0
handler call          1            1              1
```

### 6. No silent drops

Every non-happy path (fallback, error, ineligible, timeout) must still produce an **observable output**, and that must be asserted by a test — not merely "doesn't throw."

- `resolves`/`no error` is **not** evidence the message/result was delivered.
- For any "zero loss" / "nothing dropped" requirement, the regression test asserts the observable effect happened (handler called once, row written, response sent), per branch.

### 7. Cancellation must reach the real operation

If you construct a timeout or abort signal, prove it actually bounds the operation it targets.

- A signal passed to a wrapper but **not forwarded into the awaited call** (network request, query, child process) does nothing once that call is in flight.
- A synthesized timeout that nothing races against the `await` will never fire usefully.
- State the gap explicitly if you can only cancel *before* the call starts but not *during* it — "constructed a signal" is not "cancellation works."

### 8. Don't advance scope without approval

- After finishing the current unit's deliverables and verifying them via Rules 1–7, update whatever tracking artifact the project uses to reflect reality.
- **Do not start the next unit/milestone until the user explicitly approves.** Diagnostic questions or feedback on completed work mean *stay in fix/review mode*, not *move forward*.

## Pre-Commit Checklist

Items marked **(artifact)** must produce something concrete — a trace in notes, a test, a grep result. Only the last two are safe as mental checks; the rest fail silently under confidence.

1. [ ] **(artifact)** Every new production function has a real caller, and one input is traced entry→effect.
2. [ ] **(artifact)** Each branch-path combination is traced or tested.
3. [ ] **(artifact)** Each side-effect counted as exactly-N on every path (Rule 5).
4. [ ] **(artifact)** Each fallback/error path has a test asserting observable output (Rule 6).
5. [ ] **(artifact)** Any timeout/abort is forwarded into the real awaited call, or the gap is noted (Rule 7).
6. [ ] Evidence type matches each claim (Rule 2); cross-boundary imports respect direction (Rule 3).
7. [ ] Tracking artifact reflects reality; next unit not started without user approval.
