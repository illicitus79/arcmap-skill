---
name: arcmap
description: >
  Analyze any codebase and produce a single self-contained HTML architecture map
  with five interactive views: Architecture cards, Infra Diagram, Dependency Graph,
  Data Flows, and full-text search — zero server, zero dependencies, works from file://.
trigger: /arcmap
---

# /arcmap  —  Architecture Map Generator

Turn any codebase into a fully interactive, self-contained HTML architecture map.
One command → one file → open in any browser, no server needed.

## Usage

```
/arcmap                          # map current workspace to ./docs/project-map.html
/arcmap --out <path>             # custom output path
/arcmap --depth shallow          # skip method-level analysis (faster, large repos)
/arcmap --depth deep             # include every method signature (default: medium)
/arcmap --infra                  # include all detected infra components (default: on)
/arcmap --no-infra               # skip infra detection
```

---

## What You Must Do When Invoked

Follow every step in order. Do not skip or abbreviate.

---

### STEP 1 — Understand the Workspace

Scan the top-level folder. Identify:
- Every **project folder** (look for: `*.csproj`, `package.json`, `pubspec.yaml`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, `build.gradle`)
- The **primary language / framework** of each project (C#/.NET, Node/TypeScript, Python, Dart/Flutter, Rust, Go, Java, React, Next.js, Electron, etc.)
- Any **shared / domain library** projects (named `*.Core`, `*.Domain`, `*.Shared`, `*.Common`, `*.Api` with no controllers)
- Any **test** projects (skip from the map or flag as `type: "test"`)
- Any **infrastructure config** files: `docker-compose.yml`, `prometheus.yml`, `grafana/`, `k8s/`, `.env`, `appsettings*.json`, `rabbitmq.conf`

Produce a **project inventory** — an internal list you'll use in Step 2.

---

### STEP 2 — Deep-Analyse Each Project

For every non-test project in the inventory, read its source files and extract:

| Field | What to extract |
|---|---|
| `id` | Unique camelCase/PascalCase identifier (e.g. `MyApp.Api`) |
| `name` | Full project name |
| `shortName` | 2–3 word display name |
| `type` | `api` · `library` · `worker` · `frontend` · `mobile` · `desktop` · `cli` · `test` |
| `technology` | Framework + runtime (e.g. `C# / .NET 9`, `Next.js 15 / TypeScript`) |
| `port` | Listening port if detectable from config / startup code |
| `database` | Primary data store (e.g. `PostgreSQL`, `SQLite`, `MongoDB`) |
| `color` | Pick from palette below — unique per project |
| `icon` | One of: `cube` `cog` `server` `refresh` `database` `globe` `clock` `mobile` `desktop` `browser` |
| `description` | 2–3 sentence summary of the project's purpose and responsibility |
| `keyPatterns` | Up to 5 notable design patterns or technologies (e.g. `Outbox Pattern`, `CQRS`, `DI`) |
| `files` | Array of key files (see File Schema below) |

**Color palette** (assign one per project, no repeats):
`#6366f1` `#10b981` `#06b6d4` `#f59e0b` `#ef4444` `#8b5cf6` `#ec4899` `#f97316` `#14b8a6` `#a855f7` `#84cc16` `#0ea5e9`

#### File Schema

For each **meaningful** source file (skip generated, migrations, `obj/`, `bin/`, `node_modules/`, `.dart_tool/`):

```jsonc
{
  "path": "relative/path/to/File.cs",        // relative to project root
  "category": "Model|Service|Interface|Endpoint|Worker|Contract|Data|Security|Middleware|Configuration|Seeding|Handler|Messaging",
  "description": "One sentence — what this file does and why it exists.",
  "classes": [                                // top-level types in the file
    {
      "name": "ClassName",
      "type": "class|interface|enum|record|abstract class|sealed class|static class",
      "description": "One-line purpose."
    }
  ],
  "methods": [                                // only for Service / Endpoint / Handler files
    {
      "name": "MethodName",
      "signature": "ReturnType MethodName(ParamType param, ...)",
      "description": "One-line — what it does."
    }
  ],
  "dependencies": ["OtherFile.cs", "IService"],   // what this file directly uses
  "usedBy": ["CallerFile.cs", "Endpoint group"]   // what uses this file
}
```

**Category guide:**
- `Model` — entity / DTO / value object / enum
- `Service` — business logic implementation
- `Interface` — service contract / abstraction
- `Endpoint` — HTTP route handler / controller
- `Worker` — background service / hosted service / timer
- `Contract` — sync DTO / event message / API request-response
- `Data` — DbContext / repository / data access
- `Security` — auth / JWT / middleware / policy
- `Middleware` — pipeline middleware / filters
- `Configuration` — startup / DI registration / options
- `Seeding` — database seed data
- `Handler` — command / event / message handler
- `Messaging` — message publisher / consumer / queue client

---

### STEP 3 — Detect Infrastructure Components

Scan all config files, docker-compose, appsettings, env files, and source code for references to external components. Build an `infraComponents` array:

```jsonc
{
  "id": "rabbitmq",
  "label": "RabbitMQ",
  "icon": "🐇",
  "subtitle": "AMQP · :5672",
  "category": "messaging",     // messaging|database|observability|storage|auth|gateway|erp|external
  "color": "#f97316",
  "notes": "Used by StoreServer.Infrastructure for sync notifications and ERP forwarding."
}
```

**Detection patterns:**

| Component | Look for |
|---|---|
| **RabbitMQ** | `RabbitMQ.Client`, `MassTransit`, `rabbitmq`, `amqp://`, `AMQP` |
| **Redis** | `StackExchange.Redis`, `redis://`, `IDistributedCache` + Redis |
| **PostgreSQL** | `Npgsql`, `postgres://`, `Host=`, `"Postgres"`, `pg` |
| **SQLite** | `Microsoft.Data.Sqlite`, `Data Source=`, `.db` file |
| **MySQL / MariaDB** | `MySqlConnector`, `Pomelo`, `mysql://` |
| **MongoDB** | `MongoDB.Driver`, `mongodb://` |
| **Kafka** | `Confluent.Kafka`, `kafka://`, `KAFKA_BOOTSTRAP` |
| **Prometheus** | `prometheus`, `AddPrometheusExporter`, `metrics_path`, `prometheus.yml` |
| **Grafana** | `grafana`, `GF_`, grafana folder |
| **Loki** | `loki`, `Serilog.Sinks.Grafana.Loki` |
| **OpenTelemetry** | `OpenTelemetry`, `OTEL_`, `otlp` |
| **Seq** | `Serilog.Sinks.Seq`, `seq:` |
| **Elasticsearch** | `Elastic.Clients`, `elasticsearch://` |
| **JWT Auth** | `AddJwtBearer`, `JwtBearer`, `HS256`, `RS256` |
| **API Key Auth** | `X-Api-Key`, `ApiKey` middleware |
| **Docker** | `docker-compose.yml`, `Dockerfile` |
| **Kubernetes** | `k8s/`, `*.yaml` with `kind: Deployment` |
| **ERP / External** | File-drop import jobs, `ErpImport`, external HTTP clients |
| **S3 / Blob** | `AWSSDK.S3`, `Azure.Storage.Blobs`, `minio` |
| **SignalR** | `AddSignalR`, `IHubContext` |
| **gRPC** | `.proto` files, `Grpc.AspNetCore` |

---

### STEP 4 — Identify Connections

Produce a `connections` array (edges between projects):

```jsonc
{
  "from": "ProjectA.Id",
  "to": "ProjectB.Id",
  "type": "http|di|reference|amqp|grpc|websocket|file",
  "label": "Short label (e.g. REST API, Sync Protocol, AMQP)",
  "description": "One sentence describing what flows over this connection."
}
```

**Connection detection rules:**
- `http` — One project has an `HttpClient` / typed client targeting another project's port
- `di` — Project A injects interfaces defined in Project B (same solution)
- `reference` — Project A has `<ProjectReference>` to Project B (shared domain / library)
- `amqp` — Project publishes to or consumes from RabbitMQ/Kafka
- `grpc` — Project uses `.proto` stubs from another project
- `file` — Import worker watches a folder written by another component

---

### STEP 5 — Build the Architecture Layers

Group projects into 2–5 layers:

```jsonc
{
  "name": "Layer Name",
  "description": "What this layer is responsible for.",
  "projects": ["ProjectA.Id", "ProjectB.Id"]
}
```

Common layer patterns:
- **Frontend Clients** — UIs (web, mobile, desktop)
- **API Gateway / BFF** — Aggregator APIs
- **Service Layer** — Core business APIs
- **Worker Layer** — Background / async workers
- **Data Layer** — Databases, caches
- **Infra / Messaging** — Queues, brokers
- **Shared Domain** — Shared libraries, contracts

---

### STEP 6 — Build the Data Flows

Produce a `dataFlows` array tracing meaningful end-to-end paths:

```jsonc
{
  "from": "ProjectA.Id or 'External Source'",
  "to": "ProjectB.Id or 'External Sink'",
  "protocol": "HTTP REST|AMQP|Direct DI|Project Reference|gRPC|WebSocket|File Import",
  "description": "1–2 sentences: what data moves and why."
}
```

Aim for 8–15 flows covering the full lifecycle of the most important operations.

---

### STEP 7 — Assemble the DATA JSON

Combine everything into this top-level object:

```jsonc
{
  "meta": {
    "project": "YourProjectName",
    "generated": "YYYY-MM-DD",
    "description": "One sentence.",
    "totalFiles": 0,
    "totalProjects": 0
  },
  "architecture": {
    "summary": "3–5 sentence plain-English description of the overall system.",
    "layers": [ /* layer objects */ ],
    "dataFlows": [ /* dataFlow objects */ ]
  },
  "projects": [ /* project objects with files */ ],
  "connections": [ /* connection objects */ ],
  "infraComponents": [ /* infra component objects */ ]
}
```

---

### STEP 8 — Generate the Self-Contained HTML File

Read `template/map.html` from this skill's folder (same directory as this SKILL.md).
Replace the single placeholder token `{{DATA_JSON}}` with the JSON string (no pretty-print needed).
Write the result to `<workspace-root>/docs/project-map.html`.

If the template file is not available (skill installed without template), reproduce the full HTML from the **HTML Template Specification** section below.

---

## HTML Template Specification

When the template file is not available, generate a file matching this full specification.

### Required Views (tabs)

| Tab | ID | What it renders |
|---|---|---|
| Architecture | `architecture` | Layered cards grid — one card per project |
| Infra Diagram | `diagram` | SVG diagram with all infra + app components, zones, labeled edges |
| Graph | `graph` | Force/layer SVG graph — nodes are projects, edges are connections |
| Data Flows | `flows` | Vertical flow cards showing protocol + description |

### Required UI Elements

- **Header**: logo, title, view-toggle buttons, search input
- **Stats bar**: counts for Projects, Files, Classes, Connections, Data Flows
- **Sidebar**: scrollable project list + connection list; clicking opens detail panel
- **Detail panel**: slide-in from right — shows description, patterns, connections, file list
- **File accordion**: each file expands to show classes, methods, deps/usedBy
- **Search**: live filter across file names, class names, method names, descriptions
- **Keyboard**: `Esc` closes panel · `Ctrl+K` focuses search

### Infra Diagram Zones

Draw rectangular zones with faint borders and uppercase label:
- **Client Apps** — all `frontend`, `mobile`, `desktop` type projects
- **API Layer** — all `api` type projects  
- **Worker Layer** — all `worker` type projects
- **Infrastructure / Messaging** — queues, databases, cache, external
- **Observability** — Prometheus, Grafana, Loki, OpenTelemetry, Seq
- **Shared Domain** — `library` type projects (rendered at bottom, full-width)

### Design Tokens

```css
:root {
  --bg:       #0f1117;
  --surface:  #1a1d27;
  --surface2: #232736;
  --border:   #2d3148;
  --text:     #e2e4ed;
  --text-dim: #8b8fa3;
  --accent:   #6366f1;
  --green:    #10b981;
  --amber:    #f59e0b;
  --purple:   #8b5cf6;
  --red:      #ef4444;
  --orange:   #f97316;
  --cyan:     #06b6d4;
  --radius:   12px;
  --transition: .2s cubic-bezier(.4,0,.2,1);
}
```

Font stack: `'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif`
Monospace: `'Cascadia Code', 'Fira Code', monospace`

### Data Injection Pattern

The output HTML must be **fully self-contained** — no external fetches, no CDN, no `fetch()` calls.
Inject data as a JS constant at the top of the `<script>` block:

```html
<script>
const DATA = { /* full JSON here */ };
// ... all rendering functions follow
</script>
```

This ensures the file works when opened directly from the filesystem (`file://`) without a web server.

---

## Analysis Depth Guide

### .NET / C#
- Parse `.csproj` for `<ProjectReference>` (→ `reference` connections)
- Read `Program.cs` for DI registrations, port, middleware
- Read `appsettings.json` for connection strings → infer infra
- Interface files → `Interface` category; implementation files → `Service` category
- `IHostedService` / `BackgroundService` → `Worker` category
- Minimal API endpoint groups → `Endpoint` category

### TypeScript / Node.js / Next.js
- `package.json` dependencies → infer infra (pg, mongoose, redis, amqplib)
- `src/app/` (Next.js App Router) → `Endpoint` category pages/routes
- `src/services/` → `Service` category
- `src/models/` or `src/types/` → `Model` category
- `src/middleware/` → `Middleware` category

### Python
- `requirements.txt` / `pyproject.toml` → infer infra
- `class` definitions → classes array
- `def` functions in service files → methods array
- SQLAlchemy models → `Model` category
- FastAPI route functions → `Endpoint` category

### Dart / Flutter
- `pubspec.yaml` → infra (http, dio, sqflite, hive)
- `*.dart` files: `Widget` subclasses → `Model`/`Service`
- Provider / Riverpod / Bloc → `Service` category
- Repository pattern classes → `Service` or `Data`

### Java / Kotlin (Spring)
- `@RestController` → `Endpoint`
- `@Service` / `@Component` → `Service`
- `@Repository` → `Data`
- `@Entity` → `Model`
- Spring Cloud / Feign clients → `http` connections

### Go
- `http.HandleFunc` / `gin.Engine` routes → `Endpoint`
- `struct` types in `models/` → `Model`
- `interface` in `ports/` → `Interface`

---

## Quality Checklist

Before writing the output file, verify:

- [ ] Every project in the solution has an entry in `projects`
- [ ] Every infrastructure component found in config has an entry in `infraComponents`
- [ ] The `architecture.layers` cover all projects (no orphans)
- [ ] Connections have both valid `from` and `to` IDs
- [ ] The HTML file has **no** `fetch()` call — data is inline
- [ ] The output is a single `.html` file that opens correctly in Chrome/Edge/Firefox
- [ ] The infra diagram SVG renders without overlap (adjust coordinates if needed)
- [ ] Search works across all text content
- [ ] The file size is reasonable (< 2 MB for most projects)

---

## Output

Write the final file to: `<workspace-root>/docs/project-map.html`
Print a summary like:

```
✓ arcmap complete
  Output: docs/project-map.html
  Projects: 11  ·  Files: 84  ·  Classes: 210  ·  Connections: 12  ·  Infra components: 9
  Open: file:///path/to/docs/project-map.html
```
