# Skills Recommendations (OpenCode)

Skills are packaged expertise with workflows and reference materials. In OpenCode they live in `.opencode/skills/<name>/SKILL.md` and are auto-discovered. A skill is surfaced to the model by its `description`, so write the description to cover **what** it does and **when** to trigger it.

> These are common patterns. Use `websearch`/`webfetch` (with approval) to find skill ideas specific to the codebase's tools and frameworks.

## Skill structure

```
.opencode/skills/
└── my-skill/
    ├── SKILL.md           # Main instructions (required, named exactly SKILL.md)
    ├── template.yaml      # Optional supporting files
    └── examples/          # Optional reference examples
```

## Frontmatter (OpenCode)

Only these fields are meaningful; `name` and `description` are the important ones.

```yaml
---
name: my-skill
description: One sentence covering what this skill does AND when to trigger it. Front-load literal keywords/filenames the user is likely to say. Use "Use ONLY when..." to keep it quiet on adjacent topics.
license: MIT            # optional
compatibility: ">=1.0"  # optional
metadata:               # optional, string->string map
  author: team
---
```

Rules:

- `name` is required, lowercase hyphen-separated, up to 64 chars, and **must equal the folder name**.
- `description` is effectively required — skills without one are filtered out and never surfaced. Third person ("Use when...").
- **No** `disable-model-invocation` / `user-invocable` / `tools` / `context` / `agent` fields (those are Claude Code). OpenCode has no direct equivalent; control surfacing via the `description` and, for side-effecting workflows, put them in a user-invoked command instead.

## Invocation control (OpenCode approximation of Claude flags)

| Claude Code flag | OpenCode approach |
|---|---|
| `disable-model-invocation: true` (user-only) | Make it a command (`.opencode/commands/<n>.md`) the user invokes explicitly, or gate side effects inside the skill body with "ask first". |
| `user-invocable: false` (model-only background) | Keep as a skill with a description that triggers automatically; do not advertise it as a command. |
| `allowed-tools` restriction | Not supported in skill frontmatter; instead dispatch a permission-locked subagent to run the skill. |

## Custom skill examples

### API documentation with OpenAPI template
`.opencode/skills/api-doc/SKILL.md` + `openapi-template.yaml`. Skill body: read the endpoint, extract path/method/params/schemas, fill the template, output YAML.

### Database migration generator
`.opencode/skills/create-migration/SKILL.md` + `scripts/validate-migration.sh`. Body: generate migration with timestamp prefix, include up/down, run validation, report issues.

### Test generator
`.opencode/skills/gen-test/SKILL.md` + `examples/`. Body: analyze source, identify functions to test, generate tests matching conventions, place in test dir.

### Component generator
`.opencode/skills/new-component/SKILL.md` + `templates/`. Body: scaffold component + tests + stories from templates, replace `{{ComponentName}}`/`{{component-name}}`.

### PR review with checklist
`.opencode/skills/pr-check/SKILL.md` + `checklist.md`. Body: gather diff via `git` (ask bash first), review against checklist, mark each item.

### Release notes generator
`.opencode/skills/release-notes/SKILL.md`. Body: `git log` since last tag (ask bash first), group by type, write user-friendly notes, highlight breaking changes.

### Project conventions (background knowledge)
`.opencode/skills/project-conventions/SKILL.md` with a description that auto-triggers when writing/reviewing code. Body: naming conventions, patterns, forbidden constructs.

### Environment setup
`.opencode/skills/setup-dev/SKILL.md` + `scripts/check-prerequisites.sh`. Body: check prerequisites, install deps, copy env template, set up DB, verify. (Side-effecting — consider making it a command instead.)

## Quick reference: signal -> skill to create

| Codebase signal | Skill | Invocation style |
|---|---|---|
| API routes | api-doc (with OpenAPI template) | command or skill |
| Database project | create-migration (with validation script) | command (side effects) |
| Test suite | gen-test (with examples) | command (side effects) |
| Component library | new-component (with templates) | command (side effects) |
| PR workflow | pr-check (with checklist) | command |
| Releases | release-notes (with git context) | command |
| Code style | project-conventions | skill (background) |
| Onboarding | setup-dev (with prereq script) | command (side effects) |

## Registering skills from other locations

In `opencode.jsonc`:

```jsonc
{
  "skills": {
    "paths": [".opencode/skills", "/abs/path/to/skills"],
    "urls": ["https://example.com/.well-known/skills/"]
  }
}
```

`skills` is an object with `paths` and/or `urls` (not an array). `paths` are scanned recursively for `**/SKILL.md`.
