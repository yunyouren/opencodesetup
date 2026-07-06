# Plugins & Extensions Recommendations (OpenCode)

In OpenCode, "plugins" are TypeScript modules that register hooks (the equivalent of Claude Code's `hooks` + `plugins` combined). They are auto-discovered from `.opencode/plugins/*.ts` (or `.opencode/plugin/`), or listed in `opencode.jsonc` -> `plugin` (an array).

> Plugins run arbitrary TypeScript with tool/config mutation powers. **Recommend them sparingly** — prefer built-in `formatter`/`lsp` config or `permission` rules first. See `opencode-hooks-reference.md` for the risk statement. Never create a plugin without explicit user approval.

## Config shape

`plugin` is an **array** of strings or `[name, options]` tuples (NOT an object):

```jsonc
{
  "plugin": [
    "opencode-gemini-auth",            // npm spec, latest
    "opencode-foo@1.2.3",              // npm spec, pinned
    "./local-plugin.ts",               // file path, relative to declaring config
    "file:///abs/path/plugin.js",      // file URL
    ["opencode-bar", { "option": "value" }]  // tuple with options
  ]
}
```

Auto-discovery: any `*.ts` or `*.js` file in `.opencode/plugin/` or `.opencode/plugins/` is loaded with no config entry needed.

## Plugin module shape

```ts
import type { Plugin } from "@opencode-ai/plugin"

export default (async ({ client, project, directory, $ }) => {
  return {
    config: (cfg) => {
      // mutate the live merged config
    },
    "tool.execute.before": async (input, output) => {
      // mutate output.args before the tool runs
    },
    "tool.execute.after": async (input, output) => {
      // run after a tool executes
    },
  }
}) satisfies Plugin
```

The export is a **function** (not a plain object) that returns a `Hooks` object. Return `{}` if there is nothing to register. Hook surface listed in `opencode-hooks-reference.md`.

## When to recommend a plugin

Recommend a plugin **only when** all of these hold:

- An automatic, tool-event-driven action is genuinely needed (format-on-edit, block-edit, post-edit test run), AND
- The built-in `formatter`/`lsp` config and `permission` rules cannot cover it, AND
- The user has approved the risk (plugins = arbitrary code with mutation powers).

## When NOT to recommend a plugin

- "Block edits to `.env`" -> use `permission` (`edit: { ".env*": "deny" }`), not a plugin.
- "Auto-format on save" -> use `formatter` built-in, not a plugin.
- "Type-check on edit" -> use `lsp` built-in, not a plugin.
- "Auto-lint" -> use `formatter` built-in.

## Plugin-like official/community packages

OpenCode's plugin ecosystem is npm-based. Common needs and their non-plugin alternatives:

| Need | Non-plugin approach | Plugin (if non-plugin insufficient) |
|---|---|---|
| Auto-format | `formatter` config | custom `tool.execute.after` plugin |
| LSP / type-check | `lsp` config | custom plugin (rare) |
| Block sensitive edits | `permission` | custom `permission.ask` plugin (rare) |
| Provider auth (e.g. Gemini) | `provider` config | dedicated auth plugin (e.g. `opencode-gemini-auth`) |
| Post-edit test run | manual / command | custom `tool.execute.after` plugin |

## Claude Code -> OpenCode plugin mapping

| Claude Code | OpenCode |
|---|---|
| `.claude/settings.json` hooks | `.opencode/plugins/*.ts` (auto-discovered) or `opencode.jsonc` -> `plugin` array |
| Claude `PostToolUse` hook | opencode `tool.execute.after` |
| Claude `PreToolUse` hook | opencode `tool.execute.before` |
| Claude `/plugin install <name>` | add `"name"` to `opencode.jsonc` -> `plugin` array, or drop `*.ts` in `.opencode/plugins/` |
| Claude plugin (bundle of skills/commands/agents/hooks) | OpenCode: ship skills/commands/agents as files + a `*.ts` plugin for hooks; distribute via npm or git path |

## Risk reminder

Plugins can mutate tool args, tool outputs, the merged config, shell env, and permission prompts. Treat them as privileged code. The recommender must **only print** plugin recommendations and risks, never create a `.ts` file.
