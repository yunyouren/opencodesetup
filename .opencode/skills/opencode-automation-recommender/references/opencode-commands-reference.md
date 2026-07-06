# Commands Recommendations (OpenCode)

Commands are explicit, user-invoked workflows triggered with `/name`. In OpenCode they live in `.opencode/commands/<name>.md` and are auto-discovered. The file body is the prompt template opencode runs on invocation.

> Commands are the OpenCode equivalent of Claude Code slash commands. Recommend them for entry points the user invokes deliberately, especially side-effecting workflows.

## Command file (OpenCode)

`.opencode/commands/deploy.md`:

```yaml
---
description: Deploy the app to the named environment.
agent: build
model: anthropic/claude-sonnet-4-6
subtask: false
---
Deploy to: $ARGUMENTS

1. Run the build (ask before running shell commands).
2. Run tests; abort on failure.
3. Deploy via the project's deploy script.
4. Report the deployment URL.
```

Rules:

- The **body** (everything below the frontmatter) is the `template` and is required. Do not also put `template:` in the frontmatter.
- `$ARGUMENTS` = everything the user typed after the command. `$1`, `$2`, ... = positional args.
- Frontmatter fields: `description` (optional), `agent` (optional; selects which agent runs the template), `model` (optional, provider-prefixed), `variant` (optional), `subtask` (boolean; run as a forked subtask of the current agent).
- If `agent` points at a `mode: subagent` agent, prefer `subtask: true` so it runs in a forked context. Note: `ask`-permission prompts may not be answerable in a forked subtask, so bash/web may be effectively denied — that is the safe default.

## When to recommend a command vs a skill

| Need | Recommend |
|---|---|
| User explicitly invokes an entry point (`/deploy`, `/setup`, `/pr-check`) | command |
| Side-effecting workflow (deploy, commit, release) | command (so the user opts in deliberately) |
| Auto-triggered background expertise (apply when writing code) | skill |
| Repeated prompt the user retypes often | command |

## Command examples

### /setup (already shipped in this repo)
`.opencode/commands/setup.md` — read-only analysis that recommends an OpenCode automation setup. Good template for read-only reporting commands.

### /pr-check
```yaml
---
description: Review the current PR against the project checklist.
agent: build
---
Review the PR against .opencode/skills/pr-check/checklist.md.
Diff (ask before running): git diff main...HEAD
Mark each checklist item pass/fail with a one-line reason.
```

### /release-notes
```yaml
---
description: Generate release notes from commits since the last tag.
agent: build
---
Commits since last tag (ask before running): git log $(git describe --tags --abbrev=0)..HEAD --oneline
Group by type (feat/fix/docs/...), write user-friendly descriptions, highlight breaking changes.
```

### /gen-test
```yaml
---
description: Generate tests for the given file following project conventions.
agent: build
---
Generate tests for: $ARGUMENTS
Match the project's test framework and directory conventions. Place tests alongside or in tests/.
```

## Quick reference: signal -> command

| Codebase signal | Command |
|---|---|
| PR-based workflow | /pr-check |
| Releases/tags | /release-notes |
| Test suite | /gen-test |
| Onboarding | /setup-dev |
| First-time OpenCode setup | /setup |

## Registering commands

Commands in `.opencode/commands/` (or `.opencode/command/`) are auto-discovered — no `opencode.jsonc` entry needed. You can also inline a command in `opencode.jsonc` under `command` (object keyed by name, `template` required), but prefer files for anything non-trivial.
