# arcmap

**An AI agent skill that turns any codebase into a fully interactive, self-contained architecture map.**

One command → one HTML file → open in any browser, no server, no dependencies.

---

## What it produces

A single `docs/project-map.html` file (~1–3 MB) with five interactive views:

| View | Description |
|---|---|
| **Architecture** | Layered card grid — one card per project, grouped by architectural tier |
| **Infra Diagram** | SVG diagram showing all components (RabbitMQ, Postgres, Prometheus, Grafana, ERP…) with labeled edges |
| **Graph** | Force-layout graph — project nodes connected by typed edges (HTTP, DI, AMQP, gRPC…) |
| **Data Flows** | Step-by-step flow cards showing protocol and purpose for each data path |
| **Search** | Live full-text search across files, classes, method signatures, and descriptions |

Clicking any project, file, or connection opens a slide-in detail panel showing:
- Description & architecture patterns
- All connections (in/out) with protocol labels
- File list with expandable accordions containing classes, methods, dependencies

---

## How to use

### With GitHub Copilot (VS Code)

Install this skill:

```bash
npx skills add iWhite/arcmap -g
```

Then in any workspace, type:

```
/arcmap
```

### With Claude / Anthropic Console

Point the agent at `SKILL.md` in this repository and invoke:

```
/arcmap
```

### Manual installation

Clone this repo anywhere and add the path to your agent's skill configuration:

```bash
git clone https://github.com/your-username/arcmap.git ~/.agents/skills/arcmap
```

---

## Options

```
/arcmap                      # map current workspace → docs/project-map.html
/arcmap --out <path>         # custom output path
/arcmap --depth shallow      # skip method-level analysis (faster on large repos)
/arcmap --depth deep         # include all method signatures (default: medium)
/arcmap --no-infra           # skip infrastructure component detection
```

---

## What it analyses

| Language / Framework | What's extracted |
|---|---|
| **C# / .NET** | Projects, namespaces, interfaces, classes, methods, EF Core entities, DI registrations, ports |
| **TypeScript / Node.js** | Services, routes, types, Next.js pages, API handlers |
| **Python** | Classes, functions, FastAPI routes, SQLAlchemy models |
| **Dart / Flutter** | Widgets, providers, repositories, Bloc/Cubit |
| **Java / Kotlin (Spring)** | Controllers, services, repositories, entities |
| **Go** | Handlers, structs, interfaces |
| **Rust** | Modules, structs, traits |

Infrastructure detection reads: `docker-compose.yml`, `appsettings*.json`, `.env`, `prometheus.yml`, `Dockerfile`, `k8s/*.yaml`, and package manifests.

---

## Project structure

```
arcmap/
  SKILL.md                   # AI agent skill instructions (Copilot + Claude)
  README.md                  # This file
  LICENSE                    # MIT
  template/
    map.html                 # Self-contained HTML template ({{DATA_JSON}} placeholder)
  schema/
    data.schema.json         # JSON Schema for the DATA object
```

---

## Template customisation

The HTML template at `template/map.html` is a standalone file with a single placeholder:

```html
<script>
const DATA = {{DATA_JSON}};
```

You can fork this repo and modify the template freely — the CSS design tokens are at the top of the `<style>` block. The JavaScript rendering functions are well-separated by view.

**Dark theme design tokens:**

```css
:root {
  --bg:       #0f1117;
  --surface:  #1a1d27;
  --accent:   #6366f1;
  --green:    #10b981;
  --amber:    #f59e0b;
  /* ... */
}
```

---

## Output example

After running `/arcmap` on a .NET + Next.js + Flutter project:

```
✓ arcmap complete
  Output:    docs/project-map.html
  Projects:  11   ·  Files: 84   ·  Classes: 210
  Connections: 12  ·  Infra: 9   ·  Data Flows: 14
  Open: file:///your/workspace/docs/project-map.html
```

The file is 100% self-contained — no internet connection required, works offline.

---

## Why no `fetch()`?

Browsers block `fetch()` under the `file://` protocol (CORS). arcmap embeds all data as an inline JavaScript constant so the file works when double-clicked from the filesystem.

---

## License

MIT — see [LICENSE](LICENSE)
