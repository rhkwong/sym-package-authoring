# Package Authoring Examples

Side-by-side comparisons of ineffective and effective package content.

## Example 1: Instruction File Injection

### Ineffective

```markdown
## Code Quality

This project uses various code quality tools and practices. When working on code,
you should generally try to follow good coding practices and make sure your code
is clean and well-organized. Consider running linters and formatters when
appropriate. See the docs folder for more information about our coding standards.
```

**Problems:**
- Vague ("various tools", "good practices", "when appropriate")
- No specific file reference
- Hedging language ("should generally try", "consider")
- No actionable command

### Effective

```markdown
## Code Quality

**MUST** run `npm run lint` before committing. Follow `.claude/docs/quality/QUICK_REFERENCE.md` for formatting rules and error handling patterns.
```

**Why it works:**
- Specific command to run
- Exact file path to reference
- Clear requirement (MUST)
- Concise (2 sentences)

---

## Example 2: Decision Guidance

### Ineffective

```markdown
## When to Write Tests

Testing is an important part of software development. You should consider writing
tests for your code, especially when working on critical functionality. The amount
and type of testing depends on various factors including the complexity of the
code, the risk involved, and time constraints. Use your judgment to determine
the appropriate level of testing for each situation.
```

**Problems:**
- No concrete guidance
- "Use your judgment" provides no decision framework
- No actionable criteria

### Effective

```markdown
## When to Write Tests

| Scenario | Write Tests? |
|----------|--------------|
| New feature with business logic | Yes |
| Bug fix | Yes - regression test first |
| Config file change | No |
| Refactoring existing code | Verify existing tests pass |
| Prototype/spike | No |

**If unsure:** Write the test. Deletion is easier than debugging.
```

**Why it works:**
- Concrete scenarios agents can match
- Clear yes/no answers
- Fallback guidance for edge cases

---

## Example 3: Process Documentation

### Ineffective

```markdown
## Deployment Process

When you're ready to deploy your changes, you'll want to make sure everything
is properly tested and reviewed. The deployment process involves several steps
that should be followed carefully. First, you should verify that all tests pass.
Then, you'll need to get your changes reviewed. After that, you can merge your
changes and deploy them. Make sure to monitor the deployment for any issues.
```

**Problems:**
- Buried in prose
- No clear sequence
- Missing specific commands
- Vague monitoring guidance

### Effective

```markdown
## Deployment Process

1. Run `npm test` - all tests must pass
2. Run `npm run build` - verify build succeeds
3. Create PR and get approval
4. Merge to main
5. Run `./scripts/deploy.sh production`
6. Verify deployment at https://app.example.com/health

**If health check fails:** Run `./scripts/rollback.sh` immediately.
```

**Why it works:**
- Numbered steps
- Specific commands
- Clear success criteria
- Failure handling included

---

## Example 4: Code Standards

### Ineffective

```markdown
## Code Style

We follow standard coding conventions in this project. Try to write clean,
readable code that other developers can understand. Use meaningful names for
variables and functions. Keep functions small and focused. Add comments where
the code might be confusing.
```

**Problems:**
- "Standard conventions" is undefined
- "Clean, readable" is subjective
- "Meaningful names" lacks examples
- No measurable criteria

### Effective

```markdown
## Code Style

### Naming
```typescript
// Wrong
const d = getData();
const x = d.filter(i => i.active);

// Right
const users = fetchUsers();
const activeUsers = users.filter(user => user.active);
```

### Function Size
- **Max 30 lines** per function
- **Max 3 parameters** - use options object for more
- **Single responsibility** - if you use "and" describing it, split it

### Comments
- **Don't:** Comment what code does
- **Do:** Comment why non-obvious decisions were made
```

**Why it works:**
- Concrete examples
- Measurable limits (30 lines, 3 parameters)
- Do/don't format

---

## Example 5: Error Handling

### Ineffective

```markdown
## Error Handling

Proper error handling is crucial for robust applications. You should catch
errors appropriately and handle them in a way that provides good user experience.
Make sure to log errors for debugging purposes. Consider what information to
show to users versus what to log internally.
```

**Problems:**
- No specific patterns
- "Appropriately" is undefined
- No code examples

### Effective

```markdown
## Error Handling

### Pattern

```typescript
try {
  const result = await riskyOperation();
  return result;
} catch (error) {
  logger.error('Operation failed', { error, context: relevantData });
  throw new UserFacingError('Unable to complete request. Please try again.');
}
```

### Rules

| Error Type | Log? | Show to User? |
|------------|------|---------------|
| Validation error | No | Yes - specific message |
| Network error | Yes | "Connection failed" |
| Internal error | Yes - full stack | "Something went wrong" |
| Auth error | Yes - no credentials | "Please log in" |

**NEVER** expose stack traces or internal details to users.
```

**Why it works:**
- Copy-paste code pattern
- Decision table for error types
- Explicit prohibition

---

## Example 6: File References

### Ineffective

```markdown
For more details on testing, see the testing documentation. The API docs
contain information about available endpoints. Check out the architecture
doc for system design decisions.
```

**Problems:**
- No file paths
- Agent cannot locate files
- Vague descriptions

### Effective

```markdown
- Testing workflow: `.claude/docs/tdd/QUICK_REFERENCE.md`
- API endpoints: `docs/api/ENDPOINTS.md`
- Architecture decisions: `docs/architecture/ADR_INDEX.md`
```

**Why it works:**
- Exact file paths
- Agent can read immediately
- Clear purpose for each reference

---

## Example 7: Shipping Git Hooks

### Ineffective

```yaml
# config.yml — ships hook as standalone file, requires manual setup
files:
  - path: files/hooks/post-commit.sh
    dest: .project/hooks/post-commit.sh
  - path: files/HOOK_SETUP.md
    dest: .claude/docs/my-pkg/HOOK_SETUP.md
postInstall: "Run /my-setup to register the hook in .claude/settings.json"
```

**Problems:**
- User must manually run a setup skill to register the hook
- Cannot coexist with other packages that also need post-commit hooks
- Hook lives in a non-standard location, not in `.git/hooks/`

### Effective

```yaml
# config.yml — injects into shared git hook, auto-runs setup
inject:
  - source: files/post-commit.sh
    dest: .git/hooks/post-commit
    comment: "#"
    executable: true
postInstall:
  message: "Post-commit hook installed. Running setup."
  activate:
    file: files/skills/my-setup/SKILL.md
```

**Why it works:**
- `inject` wraps content in markers — multiple packages share `.git/hooks/post-commit`
- `executable: true` handles `chmod +x` automatically
- `postInstall.activate` launches the setup skill on first install (with user confirmation)
- `sym remove` cleans up only this package's section

---

## Anti-Pattern Checklist

Review your package content for these issues:

- [ ] **Hedge words**: "should", "consider", "might", "generally"
- [ ] **Vague references**: "see the docs", "check the config"
- [ ] **Missing examples**: Rules without code samples
- [ ] **Dense paragraphs**: More than 3 sentences without structure
- [ ] **Subjective criteria**: "clean code", "good practices", "appropriate"
- [ ] **Missing decisions**: "use your judgment" without framework
- [ ] **Passive voice**: "Tests should be run" vs "Run tests"
