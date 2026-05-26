# Fragments and Specific Sections

## Recommended Layout

```text
ai/
  fragments/
    rules.md
    workflow.md
    project.md
  specific.md
ai-context.config.mjs
```

## Fragment Files

Each fragment id maps to `${fragmentsDir}/${id}.md`.

For `fragments: ['rules', 'workflow']`, the generator reads:

```text
ai/fragments/rules.md
ai/fragments/workflow.md
```

If a fragment starts with a top-level title, that first `# ...` heading is stripped before insertion.

```md
# Rules

- Keep responses concise.
- Prefer local project conventions.
```

With this config:

```js
fragmentTitles: {
  rules: 'Rules & Guardrails',
}
```

The generated output contains:

```md
## Rules & Guardrails

- Keep responses concise.
- Prefer local project conventions.
```

## Specific Sections

Use `specificPath` when one shared file contains agent-specific additions.

```md
# Specific Instructions

## Agents Specific Instructions

Use AGENTS.md-compatible wording.

## Claude Specific Instructions

Prefer Claude Code terminology.
```

Section headings are slugified:

- `## Agents Specific Instructions` becomes `agents-specific-instructions`
- `## Claude Specific Instructions` becomes `claude-specific-instructions`

Use the slug as a document profile's `specificKey`.

```js
{
  name: 'Claude',
  output: 'CLAUDE.md',
  title: '# Claude Instructions',
  fragments: ['rules'],
  specificKey: 'claude-specific-instructions',
}
```

## Gotchas

- Only `##` headings create selectable specific sections.
- A top-level `#` heading in `specificPath` is ignored.
- Missing `specificPath` or missing `specificKey` section is allowed; no extra section is appended.
- Generated output is deterministic for the same config and fragment contents.

