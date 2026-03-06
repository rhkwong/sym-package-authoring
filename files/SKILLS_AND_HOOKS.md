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

Shell scripts triggered by Claude Code events (pre-tool-call, post-tool-call, etc.).

### How to Ship Hooks

1. Include the hook script in your `config.yml` files array
2. Document setup instructions in a `HOOK_SETUP.md` or via a `/setup` skill

```yaml
files:
  - path: files/hooks/pre-commit-check.sh
    dest: .para/hooks/pre-commit-check.sh
  - path: files/HOOK_SETUP.md
    dest: .claude/docs/my-pkg/HOOK_SETUP.md
```

### Important Limitation

Symposia has no `post_install` hook. Users must configure `.claude/settings.json` manually or use a setup skill to register hooks. Document this clearly.

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

| Content Type | Destination | Example |
|---|---|---|
| Agent instructions | `.claude/docs/{pkg}/` | `QUICK_REFERENCE.md` |
| Slash command skills | `.claude/skills/{name}/SKILL.md` | `para-triage` skill |
| Hook scripts | Project-specific path | `.para/hooks/pre-commit.sh` |
| Config templates | Project-specific path | `.para/config.json` |
| Gitkeep/seed files | Project-specific path | `.para/reviews/.gitkeep` |

## config.yml Example with Mixed File Types

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
  - path: files/skills/example-check/SKILL.md
    dest: .claude/skills/example-check/SKILL.md
  # Hooks
  - path: files/hooks/lint-hook.sh
    dest: .project/hooks/lint-hook.sh
  # Templates
  - path: files/templates/config.json
    dest: .project/config.json
  - path: files/templates/.gitkeep
    dest: .project/output/.gitkeep
instructions:
  - targets:
      claude: CLAUDE.md
    content: |
      ## Example Workflow

      Follow `.claude/docs/example/QUICK_REFERENCE.md`. Use `/example-check` to validate.
```
