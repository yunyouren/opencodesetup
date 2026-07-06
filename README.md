# OpenCode Setup Assistant

[中文说明](README.zh-CN.md)

This repository ports the core idea of Anthropic's Claude Code Setup plugin into an OpenCode-native, read-only setup recommender.

Its job is simple: inspect a repository, identify real signals, and recommend the smallest useful OpenCode configuration without auto-editing user files or auto-enabling risky integrations.

## What it does

The assistant recommends OpenCode configuration across these surfaces:

- `opencode.jsonc` baseline config
- `AGENTS.md` project instructions
- `.opencode/skills/*/SKILL.md`
- `.opencode/commands/*.md`
- `.opencode/agents/*.md`
- `opencode.jsonc` `mcp` snippets
- `opencode.jsonc` `permission` snippets
- `.opencode/plugins/*.ts` only when truly necessary, and only with an explicit risk warning

The recommender is intentionally **read-only**:

- it prints snippets
- it explains why they are recommended
- it does not modify user config automatically
- it does not auto-enable MCP servers or plugins

## Current capabilities

This repo currently ships:

- `/setup` — analyze the current repo and recommend OpenCode configuration
- `/setup-eval <fixture-path>` — evaluate recommender behavior against a fixture and checklist
- `setup-architect` — a permission-locked read-only subagent for isolated analysis
- `opencode-automation-recommender` — the shared skill and reference tables behind the recommendations

The recommendation output is organized into four tiers:

- `Baseline` — missing essentials such as `opencode.jsonc`, `AGENTS.md`, or core permissions
- `High confidence` — recommendations directly supported by strong repository signals
- `Optional` — useful but non-essential additions
- `Skipped` — already configured, too risky, or not supported by real signals

## Quick start

### 1. Restart OpenCode

This project uses project-level OpenCode config. After changes to `opencode.jsonc`, `AGENTS.md`, or `.opencode/`, restart OpenCode so it reloads the config.

### 2. Run the recommender

In OpenCode, ask for a setup recommendation or invoke:

```text
/setup
```

You can also focus on a category:

```text
/setup mcp
/setup skills
/setup permissions
```

### 3. Review snippets manually

The assistant prints suggestions only. Paste accepted snippets into your target repo yourself.

### 4. Check the current baseline config

This repo already includes a minimal root `opencode.jsonc` and `AGENTS.md` so the project itself follows the same baseline it recommends to others:

- no MCP is enabled by default
- no plugin is enabled by default
- edit access to OpenCode scaffolding is denied or gated
- `claude-plugins-official/` is treated as read-only upstream reference material

## Repository layout

```text
.opencode/
  agents/
    setup-architect.md
  commands/
    setup.md
    setup-eval.md
  skills/
    opencode-automation-recommender/
      SKILL.md
      references/
docs/
  evals/
    setup-recommender-checklist.md
fixtures/
  empty-opencode/
  frontend-react/
  python-api/
  claude-plugin-source/
  existing-opencode-config/
opencode.jsonc
AGENTS.md
```

## How evaluation works

This repo includes fixtures and a checklist so the recommender can be regression-tested without touching real projects.

Run:

```text
/setup-eval fixtures/empty-opencode
/setup-eval fixtures/frontend-react
/setup-eval fixtures/python-api
/setup-eval fixtures/claude-plugin-source
/setup-eval fixtures/existing-opencode-config
```

Each evaluation should stay read-only and report:

- fixture profile
- pass/fail expectations
- over-recommendations
- missing recommendations
- safety/schema issues
- source-line evidence for failures

## Safety boundaries

This project follows conservative defaults:

- never read or print `.env*`, `*.key`, `*.pem`, `credentials*`, `secrets*`, or `id_rsa*`
- never auto-enable MCP servers or plugins
- local MCP snippets must use `environment`, not `env`
- MCP snippets default to `"enabled": false`
- `opencode.jsonc` starters must include `"$schema": "https://opencode.ai/config.json"`
- permission examples use last-match-wins ordering: broad `*` first, specific rules last
- `.opencode/package.json`, `.opencode/package-lock.json`, `.opencode/node_modules/`, and `.opencode/.gitignore` are treated as OpenCode-generated scaffolding, not user intent

## For maintainers

When changing recommender behavior:

1. Update the skill or references under `.opencode/skills/opencode-automation-recommender/`
2. Keep `setup.md` and `setup-architect.md` aligned with the same rules
3. Re-run `/setup-eval` against all fixtures
4. Add a new fixture before adding a new recommendation rule when possible

Also keep these files aligned whenever recommendation rules change:

- `.opencode/skills/opencode-automation-recommender/SKILL.md`
- `.opencode/commands/setup.md`
- `.opencode/agents/setup-architect.md`
- `docs/evals/setup-recommender-checklist.md`

Recommended next fixture types:

- real `.opencode/plugins/*.ts` plugin fixture
- docs-heavy repository fixture
- monorepo fixture
- security-sensitive fixture

## Upstream reference

`claude-plugins-official/` is a read-only upstream clone used as source material for the original Claude Code Setup plugin ideas. It is intentionally kept out of this repo's git history.
