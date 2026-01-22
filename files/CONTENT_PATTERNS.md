# Content Patterns for Package Authoring

How to write instructions that agents follow reliably.

## Voice and Tone

### Use Imperative Voice

Agents execute commands. **MUST** write commands, not suggestions.

| Weak (Suggestion) | Strong (Command) |
|-------------------|------------------|
| "You should run the tests" | "Run tests" |
| "It's recommended to check coverage" | "Check coverage" |
| "Consider adding error handling" | "Add error handling" |
| "You might want to verify the output" | "Verify output" |

### Eliminate Hedging

Hedging creates ambiguity. Agents may skip hedged instructions.

| Hedged | Direct |
|--------|--------|
| "Generally, it's a good practice to..." | "Always..." |
| "In most cases, you should..." | "Do X. Exception: Y" |
| "It might be helpful to..." | "Do X" |
| "Consider whether you need to..." | Decision table |

### Use Priority Markers

Bold signals importance. Use consistently:

- **MUST** / **ALWAYS** - Non-negotiable requirements
- **NEVER** - Prohibited actions
- **IMPORTANT** - Critical context that affects decisions

```markdown
**MUST** run tests before committing.
**NEVER** commit secrets to the repository.
**IMPORTANT**: This only applies to production code.
```

## Personas

Personas frame *how* an agent approaches a task. Use selectively.

### When to Use Personas

| Task Type | Persona Useful? | Example |
|-----------|-----------------|---------|
| Security review | Yes | "Approach as an adversarial tester" |
| Code review | Yes | "Review as a maintainer who will own this code" |
| User-facing content | Yes | "Write for a developer new to the codebase" |
| Procedural steps | No | Checklists work better |
| Configuration | No | Decision tables work better |
| Reference docs | No | Direct instructions work better |

### Effective Persona Patterns

Shape mindset, not identity:

```markdown
## Security Review

Approach code as an adversarial tester. Assume all inputs are malicious
until validated. Look for: injection points, auth bypasses, data leaks.
```

```markdown
## API Design

Design for the frustrated developer at 2am debugging production.
Error messages **MUST** include: what failed, why, and how to fix it.
```

### Ineffective Persona Patterns

**Don't** use identity-based personas:

```markdown
# Wrong
You are an expert senior developer with 20 years of experience...

# Right
Review for: correctness, maintainability, edge cases.
```

**Don't** use personas for procedural tasks:

```markdown
# Wrong
As a careful QA engineer, you should consider running the tests...

# Right
1. Run `npm test`
2. Fix failures
3. Run `npm run lint`
```

### Combining Personas with Instructions

Personas work best as brief framing before procedural content:

```markdown
## Code Review

Review as someone who will maintain this code for years.

### Checklist
- [ ] Clear naming
- [ ] No magic numbers
- [ ] Error handling present
- [ ] Tests cover edge cases
```

## Decision Structures

### Decision Tables

When agents must choose between options, provide a table.

```markdown
| Scenario | Action |
|----------|--------|
| New feature with clear requirements | Use TDD |
| Bug fix | Write regression test first |
| Exploratory prototype | Skip TDD |
| Simple config change | Skip TDD |
```

### Yes/No Guidance

For binary decisions, be explicit:

```markdown
### When to Create a New File

**Yes:**
- Functionality is reusable across modules
- File would exceed 300 lines
- Distinct responsibility from existing files

**No:**
- One-off helper used in single location
- Would create file under 50 lines
- Tightly coupled to existing code
```

### Conditional Logic

For complex conditions, use explicit if/then:

```markdown
**If** tests exist for the code → Run them first
**If** no tests exist → Write tests before changing code
**If** change is trivial (typo, comment) → Tests optional
```

## Structural Patterns

### The Loop Pattern

For iterative processes, number the steps and name the loop:

```markdown
## The Core Loop

1. Write failing test
2. Write minimal code to pass
3. Refactor while green
4. Commit
5. Repeat
```

### Wrong/Right Contrast

Show mistakes alongside corrections:

```markdown
### Commit Sizing

**Wrong**: Change 15 files, hope tests pass.

**Right**: Change 1 thing, verify, commit. Repeat.
```

### Checklists

For verification steps, use task lists:

```markdown
## Before Submitting PR

- [ ] All tests pass
- [ ] No linting errors
- [ ] Documentation updated
- [ ] Changelog entry added
```

### Templates

When agents must generate content, provide copy-paste templates:

````markdown
## Commit Message Template

```
<type>: <description>

<body>
```

Types: feat, fix, refactor, test, docs
````

## Content Density

### Keep Paragraphs Short

**Wrong:**
> When you're working on implementing a new feature, it's important to consider the overall architecture of the system and how your changes will integrate with existing functionality. You should think about edge cases and error handling, and make sure to write tests that cover the main paths through your code.

**Right:**
> ## Implementing Features
>
> 1. Review existing architecture
> 2. Identify integration points
> 3. Write tests for main paths
> 4. Handle edge cases
> 5. Add error handling

### One Idea Per Section

Each heading covers exactly one concept. Covering multiple ideas? Split into multiple sections.

### Use Vertical Space

Whitespace aids scanning. Separate logical groups with blank lines.

## Context Independence

### Assume No Prior Context

Agents don't carry context between sessions. **MUST** make every instruction self-contained.

**Wrong:** "As mentioned earlier, run the validation."

**Right:** "Run `npm run validate` to check types and linting."

### Include Concrete Examples

Abstract guidance fails. Concrete examples succeed.

**Wrong:** "Use meaningful variable names."

**Right:**
```
# Wrong
const d = new Date();
const x = d.getTime();

# Right
const currentDate = new Date();
const timestamp = currentDate.getTime();
```

### Specify File Paths

When referencing files, use exact paths:

**Wrong:** "See the config file for details."

**Right:** "See `.claude/docs/tdd/TDD_WORKFLOW.md` for the full process."
