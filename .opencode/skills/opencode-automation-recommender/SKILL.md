---
name: opencode-automation-recommender
description: Analyze a codebase and recommend tailored OpenCode automations — skills, commands, subagents, MCP servers, permissions, and plugins. Use when the user asks for automation recommendations, wants to optimize their OpenCode setup, asks how to first set up OpenCode for a project, mentions improving OpenCode workflows, or wants to know what OpenCode features they should use. Use ONLY for read-only analysis and recommendations; never for writing config.
---

# OpenCode Automation Recommender

Analyze codebase patterns to recommend tailored OpenCode automations across all extensibility surfaces.

**This skill is read-only.** It analyzes the codebase and outputs recommendations plus ready-to-paste config snippets. It does NOT create or modify files, does NOT write to `opencode.jsonc` or `AGENTS.md`, and does NOT write to global directories (`~/.config/opencode`, `~/.claude`, `~/.codex`). The user implements recommendations themselves.

## Output guidelines

- **Recommend 1-2 per confidence tier** — surface only grounded items; don't force every category to appear.
- **If the user names a category**, focus there and give 3-5 options.
- **Go beyond the reference tables** — use `websearch`/`webfetch` (with approval when required) to find tool/framework-specific recommendations not in the tables.
- **Tell users they can ask for more** — end by noting they can request more in any category.

## Automation surfaces (OpenCode)

| Surface | Best for | Where it lives |
|---|---|---|
| **MCP servers** | External integrations (docs, browsers, DBs, APIs, GitHub) | `opencode.jsonc` -> `mcp` |
| **Skills** | Packaged expertise & workflows, auto-triggered by description | `.opencode/skills/<name>/SKILL.md` |
| **Commands** | Explicit user-invoked workflows (`/name`) | `.opencode/commands/<name>.md` |
| **Subagents** | Specialized, permission-scoped analysts/reviewers | `.opencode/agents/<name>.md` (`mode: subagent`) |
| **Permissions** | Tool gating (allow/ask/deny) per tool & pattern | `opencode.jsonc` -> `permission` |
| **Plugins** | TypeScript hooks (tool.execute.before/after, config, permission.ask) | `.opencode/plugins/*.ts` (auto-discovered) or `opencode.jsonc` -> `plugin` array |

## Claude Code to OpenCode concept map

| Claude Code | OpenCode |
|---|---|
| `CLAUDE.md` | `AGENTS.md` (listed in `opencode.jsonc` -> `instructions`) |
| `.claude/skills/<n>/SKILL.md` | `.opencode/skills/<n>/SKILL.md` |
| `.claude/agents/*.md` | `.opencode/agents/*.md` (`mode: subagent`) |
| `.claude/commands` (slash) | `.opencode/commands/*.md` |
| `.mcp.json` | `opencode.jsonc` -> `mcp` (local uses `environment`, not `env`) |
| `.claude/settings.json` hooks | `.opencode/plugins/*.ts` (Plugin hooks) |
| `.claude/settings.json` permissions | `opencode.jsonc` -> `permission` |
| `claude mcp add X` | edit `opencode.jsonc` -> `mcp` |
| `/plugin install` | `opencode.jsonc` -> `plugin` array, or drop `*.ts` in `.opencode/plugins/` |
| `disable-model-invocation` / `user-invocable` | (no direct equivalent) — control via `description` wording + per-agent `permission`/`tools` |

## Workflow

### Phase 1 — Codebase analysis

Prefer `glob`/`read`/`grep` over `bash`. Capture:

