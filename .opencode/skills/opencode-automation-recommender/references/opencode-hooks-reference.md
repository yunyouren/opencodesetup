# Hooks Recommendations (OpenCode) — documentation only

> **This file documents how hooks map to OpenCode plugins. It does NOT create any plugin/hook code.** Plugins are TypeScript modules with tool/config mutation powers — equivalent to running arbitrary code. **Never create a `.ts` plugin without explaining the risk and getting explicit user approval.** Prefer the built-in `formatter`/`lsp` config or `permission` rules over a plugin whenever possible.

In Claude Code, hooks live in `.claude/settings.json` and fire on tool events. In OpenCode, the equivalent is a **plugin** (`.opencode/plugins/*.ts`, auto-discovered, or listed in `opencode.jsonc` -> `plugin`). A plugin is a function returning a `Hooks` object.

## Plugin shape

```ts
import type { Plugin } from "@opencode-ai/plugin"

export default (async ({ client, project, directory, $ }) => {
  return {
    // Mutate args before a tool runs:
    "tool.execute.before": async (input, output) => {
      // output.args is mutable in place
    },
    // Run after a tool runs:
    "tool.execute.after": async (input, output) => {
      // e.g. auto-format an edited file
    },
    // Gate permission prompts:
    "permission.ask": async (input, output) => {},
    // Mutate the merged config once on init:
    config: (cfg) => {},
  }
}) satisfies Plugin
```

Hook surface (mutate `output` in place; return `void`): `event`, `config`, `chat.message`, `chat.params`, `chat.headers`, `tool.execute.before`, `tool.execute.after`, `tool.definition`, `command.execute.before`, `shell.env`, `permission.ask`, plus experimental transforms. Special object-shaped hooks: `tool`, `auth`, `provider`.

## Auto-format / auto-lint (prefer built-ins first)

OpenCode has a built-in `formatter` config — recommend that BEFORE a plugin:

```jsonc
{
  "formatter": {
    "src": { "command": ["npx", "prettier", "--write"], "extensions": [".ts", ".tsx"] }
  }
}
```

| Detection | Built-in approach | Plugin only if |
|---|---|---|
| Prettier (`.prettierrc`) | `formatter` with `prettier --write` | you need post-edit hooks beyond formatting |
| ESLint (`.eslintrc`) | `formatter` with `eslint --fix` | you need lint-gating on specific tools |
| Ruff/Black (Python) | `formatter` with `ruff format` / `black` | same |
| gofmt / rustfmt | `formatter` with `gofmt -w` / `rustfmt` | same |

## Type checking

| Detection | Approach |
|---|---|
| `tsconfig.json` | `lsp` built-in with a TS language server (preferred) over a plugin |
| mypy/pyright | `lsp` built-in |

OpenCode `lsp` config:

```jsonc
{ "lsp": { "typescript": { "command": ["typescript-language-server", "--stdio"], "extensions": [".ts", ".tsx"] } } }
```

## Protection rules (prefer `permission` over a plugin)

Most "block edits to X" needs are better served by `permission` in `opencode.jsonc` (no code, no RCE risk):

```jsonc
{
  "permission": {
    "edit": { "*": "ask", ".env*": "deny", "*lock*": "deny" },
    "bash": { "*": "ask", "rm *": "deny", "git commit *": "ask" }
  }
}
```

| Detection | Permission rule |
|---|---|
| `.env` files present | `edit: { ".env*": "deny" }` |
| Lock files present | `edit: { "*lock*": "deny" }` |
| Git internals | `edit: { ".git/**": "deny" }` |
| Secrets/keys | `edit: { "*.key": "deny", "*.pem": "deny", "credentials*": "deny" }` |

`permission` is a string action or `{ "pattern": action }`; **last match wins**, so put `*` first and specific patterns last.

## Test runner hooks (plugin territory)

Running related tests after an edit is a `tool.execute.after` plugin (no clean built-in equivalent). Only recommend if the user wants tight post-edit test loops, and flag the risk.

## Notification hooks

OpenCode has no direct "notification" hook surface like Claude's `Notification` matcher. Approximate via a plugin listening on `event` and filtering, or skip — this is rarely essential.

## Quick reference: detection -> recommendation

| If you see | Prefer | Plugin only if needed |
|---|---|---|
| Prettier/ESLint/Ruff/Black config | `formatter` built-in | post-edit hook beyond format |
| tsconfig / mypy / pyright | `lsp` built-in | custom gating |
| `.env` / lock files / secrets | `permission` deny rules | — |
| Tests dir, want post-edit test run | — (manual) | `tool.execute.after` plugin |
| Multitasking notifications | — | `event` plugin (rarely needed) |

## Risk statement (read before creating any plugin)

- A plugin is arbitrary TypeScript executed by opencode with access to `tool.execute.before/after` (can mutate tool args and outputs), `config` (can mutate the merged config), `shell.env`, and `permission.ask`.
- A malicious or buggy plugin can read secrets, alter files, suppress permission prompts, or change config silently. Treat plugin code like any code that runs with your privileges.
- Therefore: **do not create a plugin from this recommender.** Only print the recommendation and the risk; the user must explicitly approve and ideally review the plugin source themselves.
