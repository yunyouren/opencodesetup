---
description: Analyze the current repo and recommend a tailored OpenCode automation setup (skills, commands, agents, MCP servers, permissions, plugins). Read-only — prints recommendations and ready-to-paste config snippets; does not modify files.
---

Perform a read-only OpenCode setup analysis for this repository, then output a recommendations report.

Follow the three-phase workflow and reference tables in the `opencode-automation-recommender` skill at `.opencode/skills/opencode-automation-recommender/SKILL.md` (and its `references/`).

Arguments (optional): `$ARGUMENTS`

- If a category is named (one of: `mcp`, `skills`, `agents`, `commands`, `hooks`, `permissions`, `plugins`), focus on that category and give 3-5 recommendations.
- Otherwise, give the top 1-2 per relevant category and skip irrelevant ones.

## Hard rules

- **Read-only.** Do not create/edit/move/delete files. Do not mutate `opencode.jsonc`, `AGENTS.md`, or any config. Do not write to global directories (`~/.config/opencode`, `~/.claude`, `~/.codex`).
- **Never read or print** `.env`, secrets, tokens, API keys, private keys. Skip `.env*`, `*.key`, `*.pem`, `credentials*`, `secrets*`, `id_rsa*`.
- **Never run install scripts** (`npm install`, etc.) and **never auto-enable high-risk MCP**.
- All config snippets must validate against `https://opencode.ai/config.json`:
  - `opencode.jsonc` starter snippets must include `"$schema": "https://opencode.ai/config.json"` as the first key.
  - Local MCP: `type: "local"`, `command: [...]` (string array), env vars in `environment` (NOT `env`), default `enabled: false`.
  - Remote MCP: `type: "remote"`, `url`, optional `headers` / `oauth`, default `enabled: false`.
  - `permission`: string action OR `{ "pattern": action }`; broad first, narrow last (last match wins).
  - `model` values carry a provider prefix (e.g. `anthropic/claude-sonnet-4-6`).
- **Print snippets only** — tell the user which file each belongs in; let them paste it. Do not write config yourself.
- For any hook/plugin (`.ts`) recommendation, **explain the risk first and do not create it** unless the user explicitly approves.

## Phase 1 — Analyze

Use `glob`/`read`/`grep` (prefer these over `bash`). Detect manifests, framework, deps, tests, CI, and existing OpenCode config (`opencode.jsonc`, `AGENTS.md`, `.opencode/`). Capture the nine signal categories listed in the skill (language/framework, frontend, backend, database, external APIs, testing, CI/CD, issue tracking, docs). Skip secret files.

Ignore opencode auto-generated scaffolding as signals: `.opencode/package.json`, `.opencode/package-lock.json`, `.opencode/node_modules/`, and `.opencode/.gitignore` bootstrap the plugin SDK and do NOT prove a plugin-authoring workflow. Only count user-authored `.opencode/plugins/*.ts` or `opencode.jsonc` `plugin` entries as real plugin signals.

## Phase 2 — Recommend

Map signals to OpenCode recommendations via the skill's reference tables (`references/opencode-*-reference.md`).

## Phase 3 — Report

Use the report format in the skill. For each recommendation include: the triggering signal, the target file/config location, and a validated, ready-to-paste snippet. End with an offer to give more in any category or to walk through implementing an item (still read-only).

## Optional: locked-down analysis

If you (the running agent) want an isolated, permission-locked pass instead of running this inline, dispatch the `setup-architect` subagent via the task tool. Note: in a forked subtask, `ask`-permission prompts may not be answerable interactively, so the architect's `bash`/`webfetch`/`websearch` may be effectively denied — that is the safe default for a locked analyzer, and it will mark unverified items accordingly.
