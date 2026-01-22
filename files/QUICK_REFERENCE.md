# Package Authoring Quick Reference

## The Golden Rule

**IMPORTANT**: Write for agents, not humans. Agents scan, match patterns, and execute. Optimize for that.

## Package Scope

**MUST**: One package = one subject. Packages are composable building blocks.

| Do | Don't |
|----|-------|
| `sym-tdd` - just TDD workflow | `sym-quality` - TDD + linting + formatting + reviews |
| `sym-api-design` - REST/GraphQL patterns | `sym-backend` - API + database + auth + caching |
| `sym-git-workflow` - branching and commits | `sym-devops` - git + CI + deployment + monitoring |

**Why this matters:**
- Agents only read files relevant to their current task
- Prevents context rot from loading unrelated instructions
- Users install only what they need
- Easier to maintain and update

**Test:** Can you describe the package purpose in one phrase without "and"?

## Content Checklist

Before publishing a package (see `CONTENT_PATTERNS.md` for detailed guidance):

- [ ] Instructions use imperative voice ("Run tests", not "You should run tests")
- [ ] Every decision point has a table or clear Yes/No guidance
- [ ] Examples show both correct and incorrect patterns
- [ ] No hedging language ("consider", "might want to", "perhaps")
- [ ] Checklists exist for multi-step processes
- [ ] Bold marks critical keywords (**MUST**, **NEVER**, **IMPORTANT**)
- [ ] Content is scannable (bullets, tables, short paragraphs)

## File Structure Checklist

See `FILE_STRUCTURE.md` for the three-layer model and directory conventions.

- [ ] Entry point file exists (QUICK_REFERENCE.md or README.md)
- [ ] Instruction file injection references the entry point
- [ ] Detailed docs are separate files, referenced when needed
- [ ] File names are SCREAMING_SNAKE_CASE.md
- [ ] Cross-references use relative paths

## Effective Patterns

| Pattern | Use When | Example |
|---------|----------|---------|
| Decision table | Agent must choose between options | "When to TDD?" Yes/No table |
| Wrong/Right contrast | Preventing common mistakes | "Wrong: X. Right: Y." |
| Numbered steps | Sequential process | "1. Write test 2. Run test 3. Fix" |
| Checklist | Verification before/after action | Pre-commit checklist |
| Template | Agent needs to generate content | Commit message template |
| Code block | Showing syntax or structure | Config file examples |
| Persona | Shaping analytical mindset | "Review as an adversarial tester" |

## Instruction File Injection Pattern

Symposia injects into agent instruction files: CLAUDE.md, .cursorrules, .github/copilot-instructions.md, or AGENTS.md.

```markdown
## [Package Purpose]

[One sentence stating when this applies]. See `.claude/docs/[package]/QUICK_REFERENCE.md` for [what they'll find there].
```

**MUST** keep injections to 2-3 sentences. Detail belongs in referenced files.

## Anti-Patterns

See `EXAMPLES.md` for side-by-side comparisons of ineffective vs effective content.

| Don't | Do Instead |
|-------|------------|
| "You might want to consider running tests" | "Run tests" |
| Dense paragraphs explaining concepts | Bullets, tables, examples |
| "Generally speaking, it's a good idea to..." | "Always X" or decision table |
| Vague guidance ("write clean code") | Specific criteria ("functions < 20 lines") |
| Assuming context from conversation | Self-contained instructions |

## Quick Debugging

If agents aren't following your package instructions:

1. **Check injection** - Is the instruction file (CLAUDE.md, .cursorrules, etc.) referencing the right file?
2. **Check imperative voice** - Rewrite suggestions as commands
3. **Check specificity** - Add concrete examples or criteria
4. **Check scannability** - Break walls of text into structure
5. **Check decision points** - Add tables for "when to do X"
