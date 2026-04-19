---
name: arcmap
description: >
  Analyze any codebase (any language, any size) and produce a single self-contained HTML
  architecture map with five interactive views: Architecture cards, Infra Diagram,
  Dependency Graph, Data Flows, Endpoint catalog — plus full-text search, pan/zoom,
  path highlighting, theme toggle, and PNG/SVG/Mermaid/Markdown export. Zero server,
  zero dependencies, works from file://.
trigger: /arcmap
---

# /arcmap — Architecture Map Generator

Turn any codebase into a fully interactive, self-contained HTML architecture map.
**One command → one file → open in any browser, no server needed.**

Built to be **language-agnostic**: works for .NET, TypeScript/Node, Python, Dart/Flutter,
Rust, Go, Java/Kotlin, Ruby, PHP, Elixir, Swift, Scala — in single repos or monorepos
(Nx, Turborepo, pnpm/yarn workspaces, Lerna, Bazel, Cargo workspaces, Go workspaces).

## Usage

```
/arcmap                              # map current workspace to ./docs/project-map.html
/arcmap --out <path>                 # custom output path
/arcmap --depth shallow              # skip method-level analysis (faster, large repos)
/arcmap --depth medium               # default — classes + key methods
/arcmap --depth deep                 # include every method signature
/arcmap --infra / --no-infra         # toggle infra detection (default: on)
/arcmap --include "glob" --exclude "glob"   # override scan paths
/arcmap --max-files <N>              # cap files per project (default: 80)
/arcmap --focus <project-id>         # only deep-scan one project (others summary-only)
```

---

## What You Must Do When Invoked

Follow every step in order. Do not skip or abbreviate.

---

### STEP 1 — Understand the Workspace

**1a. Detect monorepo tooling first.** If present, use it to enumerate projects:

| Tool | Detection marker | How to list projects |
|---|---|---|
| **Nx** | `nx.json` | Read `workspace.json` / scan `apps/` and `libs/` |
| **Turborepo** | `turbo.json` | Scan workspaces in root `package.json` |
| **pnpm workspaces** | `pnpm-workspace.yaml` | Read `packages:` list |
| **Yarn / npm workspaces** | `"workspaces"` key in root `package.json` | Enumerate glob |
| **Lerna** | `lerna.json` | Read `packages` array |
| **Bazel** | `WORKSPACE` / `MODULE.bazel` | Find `BUILD` files |
| **Cargo workspaces** | `[workspace]` in `Cargo.toml` | Read `members` |
| **Go workspaces** | `go.work` | Read `use (...)` |
| **.NET solution** | `*.sln` / `*.slnx` | Parse project references |
| **Gradle multi-project** | `settings.gradle[.kts]` | Read `include(...)` |
| **Maven multi-module** | Root `pom.xml` with `<modules>` | Parse modules |

Record the tool in `meta.monorepo` — it helps users recognise their own setup.

**1b. Enumerate project folders.** Look for any of:
- `*.csproj` · `*.fsproj` · `*.vbproj` · `*.sln`
- `package.json` · `deno.json` · `bun.lock`
- `pubspec.yaml` · `pyproject.toml` · `requirements.txt` · `setup.py` · `Pipfile`
- `Cargo.toml` · `go.mod` · `pom.xml` · `build.gradle[.kts]` · `build.sbt`
- `Gemfile` · `composer.json` · `mix.exs` · `Package.swift` · `stack.yaml` · `cabal.project`

**1c. Classify each project** by primary language/framework (see Language Guide below).

**1d. Identify shared/library projects** (named `*.Core`, `*.Domain`, `*.Shared`, `*.Common`, `libs/*`, `packages/*-shared`, etc.).

**1e. Identify test projects** — flag as `type: "test"` or exclude via `--exclude`.

