# File Structure for Packages

How to organize files so agents find and use them effectively.

## The Entry Point Pattern

Every package needs a single entry point that agents read first.

### Structure

```
.symposia/packages/[package-name]/
├── config.yml              ← Package config (name, version, file mappings)
├── registry.yml            ← Registry metadata (author, category, scenarios)
└── files/
    ├── QUICK_REFERENCE.md  ← Entry point doc
    ├── WORKFLOW.md          ← Detailed workflow doc
    ├── skills/
    │   └── my-skill/
    │       └── SKILL.md    ← Slash command or agent-invocable skill
    ├── hooks/
    │   └── pre-commit.sh   ← Hook script
    └── templates/
        └── config.json     ← Config template
```

Installed layout in the target project:

```
.claude/
├── docs/[package-name]/
│   ├── QUICK_REFERENCE.md    ← Entry point (agents read this first)
│   ├── DETAILED_WORKFLOW.md  ← Deep dive (referenced when needed)
│   ├── PATTERNS.md           ← Examples and templates
│   └── TROUBLESHOOTING.md    ← Problem-specific guidance
└── skills/[skill-name]/
    └── SKILL.md              ← User-invocable or agent-invocable skill
```

### Entry Point Responsibilities

The entry point file (QUICK_REFERENCE.md) must:

1. **Provide the core loop or checklist** - What agents do 80% of the time
2. **Include decision tables** - Quick yes/no guidance
3. **Reference detail files** - "For full process, see X.md"
4. **Stay scannable** - Under 100 lines ideal, 150 max

### Instruction File Injection

Symposia injects content into agent instruction files based on the tool:
- **Claude**: CLAUDE.md
- **Cursor**: .cursorrules
- **Copilot**: .github/copilot-instructions.md
- **Default**: AGENTS.md

The injection points to the entry point:

```markdown
## [Topic]

[When this applies]. See `.claude/docs/[package]/QUICK_REFERENCE.md` for [what].
```

**MUST** keep injections short. They compete for attention with other packages.

## When to Split Files

### Single File (< 150 lines)

Use one file when:
- Topic is focused and self-contained
- No distinct sub-topics
- Agents need everything in one place

### Multiple Files

Split when:
- Content exceeds 150 lines
- Distinct sub-topics exist (workflow vs patterns vs troubleshooting)
- Some content is reference material (read once, refer back)

### Decision Table

| Content Type | File Strategy |
|--------------|---------------|
| Core workflow | Entry point file |
| Decision tables | Entry point file |
| Detailed procedures | Separate file, referenced |
| Code examples/templates | Separate file or inline if short |
| Troubleshooting | Separate file |
| Background/theory | Omit or keep minimal |

## When to Create Separate Packages

Packages are composable. Split content into separate packages when topics are distinct.

### Separate Package

Create a new package when:
- Topic can be used independently
- Users may want one without the other
- Different projects need different combinations

### Same Package

Keep in same package when:
- Topics are always used together
- One topic depends entirely on the other
- Splitting would create incomplete instructions

### Decision Table

| Scenario | Package Strategy |
|----------|------------------|
| TDD workflow + test patterns | Same package (patterns support workflow) |
| TDD + code review guidelines | Separate packages (independent topics) |
| API design + API security | Separate packages (different concerns) |
| Git workflow + commit message templates | Same package (templates support workflow) |
| Refactoring + code smells | Same package (smells inform refactoring) |
| Refactoring + testing | Separate packages (independent disciplines) |

### The "And" Test

If describing your package requires "and", split it:

- "TDD workflow **and** test patterns" → OK, patterns support workflow
- "TDD **and** code review" → Split, independent topics
- "API design **and** database design **and** caching" → Split, three packages

## File Naming

### Convention

Use SCREAMING_SNAKE_CASE for all documentation files:

```
QUICK_REFERENCE.md
TDD_WORKFLOW.md
CODE_SMELLS.md
SAFE_REFACTORING.md
```

### Standard Names

| Name | Purpose |
|------|---------|
| QUICK_REFERENCE.md | Entry point, scannable summary |
| README.md | Alternative entry point |
| *_WORKFLOW.md | Step-by-step process |
| *_PATTERNS.md | Examples and templates |
| TROUBLESHOOTING.md | Problem → solution mappings |

