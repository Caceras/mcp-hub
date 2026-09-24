# Host4AI MCP standard

Status: canonical operating contract for custom MCPs.

## First principles

The user expresses intent; infrastructure details stay below the interface.

The system is successful only when an integration is usable from the intended clients, survives redeploys, can be verified automatically, and can be reconstructed from source. "Container running", "code committed", and "MCP endpoint exists" are intermediate states, not success.

Prefer, in order:

1. an official provider MCP when it exposes the needed capability;
2. an existing shared Host4AI connector/provider;
3. extending an existing provider identity/runtime;
4. a new provider in the shared runtime;
5. a standalone MCP only when isolation or a different runtime is genuinely required.

Do not create a new OAuth client, repository, container, hostname, database, or auth system if an existing one can safely own the capability.

## Source of truth

Use four layers with one responsibility each:

1. **Caceras/host4ai/ops** — desired production state and reconciliation into Dokploy.
2. **Caceras/connectors** — shared runtime and provider implementations.
3. **Caceras/mcp-hub** — generated/read-model registry, compatibility matrix, health state and operational metadata. It is not an independent source of desired state.
4. **Caceras/mcp-template** — starter only for exceptional standalone MCPs.

Dokploy is a runtime target, not the source of truth. Production changes should flow from declarative state in git, then reconcile and verify.

Remote custom MCPs use `mcp.host4ai.se/<provider>/mcp` as the canonical URL. Never put credentials or bearer secrets in URLs.

## Protocol

- Target final MCP protocol revision **2026-07-28**.
- Prefer an official stable MCP SDK where available.
- New authorization clients should follow **Client ID Metadata Documents (CIMD)**. Dynamic Client Registration is compatibility-only because MCP 2026-07-28 deprecates DCR for new implementations.
- Serve modern 2026 traffic and legacy 2025-era traffic from the same logical endpoint while real clients still require compatibility.
- Remote transport is stateless HTTP where practical. Do not introduce legacy SSE for new work.
- Local developer transport may additionally expose stdio.
- Validate `Origin` on remote HTTP.
- One provider surface per logical service even when several providers share one process.
- Do not adopt newly deprecated MCP features for new work merely because an older SDK exposes them.

## Identity and authorization

There are two independent identities:

1. **MCP client identity** — ChatGPT, Claude, Ægentica or another MCP client authenticating to Host4AI.
2. **Upstream provider identity** — Host4AI accessing Google, Loopia, Simply, GitHub, etc.

Never conflate them.

For private remote Host4AI MCPs:

- OAuth protected-resource metadata.
- Authorization-server metadata.
- PKCE S256 where applicable.
- Refresh/offline capability where the client/provider supports it.
- Durable auth state across redeploys.
- Exact owner/account allowlists where appropriate.
- Credentials remain server-side and never appear in tools, logs, URLs, registry data or chat.
- Existing secret-path routes are migration-only.

For upstream providers, reuse the existing **provider project / consent brand / governance boundary** when practical, but do not collapse credentials across materially different trust scopes merely to reduce setup.

Example: Google Tasks should normally live in the same existing Google Cloud project and consent brand as the rest of the private Google integration, while using a dedicated OAuth client credential if the existing credential is also the generic sign-in identity for unrelated MCPs. This keeps least privilege and token lifecycles clean without multiplying projects.

Provider-required user consent remains an interactive security boundary; everything before and after that consent should be automated.

## Tool design

Expose user jobs, not raw APIs.

Every tool declares:

- stable machine name
- human title
- concise description explaining when to use it
- strict input schema
- structured output where practical
- `readOnlyHint`
- `destructiveHint`
- `idempotentHint`
- `openWorldHint` when applicable

Prefer a small semantic tool surface over generic `request`, `curl`, or arbitrary provider-API tools. A generic escape hatch may exist for administration, but it is not the default model-facing surface.

## Metadata

Every provider entry declares:

- stable id
- human title
- concise description
- usage instructions
- semantic version
- canonical MCP URL
- website/docs URL
- implementation repository
- desired-state manifest
- owner
- supported protocol eras
- auth model
- hosting/data-residency notes
- canonical icon
- tool count
- read/write/destructive classification
- deployed git revision
- client compatibility results
- shallow and deep health state

Metadata is generated or verified from source and live probes where possible; do not maintain the same fact manually in several places. The registry should be rebuildable from provider manifests, provider code metadata, deployment state and compatibility probes.

## Icons

The registry owns one canonical icon per integration.

- Prefer the provider's official product mark when use is appropriate.
- Otherwise use a consistent Host4AI service glyph.
- Keep an SVG/source asset plus at least a 128x128 raster where needed.
- Serve it from a stable public URL.
- Reuse the same canonical asset in MCP metadata, Host4AI, ChatGPT configuration, Claude configuration and Ægentica.
- A manually uploaded client icon is never the canonical copy.

## Safety

- Read-only operations may execute directly.
- Mutations require the client's normal approval model plus server-side policy where appropriate.
- Destructive or high-impact actions receive stronger policy gates.
- Do not add confirmation fields mechanically when the host already supplies a secure approval primitive; safety must be useful, not ritual.
- Secrets are recursively redacted from logs and tool output.
- Bound response sizes, pagination, log tails and upstream timeouts.
- Long-running work should use durable/asynchronous mechanisms rather than holding an HTTP request open.

## Health

A provider is not "working" because its process is alive.

Each provider should expose or support:

- process/router health
- protocol initialization/discovery
- metadata/icon resolution
- authenticated `tools/list`
- one harmless real upstream read
- auth refresh/renewal validation where applicable
- deployment revision
- CI contract tests
- client compatibility results

A provider is **green** only when the full chain works:

`client -> auth -> MCP -> tool -> upstream provider -> structured result`.

## Compatibility

Production integrations are verified against the actual intended consumers:

- ChatGPT remote custom app/connector
- Claude remote custom connector
- Ægentica MCP client
- a protocol conformance/smoke client

The server negotiates protocol capabilities; application code does not fork into separate ChatGPT, Claude and Ægentica implementations.

Compatibility results live in `mcp-hub/registry.json`, not in chat history.

## Release

The normal release path is:

`intent -> provider change -> tests -> desired-state manifest -> plan -> apply -> protocol smoke -> deep health -> client verification -> registry green`

Concretely:

1. Change shared provider code or register an official external MCP.
2. Run unit/schema/security tests.
3. Update `Caceras/host4ai/ops` desired state.
4. Reconciler plans drift before mutation.
5. Apply only the intended integration.
6. Verify endpoint, auth, tool discovery and harmless provider read.
7. Verify ChatGPT, Claude and Ægentica.
8. Record deployed commit and compatibility in the registry.
9. Remove compatibility routes after a defined migration window.

No production state should exist only in a browser dashboard.

## Versioning

- MCP implementation versions use semver.
- Tool names are public API and remain stable.
- Breaking tool/schema changes require a major version or compatibility alias.
- Registry records the git commit and deployed revision.
- Client tool schemas may be cached, so deploy tooling explicitly triggers or documents rescans when needed.
