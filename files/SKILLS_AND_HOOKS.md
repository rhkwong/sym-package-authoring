# Skills, Hooks, and Non-Doc Files

Packages can ship more than markdown docs. This covers skills, hooks, file injection, post-install activation, templates, and config files.

## File Delivery Mechanisms

Choose the right mechanism for each piece of content:

| Mechanism | Use when | Example |
|-----------|----------|---------|
| `files` | Static content copied verbatim | docs, templates, seed configs, skill files |
| `instructions` | Inject into agent config files (CLAUDE.md, .cursorrules) | 2-3 sentence package blurb |
| `inject` | Inject into arbitrary files with comment markers | git hooks, shell configs, CI files |

### Decision Table

| Content | Mechanism | Why |
|---------|-----------|-----|
| Markdown docs | `files` | Agents read them as-is |
| Skill files | `files` | Installed to `.claude/skills/` |
| Agent config blurb | `instructions` | Marker-wrapped injection into CLAUDE.md etc. |
| Git hook content | `inject` | Merges with other hooks in same file |
| CI config additions | `inject` | Preserves existing config, adds marked section |
| Standalone config | `files` | Copied once, no merge needed |

## Skills

Skills are structured instruction files that agents can invoke as commands or trigger programmatically.

### Location

```
.claude/skills/{skill-name}/SKILL.md
```

### Frontmatter

Every skill file has YAML frontmatter:

```yaml
---
name: my-skill
description: Short description of what this skill does
allowed-tools: [Bash, Read, Write, Edit, Grep, Glob]
user-invocable: true
agent-invocable: false
model: sonnet
---
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Skill identifier (used as `/name` slash command if user-invocable) |
| `description` | Yes | What the skill does (shown in help/discovery) |
| `allowed-tools` | No | Tools the skill can use |
| `user-invocable` | No | If `true`, becomes a `/skill-name` slash command |
| `agent-invocable` | No | If `true`, agents can trigger it programmatically |
| `model` | No | Use `sonnet` for routine tasks; omit for default model |

### Minimal Skill Template

```markdown
---
name: example-check
description: Run standard checks on the current file
allowed-tools: [Bash, Read]
user-invocable: true
---

# Example Check

Read the current file and run lint + type checks.

1. Identify the file the user is working on
2. Run `npm run lint -- {file}`
3. Run `npm run typecheck`
4. Report results as a checklist
```

### Shipping Skills via config.yml

```yaml
files:
  - path: files/skills/my-skill/SKILL.md
    dest: .claude/skills/my-skill/SKILL.md
```

## File Injection

Inject content into arbitrary files using comment-delimited markers. Content is wrapped in markers so it can be idempotently updated and cleanly removed on `sym remove`.

### When to Use Injection

| Scenario | Use `inject` | Use `files` |
|----------|-------------|-------------|
| Git hook that coexists with other hooks | Yes | No |
| Shell profile additions | Yes | No |
| Standalone config file | No | Yes |
| File that must merge with existing content | Yes | No |
| File that should be replaced entirely | No | Yes |

### config.yml Syntax

```yaml
inject:
  - source: files/hooks/post-commit.sh    # source file in package
    dest: .git/hooks/post-commit           # target file in project
    comment: "#"                           # marker comment style
    executable: true                       # chmod +x after write
```

| Field | Required | Description |
|-------|----------|-------------|
| `source` | Yes | Path to source file within the package |
| `dest` | Yes | Destination file path in target project |
| `comment` | Yes | Comment prefix for markers: `"#"`, `"//"`, `"--"` |
| `executable` | No | If `true`, `chmod +x` the dest file after write |

### Marker Format

Injected content is wrapped in comment-delimited markers:

```bash
# --- symposia:my-package ---
# injected content here
# --- /symposia:my-package ---
```

The comment prefix changes based on `comment`:

| `comment` | Start marker | End marker |
|-----------|-------------|------------|
| `"#"` | `# --- symposia:pkg ---` | `# --- /symposia:pkg ---` |
| `"//"` | `// --- symposia:pkg ---` | `// --- /symposia:pkg ---` |
| `"--"` | `-- --- symposia:pkg ---` | `-- --- /symposia:pkg ---` |

### Injection Behavior

| Scenario | What happens |
|----------|-------------|
| Dest file does not exist | Create file with marked section. If `comment: "#"` and `executable: true`, prepend `#!/bin/sh` shebang. |
| Dest file exists, no markers | Append marked section after existing content |
| Dest file exists, has markers | Replace existing marked section (idempotent) |
| Source file has shebang (`#!/...`) | Strip shebang before injection — dest file owns the shebang |
| `executable: true` | `chmod +x` applied after write |

### Removal Behavior

On `sym remove`:

| After removal | What happens |
|---------------|-------------|
| File has remaining content | Marked section stripped, file preserved |
| File is empty or only shebang | File deleted entirely |

