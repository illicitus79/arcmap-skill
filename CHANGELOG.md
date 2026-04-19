# Changelog

All notable changes to arcmap are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/), and versions follow semver.

## [0.2.0] — 2026-04-19

Flexibility, safety, and polish pass. The map is now useful on any stack, safer to
generate from untrusted codebases, and much easier to share.

### Added
- **Endpoints view** — searchable table of every HTTP route in the repo (method, path,
  project, description). Driven by the new optional `files[].endpoints[]` schema field,
  with automatic fallback to method-name parsing.
- **Light / dark theme toggle** — persists in `localStorage`; light theme screenshots
  well for external docs.
- **Pan + zoom** on Graph and Infra Diagram SVGs (drag to pan, wheel to zoom, button
  to reset).
- **Click-to-highlight** — clicking a project node dims unrelated nodes/edges so you
  can trace a dependency path at a glance.
- **Connection-type filter bar** — toggle HTTP / DI / AMQP / gRPC / GraphQL / tRPC /
  WebSocket / SSE / shared-db / env / webhook on the fly.
- **Deep-linkable URL state** — `#view=graph&project=orders-api&filter=reference,env`.
  Share a specific view of the map by copying the URL.
- **Export menu** — PNG, SVG, JSON, **Mermaid** (paste into any README / Notion /
  Obsidian), Markdown summary, and native Print / PDF.
- **Keyboard shortcuts** — `1–5` switch views, `T` theme, `E` export, `?` help modal,
  `Ctrl/Cmd+K` search, `Esc` close.
- **Endpoints schema** — new `files[].endpoints[]` with `{method, path, description, auth}`.
- **Env-var audit** (optional) — new top-level `envVars[]` for a cross-project view of
  environment dependencies.
- **Monorepo detection** — Nx, Turborepo, pnpm / Yarn / npm workspaces, Lerna, Bazel,
  Cargo workspaces, Go workspaces, .NET solutions, Gradle multi-project, Maven.
- **New connection types**: `graphql`, `trpc`, `sse`, `shared-db`, `env`, `webhook` —
  each with distinct edge styling.
- **Wide infrastructure coverage** — added detection signatures for Kafka, Redpanda,
  Pulsar, NATS, SQS/SNS/EventBridge, Pub/Sub, Azure Service Bus, Temporal, DynamoDB,
  Cosmos, Firestore, Supabase, PlanetScale, ClickHouse, Cassandra, Neo4j, Meilisearch,
  Typesense, Algolia, Loki, Tempo, Jaeger, Sentry, Datadog, New Relic, Honeycomb, MinIO,
  R2, Keycloak, Auth0, Clerk, NextAuth, Traefik, Envoy, Kong, Ocelot, YARP, Helm,
  Terraform, Pulumi, Serverless, SST, Dapr, OpenAI, Anthropic, Azure OpenAI, Vertex AI,
  Ollama, LangChain, Pinecone, Weaviate, Qdrant, Chroma, Stripe, PayPal, Twilio,
  SendGrid, Slack, Socket.IO, Cloudflare Workers, Vercel, Netlify.
- **Broader language coverage** — Rust (Axum/Actix/Rocket/Warp), Ruby (Rails, Sinatra,
  Sidekiq), PHP (Laravel, Symfony), Elixir (Phoenix, Oban), Swift (Vapor, SwiftUI),
  Scala (Play, Akka, Http4s, ZIO), Haskell (Servant, Yesod). TypeScript section expanded
  with NestJS, Hono, tRPC, SvelteKit, Nuxt, Angular.
- **Print-friendly stylesheet** — `@media print` hides chrome; content paginates cleanly.
- **Toast notifications** for clipboard actions.
- **Clickable source/sink chips** in Data Flows view jump straight to the project detail.
- **Per-project "Copy deep link" and "Copy as Markdown" actions** in the detail panel.
- **CHANGELOG.md** (this file).

### Fixed
- **Graph view crash** — syntax error in `renderGraph` positions assignment
  (`positions[p.id]={(x:…,y)}` → `{x:…,y}`) had made the view unusable.
- **HTML injection / XSS** — every `DATA`-sourced value interpolated into `innerHTML`
  is now HTML-escaped. Malformed codebases (file paths with `<`, descriptions with
  stray tags) no longer corrupt the page.
- **Connection label overlap** — edge-label backgrounds now auto-size based on label
  length.
- **Infra-component click-through** — infra nodes now open a detail panel listing the
  projects that use them.

### Changed
- **Schema** (`schema/data.schema.json`): required-core unchanged (fully backward
  compatible); new optional fields — `meta.repository`, `meta.monorepo`,
  `meta.primaryStack`, `projects[].language`, `projects[].framework`,
  `projects[].entrypoints`, `files[].endpoints`, `envVars`. `Project.type`,
  `Connection.type`, `SourceFile.category`, and `InfraComponent.category` are now
  open enums (common values styled, arbitrary strings rendered neutrally) so
  domain-specific taxonomies are welcome without a schema change.
- **Template CSS** — consolidated design tokens behind `data-theme` attribute; reduced
  shadow intensity in light mode; scrollbar styling neutralised for both themes.
- **Sidebar connection links** — now reference connections by index rather than by
  `from/to` pair, fixing ambiguity when two projects have multiple connection types.

### Migration

Existing DATA JSON from 0.1.0 renders unchanged. New features activate progressively:

- To light up the **Endpoints view**, populate `files[].endpoints[]` on files with
  `category: "Endpoint"`.
- To light up **monorepo metadata** in the header, set `meta.monorepo` and
  `meta.primaryStack`.
- To use new **connection types** (`graphql`, `trpc`, `sse`, `shared-db`, `env`,
  `webhook`), just set them in `connections[].type` — they get dedicated edge colors.

---

## [0.1.0] — 2026-04-19

Initial release. Five views (Architecture, Infra Diagram, Graph, Data Flows, Search),
multi-language analysis, self-contained HTML output.
