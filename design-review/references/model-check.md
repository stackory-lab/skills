# Model Check

Load this reference when the proposal includes state, workflow, orchestration, or event-driven behavior.

## Questions

- What is the source of truth?
- Which state is persisted, and which state is derived?
- What are the legal states?
- What events cause transitions?
- Which transitions are forbidden?
- What invariants must always hold?
- Can two actors update the same entity concurrently?
- What happens if the same event is delivered twice?
- What happens if events arrive late or out of order?
- Can partial completion leave the model inconsistent?
- How is reconciliation performed if the real world and stored state diverge?

## Red Flags

- Multiple sources of truth without reconciliation rules
- State inferred indirectly from side effects
- Event names without clear ownership or ordering guarantees
- Derived state being mutated like canonical state
- Transitions that depend on timing assumptions instead of explicit conditions
- Missing invariants for balances, quotas, uniqueness, locks, or lifecycle stages