## Post-Install Activation

Trigger automated setup after a package is installed for the first time.

### When to Use

| Scenario | Use `activate` | Use `postInstall` message |
|----------|---------------|--------------------------|
| Package has a `/setup` skill to run | Yes, with `file` | Also add message as fallback |
| Complex setup requiring agent interaction | Yes | Also add message as fallback |
| User must set env vars manually | No | Yes |
| Package works out of the box | No | No |

### config.yml Syntax

```yaml
postInstall:
  message: "Run /my-setup to complete setup."
  activate:
    file: files/skills/my-setup/SKILL.md
```

Or with an inline prompt:

```yaml
postInstall:
  message: "Configure your API endpoint."
  activate:
    prompt: "Ask the user for their API endpoint and write it to .env"
```

| Field | Description |
|-------|-------------|
| `activate.file` | Path to a skill file in the package. Extracts skill `name` from YAML frontmatter and shows `/skill-name` instruction. |
| `activate.prompt` | Inline prompt string passed directly to Claude. |

**MUST** provide either `file` or `prompt`, not both.

### Activation Behavior

| Context | What happens |
|---------|-------------|
| **Terminal (TTY)** | Prompts user: "Run /skill-name to complete setup. Launch Claude?" If accepted, spawns Claude subprocess. |
| **Inside Claude Code (non-TTY)** | Prints instruction: "Run /skill-name to complete setup." The calling agent reads this and acts. |

Activation fires **only on first install** (`sym add`), not on subsequent `sym sync` runs.

### Best Practice

Pair `activate` with a setup skill. The skill handles interactive setup (creating configs, prompting for values, installing hooks). This pattern is cleaner than inline prompts.

## Hooks

Shell scripts that integrate with git or Claude Code events.

### Shipping Hooks via File Injection

Use `inject` to install hook content that coexists with other hooks in the same file:

```yaml
inject:
  - source: files/hooks/post-commit.sh
    dest: .git/hooks/post-commit
    comment: "#"
    executable: true
```

This is the **preferred pattern** for git hooks. The injected content is wrapped in markers and cleanly removed on `sym remove`.

### When Manual Setup Is Still Needed

Use a setup skill or `postInstall` message when:
- Hooks require environment-specific configuration (API keys, paths)
- Hooks register with Claude Code's `.claude/settings.json` (PostToolUse, PreToolUse events)
- Hook behavior varies per project

## Templates and Config Files

Non-markdown files like JSON configs, seed files, and directory scaffolding.

### Shipping Templates

```yaml
files:
  - path: files/templates/config.json
    dest: .para/config.json
  - path: files/templates/.gitkeep
    dest: .para/reviews/.gitkeep
```

Ship sensible defaults. If customization is needed, provide a `/setup` skill that modifies the template after install.

## Decision Table: What Goes Where

| Content Type | Mechanism | Destination | Example |
|---|---|---|---|
| Agent instructions | `files` | `.claude/docs/{pkg}/` | `QUICK_REFERENCE.md` |
| Slash command skills | `files` | `.claude/skills/{name}/SKILL.md` | `para-triage` skill |
| Agent config blurb | `instructions` | CLAUDE.md, .cursorrules, etc. | 2-3 sentence injection |
| Git hooks | `inject` | `.git/hooks/{hook}` | post-commit script |
| Hook scripts (non-git) | `files` or `inject` | Project-specific path | `.para/hooks/check.sh` |
| Config templates | `files` | Project-specific path | `.para/config.json` |
| Seed files | `files` | Project-specific path | `.para/reviews/.gitkeep` |

## Complete config.yml Example

```yaml
name: sym-example-full
version: 1.0.0
files:
  # Docs
  - path: files/QUICK_REFERENCE.md
    dest: .claude/docs/example/QUICK_REFERENCE.md
  - path: files/WORKFLOW.md
    dest: .claude/docs/example/WORKFLOW.md
  # Skills
  - path: files/skills/example-setup/SKILL.md
    dest: .claude/skills/example-setup/SKILL.md
  # Templates
  - path: files/templates/config.json
    dest: .project/config.json
  - path: files/templates/.gitkeep
    dest: .project/output/.gitkeep
instructions:
  - targets:
      claude: CLAUDE.md
      cursor: .cursorrules
      copilot: .github/copilot-instructions.md
      default: AGENTS.md
    content: |
      ## Example Workflow

      Follow `.claude/docs/example/QUICK_REFERENCE.md`. Use `/example-check` to validate.
inject:
  - source: files/hooks/post-commit.sh
    dest: .git/hooks/post-commit
    comment: "#"
    executable: true
postInstall:
  message: "Run /example-setup to complete setup."
  activate:
    file: files/skills/example-setup/SKILL.md
```