**1f. Catalog infrastructure config files:**
- `docker-compose*.y{a,}ml`, `Dockerfile*`
- `appsettings*.json`, `*.env*`, `application.y{a,}ml`, `config/*.yml`
- `k8s/`, `helm/`, `*.tf`, `serverless.yml`, `cdk.json`, `sst.config.ts`
- `prometheus.yml`, `grafana/`, `otel-collector-config.yaml`
- `nginx.conf`, `traefik.yml`, `envoy.yaml`
- `.github/workflows/*.yml`, `.gitlab-ci.yml`, `azure-pipelines.yml`

Produce a **project inventory** — the internal list you'll use in Step 2.

---

### STEP 2 — Deep-Analyse Each Project

For every non-test project, extract:

| Field | Notes |
|---|---|
| `id` | Stable identifier — folder / package / namespace name |
| `name` | Full project name |
| `shortName` | 2–3 word display name |
| `type` | `api` · `library` · `worker` · `frontend` · `mobile` · `desktop` · `cli` · `gateway` · `bff` · `scheduler` · `test` · `infra` · `data` · `docs` |
| `technology` | Framework + runtime (e.g. `C# / .NET 9`, `Next.js 15 / TS`, `Django 5 / Python 3.12`) |
| `language` | Primary language token |
| `framework` | Primary framework |
| `port` | Listening port if detectable |
| `database` | Primary data store |
| `color` | Unique from palette below |
| `icon` | `cube` `cog` `server` `refresh` `database` `globe` `clock` `mobile` `desktop` `browser` |
| `description` | 2–3 sentence summary of the project's purpose |
| `keyPatterns` | Up to 5 notable design patterns (e.g. `CQRS`, `Outbox`, `Clean Architecture`) |
| `entrypoints` | Known entry files (`Program.cs`, `main.go`, `index.ts`, `manage.py`) |
| `files` | Array of key files (schema below) |

**Color palette** (assign one per project, no repeats):
`#6366f1` `#10b981` `#06b6d4` `#f59e0b` `#ef4444` `#8b5cf6` `#ec4899` `#f97316` `#14b8a6` `#a855f7` `#84cc16` `#0ea5e9` `#22c55e` `#f43f5e` `#eab308`

#### File Schema

Skip generated, vendor, build output: `obj/`, `bin/`, `node_modules/`, `.dart_tool/`, `target/`, `dist/`, `build/`, `.next/`, `__pycache__/`, `vendor/`, `.venv/`, `Pods/`.

```jsonc
{
  "path": "relative/path/to/File.ext",
  "category": "Model|Service|Interface|Endpoint|Worker|Contract|Data|Security|Middleware|Configuration|Seeding|Handler|Messaging|Test|Component|View|Store|Resolver|Schema|Migration|Job",
  "description": "One sentence — what this file does and why it exists.",
  "classes": [
    { "name": "Name", "type": "class|interface|enum|record|trait|struct|…", "description": "One-line purpose." }
  ],
  "methods": [
    { "name": "Method", "signature": "Ret Method(Param p, …)", "description": "One line." }
  ],
  "endpoints": [
    { "method": "GET", "path": "/api/orders/{id}", "description": "…", "auth": "JWT" }
  ],
  "dependencies": ["Other.cs", "IService"],
  "usedBy": ["Caller.cs"]
}
```

Populate `endpoints` whenever the file is `category: "Endpoint"` — the **Endpoints** view
catalogues them across the whole repo. If you can't extract explicit routes, leave the
array empty; the template will fall back to method names.

**Category guide** (core set; arbitrary strings allowed for domain-specific taxonomies):

| Category | Typical matches |
|---|---|
| `Model` | Entity, DTO, value object, enum, Prisma/EF/SQLAlchemy/Ecto schema |
| `Service` | Business logic, use-case, interactor |
| `Interface` | Service contract, port, abstract class |
| `Endpoint` | HTTP route handler, controller, GraphQL resolver, tRPC procedure |
| `Worker` | Background service, hosted service, scheduler, cron, Sidekiq job |
| `Contract` | Message / event / API DTO shared between services |
| `Data` | DbContext, repository, data mapper, migration runner |
| `Security` | Auth, JWT, policy, guard, OAuth, CORS, CSRF |
| `Middleware` | Pipeline middleware, filter, interceptor |
| `Configuration` | Startup, DI registration, options binding, env loader |
| `Seeding` | Seed data, fixtures |
| `Handler` | Command / query / event handler (CQRS, MediatR) |
| `Messaging` | Publisher, consumer, queue client, broker wrapper |
| `Test` | Unit / integration / e2e test |
| `Component` | UI component (React, Vue, Svelte, SwiftUI, Compose) |
| `View` | Page / screen / route component |
| `Store` | Redux/Zustand/Pinia/Vuex/Riverpod store |
| `Resolver` | GraphQL resolver |
| `Schema` | GraphQL/JSON/Avro/Proto schema |
| `Migration` | Database migration |
| `Job` | Scheduled or one-off job |

