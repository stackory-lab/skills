---
name: ai-docs
description: Generate or improve AI-readable project documentation (CLAUDE.md, README.md) optimized for LLM consumption
---

# AI-Readable README Skill

This skill helps generate documentation optimized for AI consumption (like CLAUDE.md) rather than human marketing.

## Core Principles

1. **Prescriptive over Descriptive** - Use imperative rules ("Must use kebab-case") not explanations ("We prefer kebab-case because...")
2. **Structured over Narrative** - Bullet points and sections over paragraphs
3. **Exact over Ambiguous** - Specific commands and paths over general descriptions
4. **Dense over Verbose** - Every line must add actionable information
5. **Constraints First** - What NOT to do is as important as what to do

## Required Document Structure

When generating AI-readable documentation, MUST include these sections in order:

### 1. Rules & Guardrails
**Purpose**: Hard constraints that prevent mistakes
**Format**:
```markdown
## Rules & Guardrails

- Never commit secrets or sample credentials
- Prefer incremental, reviewable diffs
- Do not invent APIs, file structures, or commands
- Always mention which tests should be run to validate changes
```

**Guidelines**:
- Use negative constraints: "Never", "Do NOT", "Avoid"
- Include security policies first
- Specify what requires confirmation before action
- Reference tooling alignment (Node versions, package managers)

### 2. Core Project Context
**Purpose**: Essential structure and commands
**Format**:
```markdown
## Core Project Context

- Project is a [monorepo/single-package] using [build tool]
- Repository layout:
  - `path/`: description of contents
  - `path/`: description of contents
- Primary commands:
  - Install: `exact command`
  - Dev: `exact command`
  - Build: `exact command`
  - Test: `exact command`
```

**Guidelines**:
- Use exact paths from repo root
- Provide complete command syntax with placeholders
- Explain workspace/package relationships
- List dependency management workflows

### 3. Architecture Notes
**Purpose**: Boundaries, constraints, and data flow
**Format**:
```markdown
## Architecture Notes

- **Component Type**: Purpose and responsibilities
  - Specific constraints (e.g., "Do not import in client bundles")
  - Runtime requirements (e.g., "Edge-compatible only")
- **Data Layer**: Database, ORM, connection patterns
- **Shared Packages**: Import rules and runtime constraints
```

**Guidelines**:
- Explicitly state what CANNOT be mixed (e.g., server-only vs client)
- Define package boundaries with import rules
- Specify runtime constraints (Node-only, edge-compatible, etc.)
- Document data access patterns

### 4. Coding Style
**Purpose**: Code conventions for consistency
**Format**:
```markdown
## Coding Style

- Language: TypeScript (strict). Prefer async/await.
- Formatting: [Tool name], [key settings]
- File/Folder naming: MUST use kebab-case. DO NOT use camelCase or PascalCase.
- Interfaces: MUST prefix with I (e.g., `interface IUser`)
- Imports: prefer workspace aliases over relative paths
- Testing: co-locate specs, avoid network fixtures
- Commit style: imperative conventional form
- Return types: Prefer inference; only specify for public APIs
- Require curly braces for all control statements
```

**Guidelines**:
- Use "MUST", "SHOULD", "PREFER" to indicate priority
- Provide exact patterns (not "follow existing style")
- Include both positive and negative examples when helpful
- Cover: naming, typing, imports, testing, commits

### 5. Output & Collaboration Expectations
**Purpose**: How AI should format responses
**Format**:
```markdown
## Output & Collaboration Expectations

- Default tone: concise, friendly teammate
- Reference files: `path/to/file.ts:42` (path + line number)
- Use fenced code blocks with language identifiers
- Prefer `apply_patch`-friendly diffs over full file rewrites
- Ask for confirmation before major refactors or schema changes
```

**Guidelines**:
- Specify file reference format
- State preferred edit methods
- Define when to ask for confirmation
- Include tone and communication style

### 6. Examples & Patterns (Optional)
**Purpose**: Common workflows with exact steps
**Format**:
```markdown
## Examples & Patterns

### Adding a Workspace Dependency
1. `pnpm add <dep> --filter <package> --save-exact`
2. If internal: use `workspace:*` version
3. Run `pnpm install`
4. Update `ncu.json` when appropriate
```

**Guidelines**:
- Use numbered steps for sequential workflows
- Include exact commands, not paraphrases
- Show common operations (add dependency, create package, etc.)
- Keep examples minimal and actionable

## Writing Guidelines

### DO:
- Start with the most restrictive rules (security, don't-do-this)
- Use imperative mood: "Use", "Avoid", "Must", "Never"
- Reference exact file paths from repo root
- Group related constraints together
- Include exact command syntax with placeholders
- Specify when commands should be run (e.g., "before committing")
- Use consistent formatting (same heading levels for same content types)

### DON'T:
- Include marketing language or project vision
- Use narrative explanations when rules suffice
- Add historical context or commented examples
- Repeat information across sections
- Use ambiguous terms: "usually", "typically", "generally"
- Include visual aids (badges, screenshots, emojis)
- Link to external docs in the main body (save context window)
- Explain "why" unless it affects decision-making

## File Naming Convention

- **CLAUDE.md** - Primary AI instruction file loaded in every session
- **README.md** - Human-facing project overview (marketing, setup, features)
- **.claude/rules/*.md** - Path-specific or domain-specific rules

## Context Window Optimization

Since CLAUDE.md loads in every conversation:
- Prioritize constraints over explanations
- Use bullet points over paragraphs
- Link to detailed docs rather than embedding
- Remove redundant or rarely-needed information
- Consider moving verbose architecture diagrams to README.md

## Quality Checklist

Before finalizing AI-readable documentation:

- [ ] Every constraint uses imperative mood ("Must", "Never", "Do NOT")
- [ ] All commands include exact syntax with placeholders
- [ ] File paths are from repository root
- [ ] No marketing language or aspirational statements
- [ ] No ambiguous terms ("usually", "prefer" without context)
- [ ] Sections are ordered by priority (constraints first)
- [ ] No repeated information across sections
- [ ] Examples show complete workflows, not fragments
- [ ] Architecture boundaries explicitly state what cannot mix
- [ ] Output format specifies file reference style

## Usage

Invoke this skill when:
- Creating or updating CLAUDE.md for a project
- Documenting architecture constraints for AI
- Standardizing code conventions across a team
- Setting up AI-friendly project guidelines

The skill automatically references your existing CLAUDE.md structure and adapts to your project's specific needs.
