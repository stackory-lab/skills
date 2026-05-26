# Config

Create `ai-context.config.mjs` in the target repository.

```js
import { defineConfig } from '@stackory/sync-agent-context';

export default defineConfig({
  fragmentsDir: 'ai/fragments',
  specificPath: 'ai/specific.md',
  fragmentTitles: {
    rules: 'Rules & Guardrails',
    workflow: 'Workflow',
    project: 'Project Context',
  },
  documents: [
    {
      name: 'Agents',
      output: 'AGENTS.md',
      title: '# Global Agent Instructions',
      introLines: ['AUTO-GENERATED. DO NOT EDIT.'],
      fragments: ['rules', 'workflow', 'project'],
      specificKey: 'agents-specific-instructions',
    },
    {
      name: 'Claude',
      output: 'CLAUDE.md',
      title: '# Claude Instructions',
      introLines: ['AUTO-GENERATED. DO NOT EDIT.'],
      fragments: ['rules', 'workflow'],
      specificKey: 'claude-specific-instructions',
    },
    {
      name: 'Gemini',
      output: 'GEMINI.md',
      title: '# Gemini Instructions',
      introLines: ['AUTO-GENERATED. DO NOT EDIT.'],
      fragments: ['rules', 'project'],
      specificKey: 'gemini-specific-instructions',
    },
  ],
});
```

## Fields

- `fragmentsDir`: directory containing fragment Markdown files. Fragment id `rules` maps to `rules.md`.
- `fragmentTitles`: optional display heading per fragment id. If absent, the raw id is used.
- `specificPath`: optional Markdown file containing `##` sections that can be selected per document.
- `separator`: optional string inserted between generated sections. Defaults to a Markdown horizontal rule.
- `documents`: generated output profiles.

## Document Profile

- `name`: human-readable label for CLI/API result output.
- `output`: generated file path, relative to `rootDir` unless absolute.
- `title`: top-level title written at the top of the generated document.
- `introLines`: optional lines under the title, commonly used for generated-file warnings.
- `fragments`: ordered fragment ids to include.
- `extraSections`: optional inline sections appended after fragments.
- `specificKey`: optional slug key looked up from `specificPath`.

## Gotchas

- Config must default-export the config object or export it as `config`.
- Every document needs at least one fragment.
- Relative paths are resolved from `rootDir`, not from the config file directory.