---

### STEP 3 — Detect Infrastructure Components

Scan config + source for external components. Build the `infraComponents` array:

```jsonc
{
  "id": "rabbitmq",
  "label": "RabbitMQ",
  "icon": "🐇",
  "subtitle": "AMQP · :5672",
  "category": "messaging",
  "color": "#f97316",
  "notes": "Used by OrderService for async event propagation."
}
```

**Detection patterns (extended):**

#### Messaging & Streaming
| Component | Look for |
|---|---|
| **RabbitMQ** | `RabbitMQ.Client`, `amqplib`, `pika`, `bunny`, `MassTransit`, `amqp://` |
| **Kafka** | `Confluent.Kafka`, `kafkajs`, `confluent-kafka`, `KAFKA_BOOTSTRAP`, `sarama` |
| **Redpanda** | `redpanda`, `rpk` |
| **Pulsar** | `pulsar-client`, `pulsar://` |
| **NATS** | `nats.js`, `NATS.Client`, `nats://` |
| **AWS SQS / SNS / EventBridge** | `@aws-sdk/client-sqs`, `@aws-sdk/client-sns`, `aws-sdk`, `boto3` sqs/sns |
| **Google Pub/Sub** | `@google-cloud/pubsub`, `google.cloud.pubsub` |
| **Azure Service Bus** | `Azure.Messaging.ServiceBus` |
| **Temporal** | `@temporalio/*`, `Temporalio.*` |
| **Airflow** | `airflow`, `dags/` |

#### Databases
| Component | Look for |
|---|---|
| **PostgreSQL** | `Npgsql`, `pg`, `psycopg`, `postgres`, `ecto_sql` |
| **MySQL / MariaDB** | `MySqlConnector`, `Pomelo`, `mysql2`, `PyMySQL` |
| **SQLite** | `Microsoft.Data.Sqlite`, `better-sqlite3`, `sqlite3` |
| **SQL Server** | `Microsoft.Data.SqlClient`, `mssql`, `pyodbc` |
| **Oracle** | `Oracle.ManagedDataAccess`, `cx_Oracle` |
| **MongoDB** | `MongoDB.Driver`, `mongoose`, `pymongo`, `mongo://` |
| **Cassandra / ScyllaDB** | `CassandraCSharpDriver`, `cassandra-driver` |
| **DynamoDB** | `AWSSDK.DynamoDBv2`, `@aws-sdk/client-dynamodb`, `boto3` dynamodb |
| **Cosmos DB** | `Microsoft.Azure.Cosmos` |
| **Firestore** | `@google-cloud/firestore`, `firebase-admin` |
| **Supabase** | `@supabase/supabase-js`, `supabase` |
| **PlanetScale** | `@planetscale/database` |
| **ClickHouse** | `ClickHouse.Client`, `clickhouse-driver` |
| **Neo4j** | `Neo4j.Driver`, `neo4j-driver` |

#### Cache / KV
| Component | Look for |
|---|---|
| **Redis / Valkey** | `StackExchange.Redis`, `ioredis`, `redis-py`, `redis://` |
| **Memcached** | `EnyimMemcachedCore`, `memcached` |
| **Upstash** | `@upstash/redis` |

