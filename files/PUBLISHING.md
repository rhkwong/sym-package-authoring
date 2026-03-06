# Publishing Packages to the Registry

How to prepare, publish, and maintain packages in the Symposia registry.

## registry.yml Reference

The `registry.yml` file sits alongside `config.yml` at the package root. It provides discovery metadata for the registry.

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `author` | string | Your registry username |
| `license` | string | SPDX license identifier (e.g., `MIT`, `Apache-2.0`) |
| `stability` | string | `experimental`, `stable`, or `mature` |
| `when_to_use` | string | Comma-separated activities an agent would be doing |
| `category` | string | One of: `workflow`, `quality`, `planning`, `tooling`, `conventions` |

### Important Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `scenarios` | list | Concrete task descriptions a user might type (embedded for semantic search) |
| `problems` | list | Pain points this package solves |
| `tags` | list | Short keywords for filtering |
| `agents` | list | Supported agents: `claude`, `cursor`, `windsurf`, `copilot` |
| `languages` | list | Programming languages or `any` |
| `not_when` | string | When this package is NOT appropriate |

### Relationship Fields

| Field | Type | Description |
|-------|------|-------------|
| `depends_on` | list | Packages that must be installed alongside |
| `complements` | list | Packages that work well together but aren't required |
| `conflicts_with` | list | Packages that should not be used simultaneously |
| `supersedes` | list | Older packages this one replaces |

### Complete Example

```yaml
author: rhkwong
license: MIT
stability: stable
category: workflow
when_to_use: "writing tests first, doing TDD, red-green-refactor cycle"
not_when: "writing tests after implementation, exploratory prototyping"
scenarios:
  - "implement a feature using TDD"
  - "write failing tests before code"
  - "refactor with test coverage"
problems:
  - "tests written after code miss edge cases"
  - "no clear process for test-first development"
tags: [tdd, testing, workflow, red-green-refactor]
agents: [claude, cursor, windsurf, copilot]
languages: [any]
complements: [sym-code-review]
```

## Publish Flow

### Steps

1. Ensure `registry.yml` exists alongside `config.yml` in the package root
2. Verify `version` in `config.yml` is valid semver (`x.y.z`)
3. Run the publish command:
   ```bash
   sym publish <package-name> github:owner/repo
   ```
   Or run `sym publish <package-name>` for interactive destination selection.
4. CLI pushes to GitHub and tags `v{version}`
5. If the owner is a verified publisher and `registry.yml` exists, CLI auto-notifies the registry
6. Registry indexes the package for search and discovery

### Private Packages

1. Authenticate first: `sym login`
2. Publish with org and private flags:
   ```bash
   sym publish <package-name> github:owner/repo --org <org-name> --private
   ```
3. The org must exist in the registry and you must be a member

## Writing Good Discovery Metadata

### `when_to_use`

Write as comma-separated activities an agent would be doing. Think: "This package helps when the agent is..."

| Good | Bad |
|------|-----|
| "writing tests first, doing TDD, red-green-refactor cycle" | "testing" |
| "creating symposia packages, writing agent instructions" | "package development" |
| "reviewing pull requests, checking code quality" | "code review stuff" |

### `scenarios`

Concrete task descriptions a user might type. These get embedded for semantic search, so write them as natural queries.

| Good | Bad |
|------|-----|
| "implement a feature using TDD" | "TDD" |
| "write a registry.yml for publishing" | "registry" |
| "refactor with test coverage" | "refactoring" |

### `problems`

Pain points this package solves. Write from the user's perspective.

### `tags`

Short keywords for filtering. Keep them lowercase, no spaces.

### `category` Decision Table

| Category | Use When |
|----------|----------|
| `workflow` | Package defines a step-by-step process (TDD, code review, git flow) |
| `quality` | Package enforces standards (linting rules, naming conventions, test patterns) |
| `planning` | Package helps with design/architecture (API design, system planning) |
| `tooling` | Package helps use tools or create artifacts (package authoring, CI setup) |
| `conventions` | Package defines team/project conventions (commit messages, file naming) |

## Version Management

- Bump `version` in `config.yml` before publishing
- Use `--force` to overwrite an existing tag (republish same version)
- Use `--dry-run` to preview what would be pushed without actually publishing

## Publishing Checklist

- [ ] `registry.yml` exists with all required fields
- [ ] `version` in `config.yml` is valid semver
- [ ] All files referenced in `config.yml` `files` array exist
- [ ] `when_to_use` and `scenarios` are written for search discovery
- [ ] Cross-references in content use dest paths (`.claude/docs/...`), not source paths (`files/...`)
