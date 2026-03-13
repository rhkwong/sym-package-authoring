# Skills, Hooks, and Non-Doc Files

Packages can ship more than markdown docs. This covers skills, hooks, templates, and config files.

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

## Hooks

Shell scripts triggered by git events or other system hooks.

### Shipping Hooks with `inject`

Use the `inject` block to insert hook content into shared files like `.git/hooks/post-commit`. Symposia wraps injected content in marker sections so multiple packages can coexist in the same file.

```yaml
inject:
  - source: files/post-commit.sh
    dest: .git/hooks/post-commit
    comment: "#"
    executable: true
```

| Field | Required | Description |
|-------|----------|-------------|
| `source` | Yes | Path to script within package |
| `dest` | Yes | Target file in project |
| `comment` | Yes | Comment syntax for markers (`"#"`, `"//"`, `"--"`) |
| `executable` | No | Set `chmod +x` on target file |

**Marker format** (auto-generated, do not write manually):

```bash
# --- symposia:my-package ---
# ... injected content ...
# --- /symposia:my-package ---
```

**Key behaviors:**
- **Idempotent** — re-syncing replaces the marked section, no duplication
- **Multi-package** — multiple packages can inject into the same file
- **Cleanup** — `sym remove` strips the marked section; deletes the file if empty
- **Shebang** — stripped from source content; target file gets `#!/bin/sh` on creation when `comment` is `"#"` and `executable` is `true`

### `inject` vs `files` Decision Table

| Scenario | Use |
|----------|-----|
| Content must coexist with other packages in one file | `inject` |
| Git hooks (post-commit, pre-push, etc.) | `inject` |
| Standalone script owned entirely by your package | `files` |
| Config files, templates, seed data | `files` |
| Documentation files | `files` |

### Legacy Pattern (standalone hook files)

For hooks that don't need to share a file, ship as standalone scripts via `files`:

```yaml
files:
  - path: files/hooks/my-check.sh
    dest: .project/hooks/my-check.sh
```

### First-Install Activation

Use `postInstall.activate` to auto-launch a setup skill on first install. The user is prompted for confirmation in TTY mode; inside an existing Claude session, an instruction is printed for the agent.

```yaml
postInstall:
  message: "Hook installed. Running setup to configure .gitignore."
  activate:
    file: files/skills/my-setup/SKILL.md
```

| `activate` Field | Description |
|------------------|-------------|
| `file` | Path to a skill file (relative to package root) — skill name is parsed from frontmatter |
| `prompt` | Inline text prompt passed directly to Claude |

At least one of `file` or `prompt` is required. `postInstall` fires **only on first install** (`sym add`), not on subsequent `sym sync` runs.

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

| Content Type | Mechanism | Example |
|---|---|---|
| Agent instructions | `files` | `.claude/docs/{pkg}/QUICK_REFERENCE.md` |
| Slash command skills | `files` | `.claude/skills/{name}/SKILL.md` |
| Git hooks (shared file) | `inject` | `.git/hooks/post-commit` |
| Standalone hook scripts | `files` | `.project/hooks/my-check.sh` |
| Config templates | `files` | `.para/config.json` |
| Gitkeep/seed files | `files` | `.para/reviews/.gitkeep` |

## config.yml Example with Mixed File Types

```yaml
name: sym-example-full
version: 1.0.0
postInstall:
  message: "Hook installed. Run /example-setup to configure."
  activate:
    file: files/skills/example-setup/SKILL.md
files:
  # Docs
  - path: files/QUICK_REFERENCE.md
    dest: .claude/docs/example/QUICK_REFERENCE.md
  - path: files/WORKFLOW.md
    dest: .claude/docs/example/WORKFLOW.md
  # Skills
  - path: files/skills/example-check/SKILL.md
    dest: .claude/skills/example-check/SKILL.md
  - path: files/skills/example-setup/SKILL.md
    dest: .claude/skills/example-setup/SKILL.md
  # Templates
  - path: files/templates/config.json
    dest: .project/config.json
  - path: files/templates/.gitkeep
    dest: .project/output/.gitkeep
inject:
  # Git hook — shared file, marker-isolated
  - source: files/post-commit.sh
    dest: .git/hooks/post-commit
    comment: "#"
    executable: true
instructions:
  - targets:
      claude: CLAUDE.md
    content: |
      ## Example Workflow

      Follow `.claude/docs/example/QUICK_REFERENCE.md`. Use `/example-check` to validate.
```
