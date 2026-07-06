---
description: Evaluate the setup recommender against a fixture and checklist. Read-only; outputs pass/fail findings and does not modify files.
---

Evaluate the OpenCode setup recommender against fixture path: `$ARGUMENTS`

Use checklist: `docs/evals/setup-recommender-checklist.md`

## Hard rules

- Read-only. Do not create, edit, move, or delete files.
- Do not mutate `opencode.jsonc`, `AGENTS.md`, `.opencode/`, fixtures, or global config.
- Never read or print `.env*`, `*.key`, `*.pem`, `credentials*`, `secrets*`, or `id_rsa*`.
- Prefer `glob`/`read`/`grep` over `bash`; ask before any shell or network action.
- Do not run package installs, tests, formatters, or scripts.

## Procedure

1. Read the fixture files needed to identify signals.
2. Read the relevant section of `docs/evals/setup-recommender-checklist.md`.
3. Simulate what `/setup` should recommend for that fixture using `.opencode/skills/opencode-automation-recommender/SKILL.md`.
4. Compare expected vs actual-style recommendations.
5. Output:
   - Fixture profile
   - Pass/fail table for each expectation
   - Over-recommendations
   - Missing recommendations
   - Safety/schema issues
   - Suggested minimal fixes, without applying them

## Pass criteria

- All recommended items are grounded in fixture signals.
- No forbidden category is recommended.
- MCP snippets default to `enabled: false`.
- Local MCP snippets use `environment`, not `env`.
- `opencode.jsonc` starter snippets include `"$schema": "https://opencode.ai/config.json"`.
- Permission examples follow last-match-wins ordering: broad `*` first, specific patterns last.