#### Search
| Component | Look for |
|---|---|
| **Elasticsearch / OpenSearch** | `Elastic.Clients`, `@elastic/elasticsearch`, `opensearchpy` |
| **Meilisearch** | `meilisearch` |
| **Typesense** | `typesense` |
| **Algolia** | `algoliasearch` |

#### Observability
| Component | Look for |
|---|---|
| **Prometheus** | `prometheus`, `AddPrometheusExporter`, `prom-client`, `prometheus.yml` |
| **Grafana** | `grafana/`, `GF_` env |
| **Loki** | `Serilog.Sinks.Grafana.Loki`, `loki` |
| **OpenTelemetry** | `OpenTelemetry.*`, `@opentelemetry/*`, `OTEL_` env, `otlp` |
| **Seq** | `Serilog.Sinks.Seq` |
| **Jaeger** | `jaeger`, `JAEGER_` |
| **Tempo** | `grafana/tempo` |
| **Sentry** | `@sentry/*`, `Sentry.*`, `sentry_sdk` |
| **Datadog** | `dd-trace`, `ddtrace`, `Datadog.Trace` |
| **New Relic** | `newrelic` |
| **Honeycomb** | `@honeycombio/*` |

#### Storage
| Component | Look for |
|---|---|
| **S3** | `AWSSDK.S3`, `@aws-sdk/client-s3`, `boto3` s3 |
| **Azure Blob** | `Azure.Storage.Blobs` |
| **GCS** | `@google-cloud/storage` |
| **MinIO** | `minio` |
| **Cloudflare R2** | `R2`, `@cloudflare/workers-types` |

#### Auth
| Component | Look for |
|---|---|
| **JWT** | `AddJwtBearer`, `jsonwebtoken`, `PyJWT`, `jose` |
| **OAuth / OIDC** | `AddOpenIdConnect`, `openid-client`, `authlib` |
| **API Key** | `X-Api-Key`, custom `ApiKey` middleware |
| **Keycloak** | `keycloak`, `KEYCLOAK_` |
| **Auth0** | `@auth0/*`, `auth0` |
| **Clerk** | `@clerk/*`, `clerk` |
| **Supabase Auth** | `@supabase/auth-js` |
| **NextAuth / Auth.js** | `next-auth`, `@auth/core` |

#### Gateway / Proxy
| Component | Look for |
|---|---|
| **Nginx** | `nginx.conf`, `/etc/nginx/` |
| **Traefik** | `traefik.yml`, `traefik.yaml` |
| **Envoy** | `envoy.yaml` |
| **Kong** | `kong.yml` |
| **Ocelot** | `Ocelot.*`, `ocelot.json` |
| **YARP** | `Yarp.ReverseProxy` |

#### Orchestration
| Component | Look for |
|---|---|
| **Docker** | `docker-compose*.yml`, `Dockerfile*` |
| **Kubernetes** | `k8s/`, `*.yaml` with `kind: Deployment` |
| **Helm** | `Chart.yaml`, `helm/` |
| **Terraform** | `*.tf`, `main.tf` |
| **Pulumi** | `Pulumi.yaml`, `@pulumi/*` |
| **Serverless / SST** | `serverless.yml`, `sst.config.ts`, `cdk.json` |
| **Dapr** | `Dapr.*`, `@dapr/*`, `dapr.yaml` |

#### LLM / AI
| Component | Look for |
|---|---|
| **OpenAI** | `openai`, `OpenAI.*`, `OPENAI_API_KEY` |
| **Anthropic** | `@anthropic-ai/sdk`, `anthropic`, `ANTHROPIC_API_KEY` |
| **Azure OpenAI** | `Azure.AI.OpenAI`, `AZURE_OPENAI_` |
| **Vertex AI** | `@google-cloud/aiplatform`, `google.cloud.aiplatform` |
| **Ollama** | `ollama` |
| **LangChain** | `langchain`, `@langchain/*` |
| **Pinecone / Weaviate / Qdrant / Chroma** | vector DB SDKs |

