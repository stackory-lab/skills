# Programmatic API

## Generate or Check Documents

```ts
import {
  defineConfig,
  syncAiContext,
} from '@stackory/sync-agent-context';

const config = defineConfig({
  fragmentsDir: 'ai/fragments',
  documents: [
    {
      name: 'Agents',
      output: 'AGENTS.md',
      title: '# Global Agent Instructions',
      fragments: ['rules', 'workflow'],
    },
  ],
});

const result = await syncAiContext({
  config,
  mode: 'write',
  rootDir: process.cwd(),
});

for (const output of result.outputs) {
  console.log(output.name, output.output, output.changed);
}
```

Use `mode: 'check'` to compare without writing.

## Build One Document In Memory

```ts
import { buildDocument } from '@stackory/sync-agent-context';

const markdown = await buildDocument({
  title: '# Agent Instructions',
  introLines: ['AUTO-GENERATED. DO NOT EDIT.'],
  fragments: ['rules'],
  fragmentsDir: 'ai/fragments',
  fragmentTitles: {
    rules: 'Rules & Guardrails',
  },
});
```

## Load Specific Sections

```ts
import { loadSpecificInstructions } from '@stackory/sync-agent-context';

const sections = await loadSpecificInstructions('ai/specific.md');
const claude = sections['claude-specific-instructions'];
```

## Slug Behavior

```ts
import { slugify } from '@stackory/sync-agent-context';

slugify('Claude Specific Instructions'); // claude-specific-instructions
```

## Result Shape

```ts
type ISyncAiContextResult = {
  mode: 'write' | 'check';
  outputs: Array<{
    name: string;
    output: string;
    changed: boolean;
  }>;
};
```

## Gotchas

- `syncAiContext` validates config before reading files.
- `loadSpecificInstructions` returns `{}` when the file does not exist.
- `buildDocument` expects `fragmentsDir` to already point to the actual fragment directory.

