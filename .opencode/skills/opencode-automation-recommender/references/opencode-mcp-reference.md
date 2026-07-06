# MCP Server Recommendations (OpenCode)

MCP servers extend opencode with external tools/services. In OpenCode they are configured in `opencode.jsonc` under the top-level `mcp` object (NOT a separate `.mcp.json` file).

> These are common servers. Use `websearch`/`webfetch` (with approval when required) to find servers specific to the codebase's services. **Never auto-enable high-risk MCP** — print the snippet and let the user decide.

## Setup & team sharing

- **Project config** (`opencode.jsonc` at repo root): available in this project. Check into git so the whole team gets the same servers.
- **Global config** (`~/.config/opencode/opencode.json`): available across projects. (Recommender must NOT write here.)
- Disable an inherited server with `{ "enabled": false }`.

## Config shape (must validate against the schema)

> All snippets below default to `"enabled": false` — the user opts in; never auto-enable. Tag risk tier (low = docs/read-only, high = filesystem/db/cloud/auth).

Local server — `type` required, `command` is a **string array**, env vars go in `environment` (NOT `env`):

```jsonc
{
  "mcp": {
    "context7": {
      "type": "local",
      "command": ["npx", "-y", "@upstash/context7-mcp"],
      "environment": { "DEFAULT_MINIMUM_TOKENS": "10000" },
      "enabled": false
    }
  }
}
```

Remote server — `type` required, `url` required, optional `headers` (supports `{env:VAR}`), optional `oauth`:

```jsonc
{
  "mcp": {
    "github": {
      "type": "remote",
      "url": "https://api.githubcopilot.com/mcp/",
      "enabled": false,
      "headers": { "Authorization": "Bearer {env:GITHUB_TOKEN}" }
    }
  }
}
```

Optional fields: `cwd` (local), `timeout` (ms, default 5000).

## Documentation & knowledge

### context7
**Best for:** projects using popular libraries/SDKs where you want up-to-date docs.
Recommend when: React/Vue/Angular, Express/FastAPI/Django, Prisma/Drizzle, Stripe/Twilio, AWS/GCP SDKs, LangChain/OpenAI SDK.
Value: live docs instead of stale training data; fewer hallucinated APIs.

## Browser & frontend

### Playwright MCP
**Best for:** frontend projects needing browser automation, testing, screenshots.
Recommend when: React/Vue/Angular app, E2E tests, visual regression, debugging UI.
Snippet: `{ "type": "local", "command": ["npx", "-y", "@playwright/mcp"], "enabled": false }`

### Puppeteer MCP
**Best for:** headless browser automation, web scraping, PDF generation from HTML.

## Databases

### Supabase MCP — when `@supabase/supabase-js` in deps. Query tables, manage auth, storage.
### Convex MCP — when `convex` in deps / `convex/` dir / `convex.json`. Introspect live deployment. Run via `npx convex mcp start`.
### PostgreSQL MCP — raw Postgres, no ORM. Migrations, data analysis.
### Neon MCP — Neon serverless Postgres.
### Turso MCP — Turso/libSQL edge database.

## Version control & DevOps

### GitHub MCP — `.git` with GitHub remote; issue/PR workflows, Actions, releases. (Remote form shown above.)
### GitLab MCP — GitLab-hosted repos.
### Linear MCP — Linear issue refs like `ABC-123`; sprint planning.

## Cloud infrastructure

### AWS MCP — `@aws-sdk/*`, Terraform/CDK/SAM, Lambda, S3/DynamoDB.
### Cloudflare MCP — Workers/Pages/R2/D1.
### Vercel MCP — Vercel deployment & config.

## Monitoring & observability

### Sentry MCP — `@sentry/*` in deps; error investigation, release correlation.
### Datadog MCP — APM, logs, metrics.

## Communication

### Slack MCP — team notifications, deploy alerts, incident response.
### Notion MCP — docs/knowledge base, meeting notes.

## File & data

### Filesystem MCP — enhanced file ops beyond built-in tools (rarely needed; opencode has read/glob/grep/list).
### Memory MCP — persistent memory across sessions; long-running projects, user prefs.

## Containers & DevOps

### Docker MCP — `docker-compose.yml`/`Dockerfile`; container management.
### Kubernetes MCP — K8s manifests, Helm charts, cluster debugging.

## AI & ML

### Exa MCP — web search/research, competitive analysis, filling doc gaps.

## Quick reference: detection -> MCP

| Look for | Suggests |
|---|---|
| Popular npm packages | context7 |
| React/Vue/Next.js | Playwright MCP |
| `@supabase/supabase-js` | Supabase MCP |
| `convex` in deps / `convex/` / `convex.json` | Convex MCP |
| `pg` / `postgres` | PostgreSQL MCP |
| GitHub remote | GitHub MCP |
| `.linear` or `ABC-123` refs | Linear MCP |
| `@aws-sdk/*` | AWS MCP |
| `@sentry/*` | Sentry MCP |
| `docker-compose.yml` | Docker MCP |
| Slack webhook URLs | Slack MCP |