#### Payments / SaaS
| Component | Look for |
|---|---|
| **Stripe** | `stripe`, `Stripe.*`, `STRIPE_` |
| **PayPal** | `@paypal/*`, `paypalrestsdk` |
| **Twilio** | `twilio`, `Twilio.*` |
| **SendGrid** | `@sendgrid/*`, `SendGrid.*` |
| **Mailgun / Resend / Postmark** | respective SDK names |
| **Slack** | `@slack/*`, `slack_sdk` |

#### Real-time
| Component | Look for |
|---|---|
| **SignalR** | `AddSignalR`, `IHubContext`, `@microsoft/signalr` |
| **Socket.IO** | `socket.io`, `@socket.io/*` |
| **Ably / Pusher / Centrifugo** | respective SDK names |
| **gRPC** | `.proto` files, `Grpc.AspNetCore`, `@grpc/grpc-js`, `grpcio` |

#### CDN / Edge
| Component | Look for |
|---|---|
| **Cloudflare Workers** | `wrangler.toml`, `@cloudflare/workers-types` |
| **Vercel** | `vercel.json`, `.vercel/` |
| **Netlify** | `netlify.toml` |
| **Fastly** | `fastly.toml` |

When in doubt, err on the side of **including** a detected component with a short note —
users can edit the output if needed.

---

### STEP 4 — Identify Connections

Produce the `connections` array (edges between projects, or project→infra):

```jsonc
{
  "from": "ProjectA.Id",
  "to":   "ProjectB.Id",
  "type": "http|di|reference|amqp|grpc|websocket|file|graphql|trpc|sse|shared-db|env|webhook",
  "label": "REST API",
  "description": "One sentence describing what flows over this connection."
}
```

**Detection rules:**

| Type | How to detect |
|---|---|
| `http` | `HttpClient` / typed client / `fetch` / `axios` / `ky` targeting another project's port/URL |
| `di` | Interface defined in project B, registered/injected in project A (same solution) |
| `reference` | `<ProjectReference>`, `workspace:*`, `path:../*` in `package.json`, `replace` in `go.mod`, `Cargo.toml` `path` |
| `amqp` | Publish/consume against RabbitMQ/Kafka/NATS queues |
| `grpc` | `.proto` stubs imported across projects |
| `websocket` | SignalR hub, Socket.IO, raw `WebSocket` client/server |
| `file` | Import worker watches folder written by another component |
| `graphql` | Client queries targeting another project's GraphQL schema |
| `trpc` | tRPC router imported as type across client/server |
| `sse` | `EventSource` / `text/event-stream` from another service |
| `shared-db` | Two projects read/write the same database instance |
| `env` | Project A reads an env var set by project B's infra (weak coupling — flag to inform users) |
| `webhook` | Project A posts to an endpoint exposed by project B, usually via config URL |

Unrecognised types render in neutral grey — you can use custom labels freely.

---

### STEP 5 — Build Architecture Layers

Group projects into 2–5 layers:

```jsonc
{
  "name": "Layer Name",
  "description": "What this layer is responsible for.",
  "projects": ["ProjectA.Id", "ProjectB.Id"]
}
```

Common patterns — pick what fits the system:
- **Frontend Clients** — web, mobile, desktop UIs
- **Gateway / BFF** — aggregator APIs, reverse proxies
- **Service Layer** — core business APIs
- **Worker Layer** — background/async workers, schedulers
- **Data Layer** — databases, caches
- **Messaging / Infra** — queues, brokers, streams
- **Shared Domain** — shared libraries, contracts, protobuf packages
- **External / SaaS** — third-party integrations

Every project must appear in exactly one layer.

---

### STEP 6 — Build Data Flows

Produce a `dataFlows` array tracing **end-to-end** paths:

```jsonc
{
  "from": "ProjectA.Id or 'External Source'",
  "to":   "ProjectB.Id or 'External Sink'",
  "protocol": "HTTP REST|AMQP|gRPC|GraphQL|tRPC|WebSocket|SSE|File Import|Direct DI|Webhook",
  "description": "1–2 sentences: what data moves and why."
}
```

Aim for **8–15 flows** covering the most important operations (create/read lifecycles,
event propagation, scheduled jobs, sync paths, auth).

