# Project Instructions

This repository ports Anthropic's Claude Code Setup plugin ideas into OpenCode project-level automations.

## Scope

- Keep project automations under `.opencode/`.
- Treat `claude-plugins-official/` as a read-only upstream reference clone unless the user explicitly asks to update it.
- Use fixtures under `fixtures/` and the checklist in `docs/evals/setup-recommender-checklist.md` to evaluate recommendation behavior.

## Safety

- Do not read, print, or modify `.env*`, `*.key`, `*.pem`, `credentials*`, `secrets*`, or `id_rsa*`.
- Do not write global OpenCode, Claude, or Codex config directories.
- Do not enable MCP servers or plugins automatically; print suggested snippets for user review.
- Do not create `.opencode/plugins/*.ts` without first explaining the risk and receiving explicit approval.
- Prefer read-only analysis for setup recommendations. Any generated config should be shown as a snippet before being applied.

## OpenCode Config Rules

- `opencode.jsonc` snippets must include `"$schema": "https://opencode.ai/config.json"`.
- Local MCP snippets use `type: "local"`, `command: [...]`, and `environment` (not `env`).
- MCP snippets default to `enabled: false`.
- Permission objects use last-match-wins ordering: broad `*` rules first, specific rules last.
- Ignore `.opencode/package.json`, `.opencode/package-lock.json`, `.opencode/node_modules/`, and `.opencode/.gitignore` as opencode-generated scaffolding, not user intent.

## Validation

- After changing recommender behavior, run or simulate `/setup-eval` against the fixtures.
- At minimum, check for regressions with `fixtures/empty-opencode`, `fixtures/frontend-react`, `fixtures/python-api`, `fixtures/claude-plugin-source`, and `fixtures/existing-opencode-config`.