| Category | Look for | Informs |
|---|---|---|
| Language/framework | manifests (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`) | skills, MCP, plugins |
| Frontend | React/Vue/Angular/Next/Svelte in deps | Playwright MCP, frontend skills |
| Backend | Express/FastAPI/Django/Spring in deps | API/docs skills, MCP |
| Database | Prisma/Drizzle/Supabase/Convex/raw SQL | DB MCP |
| External APIs | Stripe/Twilio/SendGrid/AWS SDK | context7 MCP |
| Testing | jest/pytest/vitest/playwright configs | test skills, subagents |
| CI/CD | `.github/workflows`, CircleCI | GitHub MCP |
| Issue tracking | Linear/Jira refs | issue-tracker MCP |
| Docs | OpenAPI/JSDoc/docstrings | docs skills |
| Existing OpenCode | `opencode.jsonc`, `AGENTS.md`, `.opencode/` | what's already configured |

**Never read:** `.env*`, `*.key`, `*.pem`, `credentials*`, `secrets*`, `id_rsa*`.

**Ignore as signals (opencode auto-generated scaffolding, NOT user intent):** `.opencode/package.json`, `.opencode/package-lock.json`, `.opencode/node_modules/`, and `.opencode/.gitignore` are auto-created by opencode to bootstrap the `@opencode-ai/plugin` SDK in any `.opencode/` project. Do NOT treat `@opencode-ai/plugin` in `.opencode/package.json` as a user dependency or as evidence of a plugin-authoring workflow. Only count an actual user-authored `.ts` plugin in `.opencode/plugins/` or an entry in `opencode.jsonc` -> `plugin` as a real plugin signal.

### Phase 2 — Generate recommendations

Use the reference tables:

- [opencode-mcp-reference.md](references/opencode-mcp-reference.md)
- [opencode-skills-reference.md](references/opencode-skills-reference.md)
- [opencode-agents-reference.md](references/opencode-agents-reference.md)
- [opencode-commands-reference.md](references/opencode-commands-reference.md)
- [opencode-hooks-reference.md](references/opencode-hooks-reference.md) — **docs only; no `.ts` creation**
- [opencode-extensions-reference.md](references/opencode-extensions-reference.md)

Baseline branch: if the repo has no application/framework signals and no project OpenCode config, recommend only a minimal `opencode.jsonc` starter (with `"$schema": "https://opencode.ai/config.json"` and `instructions: ["AGENTS.md"]`) plus a project `AGENTS.md`; skip MCP, skills, commands, subagents, and plugins unless there is a real signal.

### Phase 3 — Output report

Use this format. **Only 1-2 per confidence tier** unless the user asked for a specific category. Put already-present items in `Skipped` with `already configured`.

```
## OpenCode Automation Recommendations

### Codebase Profile
- Type: [detected language/runtime]
- Framework: [detected framework]
- Key libraries: [relevant deps]

---

### Baseline

#### [opencode.jsonc / AGENTS.md / permissions]
Status: recommend / skipped
Why: [missing baseline config OR already configured]
Snippet/location: [ready-to-paste snippet or existing path]

---

### High confidence

#### [skill / command / subagent / MCP / permission]
Why: [strong signal from files/deps/config]
Create/add: [target path or config location]

---

### Optional

#### [MCP / formatter / helper]
Why: [useful but not required]
Add to: [target path or config location]

---

### Skipped

#### [item/category]
Reason: [already configured / no signal / too risky / plugin not warranted]

Examples:
- `opencode.jsonc` — already configured
- `AGENTS.md` — already configured
- `/setup` — already configured
- `setup-architect` — already configured
- Plugins/hooks — no real plugin signal; `.ts` plugins require explicit approval

---

### Snippet examples

#### MCP server
#### [name]
Why: [signal-based reason]
Add to `opencode.jsonc` -> `mcp`:
{ "type": "local", "command": [...], "environment": { ... }, "enabled": false }

#### Skill
#### [name]
Why: [reason]
Create: `.opencode/skills/[name]/SKILL.md`

#### Command
#### [name]
Why: [reason]
Create: `.opencode/commands/[name].md`

#### Subagent
#### [name]
Why: [reason]
Create: `.opencode/agents/[name].md` (mode: subagent)

#### Permission
#### [rule]
Why: [reason]
Add to `opencode.jsonc` -> `permission`:
{ "edit": { "*": "ask", ".env*": "deny" } }

#### Plugin (hook) — higher risk
#### [hook]
Why: [reason]
Risk: plugins run as TypeScript with tool/config mutation powers. Create `.opencode/plugins/[name].ts` only after explicit approval.
```

End with: "Want more? Ask for additional recommendations in any category. Want help implementing? I can walk through any item above (still read-only for analysis)."

## Decision framework

- **Recommend baseline when:** `opencode.jsonc`, `AGENTS.md`, or protective `permission` is missing. If present, put it in `Skipped` as `already configured`.
- **Recommend MCP when:** external service/docs/browser/team-tool integration needed.
- **Recommend skills when:** repeated prompts/workflows, project-specific tasks, packaged expertise.
- **Recommend commands when:** explicit user-invoked entry points (`/deploy`, `/setup`, `/pr-check`).
- **Recommend subagents when:** specialized isolated expertise (security/perf review), parallel work, permission-locked analysis.
- **Recommend permissions when:** protect secrets/lock files, gate edits/bash, enforce plan-mode-style deny.
- **Recommend plugins (hooks) when:** automatic post-edit actions (format/lint/typecheck), block edits — but only if the built-in `formatter`/`lsp` config is insufficient; plugins are higher-risk.

## Config-snippet rules (must validate against the schema)

- `opencode.jsonc` starter: always include `"$schema": "https://opencode.ai/config.json"` as the first key so the editor catches mistakes; never omit it.
- Local MCP: `type: "local"`, `command: [array]`, env in `environment` (NOT `env`). Default printed snippets to `"enabled": false` — the user opts in; never auto-enable. Tag risk tier (low = docs/read-only, high = filesystem/db/cloud/auth).
- Remote MCP: `type: "remote"`, `url`, optional `headers` (supports `{env:VAR}`), optional `oauth`. Same `"enabled": false` default.
- `permission`: string action OR object `{ "pattern": action }`; **last match wins**, so put `*`-rules first, specific patterns last.
- `model`: always `provider/model-id` (e.g. `anthropic/claude-sonnet-4-6`).
- Skill frontmatter: `name` (must equal the folder name) + `description` only, optionally `license`/`compatibility`/`metadata`.
- Agent frontmatter: `description`, `mode`, optional `model`/`permission`/`steps`/`hidden`/`color`/`temperature`/`top_p`.
- Command frontmatter: `description` (optional), optional `agent`/`model`/`variant`/`subtask`; body is the template (required), `$ARGUMENTS`/`$1` for args.

## Notes on gaps vs Claude Code

- OpenCode skills have **no** `disable-model-invocation`/`user-invocable` flags. Control surfacing via the `description` ("Use ONLY when...") and, for side-effecting skills, keep them in commands the user invokes explicitly.
- OpenCode hooks = TypeScript plugins (more powerful, more risk). Prefer built-in `formatter`/`lsp` config over a plugin when possible.
