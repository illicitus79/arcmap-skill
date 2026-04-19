# arcmap

**Turn any codebase into a fully interactive, self-contained architecture map — in one command.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Works with Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-6366f1)](https://claude.com/claude-code)
[![Works with Codex](https://img.shields.io/badge/OpenAI%20Codex-compatible-10a37f)](https://github.com/openai/codex)
[![Works with Copilot](https://img.shields.io/badge/GitHub%20Copilot-compatible-0ea5e9)](https://github.com/features/copilot)
[![Language-agnostic](https://img.shields.io/badge/languages-any-10b981)](#what-it-analyses)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-f97316)](#contributing)

> One command → one HTML file → open in any browser. **No server, no CDN, no dependencies.**
> Double-click and it works — even offline, even from a USB stick, even when your corp proxy blocks everything.

arcmap is an AI-agent skill (for Claude Code, OpenAI Codex, GitHub Copilot, Cursor, Aider,
or any compatible agent) that scans your workspace, understands it, and emits a single
self-contained HTML file you can share, ship as docs, attach to a PR, or paste into
onboarding material.

---

## Why arcmap?

Every team eventually says *"we need a diagram."* Then:
- someone draws one in draw.io → goes stale in two weeks
- someone generates a class-level UML → nobody reads it
- someone writes a wiki page → nobody finds it

arcmap is different because it's **re-generated on demand from the code itself**, interactive
enough to be useful, and a **single file** that lives next to your code.

| arcmap | vs. tool X |
|---|---|
| Self-contained `.html` · works from `file://` | Most tools need a server or an account |
| Any language, any framework, any monorepo | Most tools are per-stack |
| Written by an AI that read your code | Auto-generators miss semantic meaning |
| Editable output (it's just JSON + HTML) | Most SaaS output is locked |
| Free, MIT, no telemetry | Most alternatives are paid or track you |

---

## What it produces

A single `docs/project-map.html` file (~1–3 MB) with five interactive views plus global search:

| View | Description |
|---|---|
| **Architecture** | Layered card grid — one card per project, grouped by tier |
| **Infra Diagram** | SVG with RabbitMQ, Postgres, Prometheus, Stripe… edges labeled by protocol |
| **Graph** | Force-layout graph — pan/zoom, click to highlight one project's blast radius |
| **Data Flows** | Step-by-step cards showing protocol + purpose for each end-to-end path |
| **Endpoints** | Searchable catalog of every HTTP route in the repo with method, path, source |

Plus everywhere:
- 🔎 **Full-text search** across files, classes, methods, descriptions
- 🌓 **Light / dark theme** toggle (persisted)
- 🔗 **Deep links** — every view + project has a shareable URL
- 🎯 **Click-to-highlight** — dim unrelated nodes to trace a dependency path
- 🎛️ **Filter connections** by type (HTTP, DI, AMQP, gRPC, GraphQL, tRPC, …)
- ⬇️ **Export** — PNG, SVG, JSON, **Mermaid** (paste into any README), Markdown summary, Print/PDF
- ⌨️ **Keyboard shortcuts** — `?` shows the full list

---

## How to use

arcmap ships as a **bundle of three files** that must live together:

| File | Purpose |
|---|---|
| [SKILL.md](SKILL.md) | Instructions the agent follows when `/arcmap` is invoked |
| [template/map.html](template/map.html) | Self-contained HTML template with a `{{DATA_JSON}}` placeholder |
| [schema/data.schema.json](schema/data.schema.json) | JSON Schema used to validate the generated data |

Every install below clones all three into a single directory so the agent can find the
template alongside the instructions. `npx degit` downloads a clean copy without the
`.git` folder — it's a one-liner replacement for `git clone`.

---

### With Claude Code (recommended)

Claude Code's skills system picks up any `~/.claude/skills/<name>/SKILL.md` and bundles
sibling files automatically.

```bash
npx degit illicitus79/arcmap-skill ~/.claude/skills/arcmap
```

Then in any workspace:

```
/arcmap
```

To install per-project instead: `npx degit illicitus79/arcmap-skill .claude/skills/arcmap`.

---

### With OpenAI Codex (CLI)

Codex CLI's custom prompts (`~/.codex/prompts/*.md`) are slash commands — but they're
pure text, with no colocated-resource convention. The install therefore **downloads the
bundle separately** and **creates a small prompt that points Codex at it**:

```bash
# 1. Download the bundle (SKILL.md + template/ + schema/) to a stable location
npx degit illicitus79/arcmap-skill ~/.codex/skills/arcmap

# 2. Register the /arcmap slash command as a wrapper that tells Codex where to look
mkdir -p ~/.codex/prompts
cat > ~/.codex/prompts/arcmap.md <<'EOF'
Run the arcmap architecture-map generator.

1. Read the full skill instructions at ~/.codex/skills/arcmap/SKILL.md and follow every step.
2. When generating the HTML, use the template at ~/.codex/skills/arcmap/template/map.html
   (replace the single `{{DATA_JSON}}` placeholder — do not regenerate the template).
3. Validate the DATA object against ~/.codex/skills/arcmap/schema/data.schema.json before injecting.
4. Write the result to ./docs/project-map.html in the current workspace.
EOF
```

Then in any workspace:

```
codex
> /arcmap
```

**Windows (PowerShell):**

```powershell
npx degit illicitus79/arcmap-skill $HOME\.codex\skills\arcmap
New-Item -ItemType Directory -Force -Path $HOME\.codex\prompts | Out-Null
@'
Run the arcmap architecture-map generator.
1. Read ~/.codex/skills/arcmap/SKILL.md and follow every step.
2. Use the template at ~/.codex/skills/arcmap/template/map.html (replace {{DATA_JSON}}).
3. Validate against ~/.codex/skills/arcmap/schema/data.schema.json.
4. Write docs/project-map.html in the current workspace.
'@ | Out-File -Encoding utf8 $HOME\.codex\prompts\arcmap.md
```

**One-shot, no install:** `codex exec "Clone https://github.com/illicitus79/arcmap-skill into /tmp/arcmap then follow /tmp/arcmap/SKILL.md against the current workspace."`

---

### With GitHub Copilot (VS Code)

Copilot Chat reads reusable prompts from `.github/prompts/*.prompt.md` in the workspace.
Install arcmap **per-project** by vendoring the bundle into `.arcmap/` and registering
a prompt file:

```bash
# 1. Vendor the bundle into the repo (SKILL.md + template/ + schema/)
npx degit illicitus79/arcmap-skill .arcmap

# 2. Register /arcmap as a Copilot prompt
mkdir -p .github/prompts
cat > .github/prompts/arcmap.prompt.md <<'EOF'
---
mode: agent
description: Generate an interactive architecture map for this workspace
---
Read `.arcmap/SKILL.md` and follow every step.
Use the template at `.arcmap/template/map.html` (replace the `{{DATA_JSON}}` placeholder).
Validate the DATA object against `.arcmap/schema/data.schema.json`.
Write the result to `docs/project-map.html`.
EOF

# 3. (Optional) ignore the vendored skill from commits
echo ".arcmap/" >> .gitignore
```

Then in Copilot Chat:

```
/arcmap
```

> **Note:** `.github/prompts/` requires VS Code with the Copilot prompt-files feature
> enabled. If your version doesn't support it, you can instead tell Copilot in chat:
> *"Follow the instructions in `.arcmap/SKILL.md` to generate an architecture map."*

---

### With Cursor, Windsurf, Aider, or any agent that understands skills

```bash
# Cursor — project-level rules
npx degit illicitus79/arcmap-skill .cursor/rules/arcmap

# Windsurf / Aider / generic — clone anywhere and point the agent at SKILL.md
npx degit illicitus79/arcmap-skill .arcmap
```

The skill is self-describing — any agent with filesystem tools can read `SKILL.md`,
`template/map.html`, and `schema/data.schema.json` and execute the steps.

### Manual (no agent)

```bash
npx degit illicitus79/arcmap-skill ./arcmap
# Open arcmap/template/map.html, replace {{DATA_JSON}} with a DATA object matching arcmap/schema/data.schema.json
```

---

## Options

```
/arcmap                              # map current workspace → docs/project-map.html
/arcmap --out <path>                 # custom output path
/arcmap --depth shallow              # skip method-level analysis (faster, huge repos)
/arcmap --depth deep                 # include every method signature
/arcmap --no-infra                   # skip infra detection
/arcmap --include "glob" --exclude "glob"   # override scan paths
/arcmap --max-files 120              # cap files per project
/arcmap --focus <project-id>         # deep-scan one project, summary for rest
```

---

## What it analyses

arcmap is **language-agnostic** — if your code has a `package.json`, `Cargo.toml`,
`pyproject.toml`, `.csproj`, `go.mod`, `pom.xml`, `build.gradle`, `mix.exs`, `Gemfile`,
`composer.json`, `Package.swift`, or similar, arcmap understands it.

| Language / Framework | What's extracted |
|---|---|
| **C# / .NET** (ASP.NET, Minimal API, Worker) | Projects, interfaces, services, controllers, DI, EF Core entities |
| **TypeScript / Node** (Next, Nest, Express, Fastify, Hono, tRPC) | Routes, services, components, stores, Prisma schemas |
| **Python** (FastAPI, Django, Flask, Celery) | Routes, views, models, tasks, Pydantic contracts |
| **Dart / Flutter** | Widgets, providers, bloc/cubit, repositories |
| **Java / Kotlin** (Spring, Micronaut, Quarkus, Ktor, Android) | Controllers, services, repositories, entities |
| **Go** (Gin, Echo, Chi, net/http) | Handlers, structs, interfaces, proto |
| **Rust** (Axum, Actix, Rocket, Warp) | Routes, structs, traits, SQLx |
| **Ruby** (Rails, Sinatra, Sidekiq) | Controllers, models, services, jobs |
| **PHP** (Laravel, Symfony) | Controllers, models, services, Eloquent |
| **Elixir** (Phoenix, LiveView, Oban) | Routers, views, contexts, GenServers |
| **Swift** (Vapor, SwiftUI) | Routes, views, Core Data entities |
| **Scala** (Play, Akka, Http4s, ZIO) | Controllers, case classes, traits |
| **Haskell** (Servant, Yesod, Scotty) | Handlers, data types |

**Monorepo tooling detected:** Nx · Turborepo · pnpm/Yarn/npm workspaces · Lerna · Bazel ·
Cargo workspaces · Go workspaces · .NET solutions · Gradle multi-project · Maven multi-module.

**Infra components detected:** RabbitMQ · Kafka · Redpanda · Pulsar · NATS · SQS/SNS ·
Pub/Sub · Azure Service Bus · Temporal · PostgreSQL · MySQL · SQLite · SQL Server · MongoDB ·
DynamoDB · Cosmos · Firestore · Supabase · PlanetScale · ClickHouse · Cassandra · Neo4j ·
Redis · Memcached · Elasticsearch · OpenSearch · Meilisearch · Typesense · Algolia ·
Prometheus · Grafana · Loki · Tempo · Jaeger · OpenTelemetry · Sentry · Datadog · New Relic ·
Honeycomb · S3 · Azure Blob · GCS · MinIO · R2 · JWT · OAuth · Keycloak · Auth0 · Clerk ·
NextAuth · Nginx · Traefik · Envoy · Kong · Ocelot · YARP · Docker · Kubernetes · Helm ·
Terraform · Pulumi · Serverless · SST · Dapr · OpenAI · Anthropic · Azure OpenAI · Vertex AI ·
Ollama · LangChain · Pinecone · Weaviate · Qdrant · Chroma · Stripe · PayPal · Twilio ·
SendGrid · Slack · SignalR · Socket.IO · gRPC · Cloudflare Workers · Vercel · Netlify.

---

## Example output

After running `/arcmap` on a mixed .NET + Next.js + Flutter repo:

```
✓ arcmap complete
  Output:    docs/project-map.html
  Projects:  11   ·  Files: 84   ·  Classes: 210
  Connections: 12  ·  Infra: 9   ·  Data Flows: 14  ·  Endpoints: 47
  Monorepo:  pnpm
  Stack:     Next.js, .NET 9, Flutter
  Open: file:///your/workspace/docs/project-map.html
```

The HTML is 100% self-contained. Zip it, attach to an email, open without internet.

---

## CI integration

Regenerate the map on every main-branch merge so `docs/project-map.html` stays fresh:

```yaml
# .github/workflows/arcmap.yml
name: Update architecture map
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  regenerate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          prompt: "/arcmap"
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
      - name: Commit if changed
        run: |
          git config user.name "arcmap-bot"
          git config user.email "bot@users.noreply.github.com"
          git add docs/project-map.html
          git diff --staged --quiet || git commit -m "docs: refresh architecture map"
          git push
```

Then publish via GitHub Pages: `Settings → Pages → Branch: main → /docs`. Your team gets
a living architecture map at `https://<org>.github.io/<repo>/project-map.html`.

---

## Project structure

```
arcmap-skill/
  SKILL.md                   # Agent instructions (what "/arcmap" means)
  README.md                  # This file
  LICENSE                    # MIT
  CHANGELOG.md               # Release notes
  template/
    map.html                 # Self-contained HTML template — {{DATA_JSON}} placeholder
  schema/
    data.schema.json         # JSON Schema for the DATA object (validate before injecting)
```

---

## Customisation

### Modify the template

The template at [template/map.html](template/map.html) is one standalone file with a single
`{{DATA_JSON}}` placeholder. Fork and edit freely. CSS tokens are at the top of `<style>`;
the five renderers (`renderArchitecture`, `renderInfraDiagram`, `renderGraph`, `renderFlows`,
`renderEndpoints`) are cleanly separated in the `<script>` block.

Both light and dark themes ship out of the box. The `data-theme` attribute on `<html>`
flips the whole palette.

### Extend the schema

All schema fields beyond the required core (`meta.project`, `projects[].id`, connections
with valid endpoints) are optional. The template tolerates missing data gracefully.

Want a custom category (`"Saga"`, `"Policy"`, `"Feature Flag"`)? Just use the string —
the renderer accepts any category and falls back to neutral styling for unknown ones.

Want a custom connection type (`"kafka-stream"`, `"soap"`, `"mqtt"`)? Same — neutral grey
edge, still labeled, still filterable.

---

## Why no `fetch()`?

Browsers block `fetch()` under the `file://` protocol for security (CORS). arcmap embeds
all data as an inline JavaScript constant so the file works when double-clicked from the
filesystem — no web server, no `--allow-file-access-from-files` workarounds.

---

## Contributing

Issues and PRs welcome. The highest-value contributions are:

1. **Language/framework detection rules** you'd like supported
2. **Infra component signatures** for tools arcmap doesn't detect yet
3. **Example outputs** from real repos (anonymised) — they help others evaluate the tool
4. **Template polish** — accessibility, print styles, ergonomics

Before a PR: run `/arcmap` on two or three real repos and make sure the output still opens
cleanly in Chrome, Firefox, and Safari.

---

## License

MIT — see [LICENSE](LICENSE). Use it, fork it, ship it.

---

## Credits

Built to help engineers **trace logic through unfamiliar code faster**. If arcmap saved
you an hour, a ⭐ on GitHub is the best thank-you.
