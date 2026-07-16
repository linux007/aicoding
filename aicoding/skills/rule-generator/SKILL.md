---
name: rule-generator
description: Analyze code examples and existing patterns to create or improve Cursor-style rule files (.claude/rules/). Use when the user wants to generate rules, create rule files, improve rules, analyze code patterns, or convert existing project conventions into reusable rule files. Also triggers when user says "generate rules", "create rule file", "improve rules", "analyze patterns", or asks to extract patterns from code. If the user wants to create rules for a specific language, framework, or layer (e.g., controller, service, API), this skill handles it.
---

# Rule Generator

You analyze code examples and existing patterns to create or improve rule files in the `.claude/rules/` format.

## What you do

1. **Analyze** the provided code or existing codebase patterns
2. **Extract** common conventions, patterns, and best practices
3. **Generate** actionable, specific rule files
4. **Structure** rules following the Cursor rule file format

## Process

### Step 1: Identify what to analyze

If the user provides code examples directly, work with those. Otherwise:

1. Ask the user what area of the codebase they want rules for (specific layer, module, or technology)
2. If unclear, explore the codebase to understand the relevant patterns
3. For the edutube project specifically, check existing `.claude/rules/` files as a starting point

### Step 2: Pattern recognition

Analyze code across these dimensions:

- **Naming conventions**: class, function, variable, file naming patterns
- **Structural patterns**: file organization, function/method structure, layer separation
- **Architectural patterns**: dependency injection, error handling, testing patterns, data validation
- **Code style preferences**: comment usage, type annotations, function length, parameter patterns
- **Technology-specific patterns**: framework conventions, library usage, configuration patterns

### Step 3: Generate rule files

**Prefer many small, focused rule files** over one large general file. Each rule file should follow the Single Responsibility Principle.

Every rule file MUST follow this format:

```markdown
---
description: One-line description of what the rule enforces
paths:
  - "components/request_params/**/*.go"
  - "components/response_params/**/*.go"
alwaysApply: boolean
---

# Rule Category Name

- **Main Points in Bold**
  - Sub-points with details
  - Examples and explanations
```

### Rule organization priority

1. **General/Language Rules First** — broad programming principles and language-specific rules
2. **Project-Specific Rules** — rules for specific libraries, frameworks, or project patterns
3. **File-Specific Rules** — rules tied to specific file types or paths

### Writing guidelines

- **Keep Rules Concise** — shortest possible rules that still convey requirements clearly
- **Be Specific and Actionable** — implementable, not theoretical
- **Use Strong Language for Critical Rules** — NEVER, ALWAYS, MUST
- **Use Soft Language for Preferences** — prefer, consider, should
- **Include Both DO and DON'T** — only when essential for clarity
- **Show Good Patterns Only** — avoid showing bad examples unless critical
- **Reference other rule files** with `@filename` to keep rules DRY

### Rule quality checklist

Before finalizing, verify:

- [ ] Strong language (NEVER/ALWAYS) is used for critical rules
- [ ] Rules follow the Single Responsibility Principle
- [ ] No contradictions with other rules in the file
- [ ] Proper frontmatter with description and globs
- [ ] Clear markdown structure with headings
- [ ] Minimal but sufficient information
- [ ] Rules reflect the actual project, not generic standards

## Output

For each rule file you create:

1. Write the complete file content with proper frontmatter
2. Explain briefly what patterns you extracted and why
3. Suggest appropriate glob patterns for when the rule should apply
4. Recommend where to place it (typically `.claude/rules/<name>.md`)

## When to add, modify, or remove rules

### Add new rules when:
- A new technology/pattern is used in 3+ files
- Common bugs could be prevented by a rule
- Code reviews repeatedly mention the same feedback
- New security or performance patterns emerge

### Modify existing rules when:
- Better examples exist in the codebase
- Related rules have been updated
- Implementation details have changed
- After major refactoring efforts

### Remove rules when:
- Technology or pattern is no longer used
- Rule contradicts new project direction
- Rule is too generic to be useful
- Rule is covered by more specific rules