## Cross-References

### When to Reference Another File

Reference detail files when:
- Agent needs more context for edge cases
- Full procedure is too long for entry point
- Content is "read once, refer back" material

### Reference Format

Always use relative paths from project root:

```markdown
For the full workflow, see `.claude/docs/tdd/TDD_WORKFLOW.md`.
```

**NEVER** use:
- Relative paths from current file (`./WORKFLOW.md`)
- Vague references ("see the workflow doc")
- Assumptions about file locations ("check the config")

### Inline vs Reference

| Situation | Strategy |
|-----------|----------|
| Agent needs info immediately | Inline it |
| Info is supplementary/edge case | Reference it |
| Content > 20 lines | Reference it |
| Agent must read to proceed | Inline it |

## Directory Placement

### Standard Location

```
.claude/docs/[package-name]/
```

### Naming the Directory

- Use lowercase with hyphens: `package-authoring`, `tdd`, `refactoring`
- Match the package name when possible
- Keep it short but descriptive

## Package Config Structure

The `config.yml` maps package files to project locations:

```yaml
name: sym-example
version: 1.0.0
files:
  # Docs — path is relative to package root, dest is where it lands in target project
  - path: files/QUICK_REFERENCE.md
    dest: .claude/docs/example/QUICK_REFERENCE.md
  - path: files/WORKFLOW.md
    dest: .claude/docs/example/WORKFLOW.md
  # Skills
  - path: files/skills/example-check/SKILL.md
    dest: .claude/skills/example-check/SKILL.md
  # Templates and hooks
  - path: files/templates/config.json
    dest: .project/config.json
  - path: files/hooks/lint-hook.sh
    dest: .project/hooks/lint-hook.sh
instructions:
  - targets:
      claude: CLAUDE.md
      cursor: .cursorrules
      copilot: .github/copilot-instructions.md
      default: AGENTS.md
    content: |
      ## Example Workflow

      When doing X, follow `.claude/docs/example/QUICK_REFERENCE.md`.
```

**Key convention**: `path` is relative to the package root (where `config.yml` lives). `dest` is where the file lands in the target project.

### Post-Install Messages

Add `postInstall` to `config.yml` when users must take manual action after installation (set environment variables, install external dependencies, run setup commands, etc.):

```yaml
name: sym-example
version: 1.0.0
postInstall: |
  Set your API key:
    export EXAMPLE_API_KEY=your-key-here
  Then run: npx example-setup
files:
  - path: files/QUICK_REFERENCE.md
    dest: .claude/docs/example/QUICK_REFERENCE.md
```

| Use `postInstall` When | Don't Use When |
|------------------------|----------------|
| Environment variables must be set | Package works out of the box |
| External tool must be installed | All dependencies are bundled |
| User must run a setup command | Sync handles everything |
| Credentials or config needed | Instructions alone are sufficient |

The message displays **only on first install** (`sym add`), not on subsequent `sym sync` runs. Keep it short and actionable — tell users exactly what to do, not why.

A `registry.yml` file sits alongside `config.yml` at the package root. See `.claude/docs/package-authoring/PUBLISHING.md` for the full field reference.

## Layering Information

### Three-Layer Model

```
Layer 1: Instruction file injection (CLAUDE.md, .cursorrules, etc.)
         → 2-3 sentences, triggers agent to read more

Layer 2: QUICK_REFERENCE.md
         → Core workflow, decision tables, checklists
         → Agent handles 80% of cases here

Layer 3: Detail files (WORKFLOW.md, PATTERNS.md)
         → Full procedures, extensive examples
         → Agent digs in for complex cases
```

### Information Flow

```
Instruction file: "Follow TDD. See QUICK_REFERENCE.md"
                    ↓
QUICK_REFERENCE: "Red→Green→Refactor. For test patterns, see TEST_PATTERNS.md"
                    ↓
TEST_PATTERNS: [Detailed examples, edge cases, anti-patterns]
```

Each layer **MUST** make sense standalone. Agents may not read all layers.