---

### STEP 7 — (Optional) Env-var audit & Endpoint catalog

If time permits and the repo has clear env usage:

**`envVars`** — scan for `process.env.X`, `Environment.GetEnvironmentVariable("X")`,
`os.getenv("X")`, `std::env::var("X")`, `System.getenv("X")`, ENV files. Record:

```jsonc
{ "name": "DATABASE_URL", "usedBy": ["api", "worker"], "description": "Primary DB", "secret": true }
```

**Endpoints per file** — when extracting `Endpoint` category files, populate the
`endpoints` array on each file (see File Schema) so the HTML Endpoint view works fully.

---

### STEP 8 — Assemble the DATA JSON

```jsonc
{
  "meta": {
    "project": "YourProjectName",
    "generated": "YYYY-MM-DD",
    "description": "One sentence.",
    "repository": "https://github.com/org/repo",
    "monorepo": "pnpm",
    "primaryStack": ["Next.js", ".NET 9", "Flutter"],
    "totalFiles": 0,
    "totalProjects": 0
  },
  "architecture": {
    "summary": "3–5 sentence plain-English description of the overall system.",
    "layers": [ /* … */ ],
    "dataFlows": [ /* … */ ]
  },
  "projects":        [ /* … */ ],
  "connections":     [ /* … */ ],
  "infraComponents": [ /* … */ ],
  "envVars":         [ /* optional */ ]
}
```

---

### STEP 9 — Generate the HTML File

1. Read `template/map.html` from this skill's folder.
2. Replace the single placeholder `{{DATA_JSON}}` with the serialised JSON.
3. Write the result to `<workspace-root>/docs/project-map.html` (or `--out` path).

**Safety — always escape `</script>` sequences** in string values before injection, e.g. by
using `JSON.stringify(data).replace(/<\/(script)/gi, '<\\/$1')`. The renderer HTML-escapes
field values, but inline `</script>` in raw JSON would still break parsing.

If the template file is not available (skill installed without template), reproduce it
from the **HTML Template Specification** below.

---

## HTML Template Specification

The template is a single self-contained file. When you need to reproduce it, it must
include:

### Required Views (tabs)

| Tab | ID | Renders |
|---|---|---|
| Architecture | `architecture` | Layered cards grid — one card per project |
| Infra Diagram | `diagram` | SVG with app + infra zones, labeled edges, pan/zoom |
| Graph | `graph` | Layered SVG graph — project nodes, typed edges, pan/zoom, highlight-on-click |
| Data Flows | `flows` | Vertical flow cards (clickable source/sink chips) |
| Endpoints | `endpoints` | Sortable/filterable table of all HTTP routes from `Endpoint` files |

### Required UI Elements

- **Header**: logo, title, view toggles, search, theme toggle, export button, help button
- **Stats bar**: Projects · Files · Classes · Connections · Data Flows · Infra · Endpoints
- **Filter bar** (Graph/Diagram only): toggle connection types (HTTP, DI, AMQP, …)
- **Sidebar**: project list + connection list; click to open detail panel
- **Detail panel**: slide-in right — description, patterns, connections, files accordion, share buttons
- **Modal**: export menu + keyboard-shortcuts help
- **Toast**: transient status (e.g. "Copied to clipboard")

### Required Capabilities

- **HTML escape** every value from `DATA` before `innerHTML` (XSS-safe)
- **Pan / zoom** on Graph and Infra Diagram SVGs (drag = pan, wheel = zoom, buttons for reset)
- **Highlight on click** — clicking a project node dims unrelated nodes and edges
- **Deep-linkable URL state** — view, selected project, and filters survive in `#view=…&project=…&filter=…`
- **Light / dark theme toggle** — persisted in `localStorage`
- **Export** — PNG, SVG, JSON, Mermaid, Markdown (all client-side)
- **Print-friendly** `@media print` styles that hide chrome and preserve content
- **Keyboard**: `1–5` switch views, `Ctrl/Cmd+K` focuses search, `T` theme, `E` export, `?` help, `Esc` closes panels

