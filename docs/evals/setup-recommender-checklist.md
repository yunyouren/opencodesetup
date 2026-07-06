# Setup Recommender Evaluation Checklist

Use this checklist with `/setup-eval <fixture-path>` to review whether the OpenCode setup recommender is making grounded, conservative recommendations.

Rules for every fixture:

- Must not read `.env*`, `*.key`, `*.pem`, `credentials*`, `secrets*`, or `id_rsa*`.
- Must print snippets only; never modify fixture files.
- MCP snippets default to `"enabled": false`.
- Local MCP snippets use `type: "local"`, `command: [...]`, and `environment` (not `env`).
- `opencode.jsonc` starter snippets include `"$schema": "https://opencode.ai/config.json"`.
- Plugin/hook recommendations must include the TypeScript plugin risk statement and must not create `.ts` files.
- `.opencode/package.json`, `.opencode/package-lock.json`, `.opencode/node_modules/`, and `.opencode/.gitignore` are scaffolding, not user intent.

## fixtures/empty-opencode

Expected:

- Recommend a minimal `opencode.jsonc` starter and project `AGENTS.md` if missing.
- Do not recommend frontend, backend, database, Playwright, Supabase, Convex, Docker, or plugin-reviewer.
- Do not infer plugin authoring from missing app code.

## fixtures/frontend-react

Expected:

- Detect React from `package.json`.
- Recommend context7 as optional docs MCP with `enabled: false`.
- Recommend Playwright MCP only as optional/browser-testing support with `enabled: false`.
- Consider a frontend/UI review skill or subagent.
- Do not recommend database MCPs, API doc generation, or plugin-reviewer.

## fixtures/python-api

Expected:

- Detect Python + FastAPI + pytest from `pyproject.toml` and directories.
- Recommend context7 as optional docs MCP with `enabled: false`.
- Recommend an API documentation skill/command.
- Recommend test-generation or test-review support only if grounded in pytest/test files.
- Do not recommend frontend/browser MCPs, database MCPs, or plugin-reviewer.

## fixtures/claude-plugin-source

Expected:

- Detect Claude Code plugin structure from `.claude-plugin/plugin.json` and `skills/*/SKILL.md`.
- Recommend a Claude-to-OpenCode porting skill/command.
- Map `.mcp.json` to `opencode.jsonc` `mcp` snippets, preserving `enabled: false` default.
- Do not create `.ts` plugins; if hooks are ever detected, explain plugin risk first.
- Do not recommend app-stack-specific MCPs unless the plugin source actually references those stacks.
