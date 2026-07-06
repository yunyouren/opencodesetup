# Subagent Recommendations (OpenCode)

Subagents are specialized opencode instances with their own context and a scoped permission set. They're ideal for focused, isolated reviews or analysis. In OpenCode they live in `.opencode/agents/<name>.md` with `mode: subagent`.

> Design custom subagents based on the codebase's specific review/analysis needs. These are common patterns.

## Agent file (OpenCode)

`.opencode/agents/my-reviewer.md`:

```yaml
---
description: Reviews PRs for style violations.
mode: subagent
model: anthropic/claude-sonnet-4-6
permission:
  read: allow
  glob: allow
  grep: allow
  edit: deny
  bash: ask
---
You are a strict PR reviewer. Focus on...
```

- The file **body** becomes the agent's prompt. Do not also put `prompt:` in frontmatter.
- `mode`: `subagent` (dispatched), `primary` (top-level, selectable), `all`.
- Allowed frontmatter fields: `name, model, variant, description, mode, hidden, color, steps, options, permission, disable, temperature, top_p`. Unknown fields route into `options`.
- `model` always carries a provider prefix: `anthropic/claude-sonnet-4-6`, `openai/gpt-4o`, etc.
- Per-agent `permission:` overrides the top-level `permission:`.

## Permission shapes for agents

```yaml
permission:
  read: allow            # string action: allow | ask | deny
  bash:                  # object { pattern: action }; last match wins (broad * first, specific last)
    "*": ask
    "git *": allow
    "rm *": deny
  edit: deny
  task: deny
```

Known keys: `read, edit, glob, grep, list, bash, task, external_directory, todowrite, question, webfetch, websearch, lsp, doom_loop, skill`. Flat-action-only keys (`todowrite, question, webfetch, websearch, doom_loop`) take a single action, not a pattern object.

## Code review agents

### code-reviewer
**Best for:** automated quality checks on large codebases.
Recommend when: >500 files, frequent changes, team wants consistent review.
Model: a balanced sonnet-class. Tools: read/grep/glob (+ bash ask for git).
Permission: read/glob/grep=allow, edit=deny, bash=ask.

### security-reviewer
**Best for:** security-focused review.
Recommend when: `auth/`/`login`/`session`, payment (`stripe`/`payment`/`billing`), user/PII data, API-key patterns.
Model: sonnet-class. Permission: read-only (read/glob/grep=allow, edit/bash=deny) for safety.

### test-writer
**Best for:** generating test coverage.
Recommend when: low coverage, test suite exists, jest/pytest/vitest configured.
Model: sonnet-class. Permission: read/glob/grep=allow, edit=allow (writes tests), bash=ask.

## Specialized agents

### api-documenter
**Best for:** API doc generation.
Recommend when: REST endpoints (Express/FastAPI), GraphQL schema, OpenAPI exists, undocumented APIs.
Permission: read=allow, edit=allow (writes docs), bash=deny.

### performance-analyzer
**Best for:** finding bottlenecks.
Recommend when: ORM/raw SQL, hot paths, complex algorithms.
Permission: read/grep/glob=allow, edit=deny, bash=ask (to run profilers/benchmarks).

### ui-reviewer
**Best for:** accessibility & UX review.
Recommend when: React/Vue/Angular, component library, user-facing UI.
Permission: read-only.

## Utility agents

### dependency-updater
**Best for:** safe dependency updates.
Recommend when: `npm outdated`/`npm audit` results, major version gaps.
Permission: read/edit=allow, bash=ask (run install/test).

### migration-helper
**Best for:** framework/version migrations.
Recommend when: very old framework version, deprecation warnings, planned refactor.
Model: a strong reasoning model. Permission: read/edit/glob/grep=allow, bash=ask.

## Quick reference: detection -> subagent

| If you see | Recommend |
|---|---|
| Large codebase (>500 files) | code-reviewer |
| Auth/payment code | security-reviewer |
| Few tests vs source | test-writer |
| API routes | api-documenter |
| Database heavy | performance-analyzer |
| Frontend components | ui-reviewer |
| Outdated packages | dependency-updater |
| Old framework version | migration-helper |

## Model selection guide

| Model class | Best for | Trade-off |
|---|---|---|
| Fast/cheap (haiku-class) | Simple repetitive checks | Fast, cheap, less thorough |
| Balanced (sonnet-class) | Most review/analysis | Recommended default |
| Strong reasoning (opus-class) | Complex migrations, architecture | Thorough, slower, costlier |

Always write `model` as `provider/model-id`.

## Tool access guide (via permission)

| Access level | Permission | Use case |
|---|---|---|
| Read-only | read/glob/grep/list=allow, edit/bash=deny | Reviews, analysis |
| Writing | + edit=allow | Code/docs generation |
| Full | + bash=ask/allow | Migrations, testing |

## A ready-made example: setup-architect

This repo already ships `.opencode/agents/setup-architect.md` — a read-only analyst (read/glob/grep/list/skill=allow, edit/task=deny, bash/webfetch/websearch=ask) that runs the recommender workflow in isolation. Recommend dispatching it when the user wants a locked-down analysis pass.