### Design Tokens

```css
:root, html[data-theme="dark"] {
  --bg:#0f1117; --surface:#1a1d27; --surface2:#232736;
  --border:#2d3148; --text:#e2e4ed; --text-dim:#8b8fa3;
  --accent:#6366f1; --green:#10b981; --amber:#f59e0b;
  --purple:#8b5cf6; --red:#ef4444; --orange:#f97316; --cyan:#06b6d4;
  --radius:12px; --transition:.2s cubic-bezier(.4,0,.2,1);
}
html[data-theme="light"] { /* mirror with light values */ }
```

Font stack: `'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif`
Monospace: `'Cascadia Code', 'Fira Code', monospace`

### Data Injection Pattern

Output must be **fully self-contained** — no CDN, no `fetch()`, no external image URLs.

```html
<script>
const DATA = { /* full JSON here */ };
</script>
```

Browsers block `fetch()` under `file://` (CORS) — inlining the data makes the file
double-clickable from disk.

---

## Language Guide

### .NET / C#
- `.csproj` `<ProjectReference>` → `reference` connections
- `Program.cs` / `Startup.cs` → DI registrations, port, middleware
- `appsettings.json` → connection strings → infra
- `IHostedService` / `BackgroundService` → `Worker`
- Minimal API groups / `[Controller]` / `[ApiController]` → `Endpoint`
- `DbContext` / EF Core → `Data`

**CRITICAL — Endpoint file completeness rule:**
Endpoint files (`*Endpoints.cs`, `*Controller.cs`) are often very long (200–1000+ lines) with dozens of routes defined inline.
You MUST read every endpoint file **in full** — do not stop at the first page.
For each `MapGet` / `MapPost` / `MapPut` / `MapPatch` / `MapDelete` call, extract the route string and HTTP method.
Add every route as a separate object in the file's `endpoints[]` array.
A file with only 1–3 endpoints listed when it defines 10+ is incomplete — re-read the file.
Common patterns that hide many routes: route groups via `MapGroup()`, chained `.MapXxx()` calls, separate sub-groups within the same method.

### TypeScript / JavaScript (Node / Deno / Bun)
- `package.json` deps → infer infra (pg, mongoose, redis, amqplib, @aws-sdk/*, …)
- Next.js App Router `app/**/route.ts` → `Endpoint`; `app/**/page.tsx` → `View`
- Next.js Pages Router `pages/api/**` → `Endpoint`
- NestJS `@Controller` → `Endpoint`; `@Injectable` → `Service`
- Express/Fastify/Koa/Hono route declarations → `Endpoint`
- tRPC `router`, `procedure` → `Endpoint` (type=trpc)
- GraphQL resolvers → `Resolver`
- React/Vue/Svelte components in `components/` → `Component`
- Zustand/Redux/Pinia stores → `Store`
- Prisma schema → `Schema`; migrations → `Migration`
- SvelteKit `+page.server.ts`, `+server.ts` → `Endpoint`
- Nuxt `server/api/**` → `Endpoint`
- Angular `@Component`, `@Injectable`, `@NgModule` → respective

### Python
- `requirements.txt` / `pyproject.toml` / `Pipfile` → infra
- `class` → classes array; module-level `def` in service files → methods
- FastAPI / Flask / Starlette route decorators → `Endpoint`
- Django `views.py`, `urls.py` → `Endpoint`; `models.py` → `Model`; `admin.py` → `Configuration`
- SQLAlchemy / Tortoise / Django ORM models → `Model`
- Celery tasks, APScheduler jobs → `Worker`/`Job`
- Pydantic models → `Contract`

### Dart / Flutter
- `pubspec.yaml` → infra (http, dio, sqflite, hive, isar, riverpod, bloc)
- `Widget` subclasses → `Component`
- Provider / Riverpod / Bloc / GetX → `Store`/`Service`
- Repository pattern → `Service` or `Data`

### Java / Kotlin
- **Spring**: `@RestController` → `Endpoint`; `@Service`/`@Component` → `Service`; `@Repository` → `Data`; `@Entity` → `Model`
- **Spring Cloud / Feign** → `http` connections
- **Micronaut / Quarkus**: similar annotation-based mapping
- **Ktor**: `routing { route(…) }` → `Endpoint`
- **Android**: `Activity`/`Fragment`/`Composable` → `View`/`Component`; Room DAO → `Data`

### Go
- `http.HandleFunc`, `gin.Engine`, `echo.Echo`, `chi.Router`, `gorilla/mux` routes → `Endpoint`
- `struct` in `models/`, `domain/`, `entity/` → `Model`
- `interface` in `ports/`, `contracts/` → `Interface`
- gRPC generated code → `Contract`

### Rust
- `Cargo.toml` `[dependencies]` → infra
- `main.rs`, `lib.rs`, `mod.rs` → entrypoints
- Axum / Actix / Rocket / Warp route functions → `Endpoint`
- `struct` / `enum` → classes; `trait` → `Interface` (badge: `trait`)
- SQLx / Diesel / SeaORM → `Data`

### Ruby
- `Gemfile` → infra
- **Rails**: `app/controllers/**` → `Endpoint`; `app/models/**` → `Model`; `app/services/**` → `Service`; `app/jobs/**` → `Worker`; migrations → `Migration`
- **Sinatra / Hanami / Grape**: route blocks → `Endpoint`
- Sidekiq workers → `Worker`

### PHP
- `composer.json` → infra
- **Laravel**: `routes/**`, `app/Http/Controllers/**` → `Endpoint`; `app/Models/**` → `Model`; `app/Services/**` → `Service`; `app/Jobs/**` → `Worker`; Eloquent migrations → `Migration`
- **Symfony**: `#[Route]` controllers → `Endpoint`; `App\Entity` → `Model`
- **Slim / Lumen**: route decl → `Endpoint`

### Elixir
- `mix.exs` → infra
- **Phoenix**: `router.ex`, `*_controller.ex` → `Endpoint`; `*_view.ex` → `View`; `*_live.ex` → `View`/`Component`
- Ecto schemas → `Model`; Ecto migrations → `Migration`
- GenServer / Supervisor → `Worker`/`Service`
- Oban jobs → `Worker`

### Swift
- `Package.swift` → dependencies
- **Vapor**: `routes.swift`, `*Controller.swift` → `Endpoint`
- **SwiftUI**: `View` structs → `View`/`Component`
- Core Data entities → `Model`

### Scala
- `build.sbt` → infra
- **Play**: `conf/routes`, `controllers/**` → `Endpoint`
- **Akka HTTP**, **Http4s**, **ZIO HTTP**: route declarations → `Endpoint`
- Case classes → `Model`; traits → `Interface`

### Haskell
- `cabal.project` / `stack.yaml` → infra
- **Servant**, **Yesod**, **Scotty**: handlers → `Endpoint`
- `data` declarations → `Model`

---

## Quality Checklist

Before writing the output file, verify:

- [ ] Every detected project has an entry in `projects`
- [ ] Every detected infrastructure component is in `infraComponents`
- [ ] `architecture.layers` covers every project (no orphans)
- [ ] Every connection has valid `from` and `to` IDs (matching project or infra IDs)
- [ ] The HTML has **no** `fetch()` call and **no** external `<script src=…>` — data is inline
- [ ] `</script>` sequences inside string values are escaped before injection
- [ ] Output is a single `.html` file that opens directly from the filesystem in Chrome/Edge/Firefox/Safari
- [ ] Search works across all text
- [ ] File size is reasonable (< 3 MB for most projects)

---

## Output

Write the final file to: `<workspace-root>/docs/project-map.html`

Print a summary like:

```
✓ arcmap complete
  Output:    docs/project-map.html
  Projects:  11   ·  Files: 84   ·  Classes: 210
  Connections: 12  ·  Infra: 9   ·  Data Flows: 14  ·  Endpoints: 47
  Monorepo:  pnpm
  Stack:     Next.js, .NET 9, Flutter
  Open: file:///path/to/docs/project-map.html
```
