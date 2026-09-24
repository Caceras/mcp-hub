# Host4AI MCP standard

Status: canonical operating contract for custom MCPs.

## Architecture

Use three layers:

1. **Caceras/connectors** — shared runtime and provider implementations.
2. **Caceras/mcp-template** — starter for exceptional standalone MCPs that cannot fit the shared runtime.
3. **Caceras/mcp-hub** — registry, compatibility matrix, health state and operational documentation.

Production custom MCPs live in Dokploy project **connectors**, environment **production**. Remote MCPs use `mcp.host4ai.se/<provider>/mcp` as the canonical URL. Do not put credentials or bearer secrets in URLs.

## Protocol

- Target final MCP protocol revision **2026-07-28**.
- Use the stable MCP SDK v2 where a language SDK is available.
- Serve modern 2026 traffic and legacy 2025-era traffic from the same endpoint while ChatGPT/Claude compatibility still requires it.
- Remote transport: Streamable HTTP / SDK HTTP handler, stateless where practical.
- Local developer transport: stdio may be offered in addition to HTTP.
- Validate `Origin` on remote HTTP.
- One provider surface per logical service even when implementations share a process.

## Authentication

Remote private MCPs use OAuth. Requirements:

- OAuth protected-resource metadata.
- Authorization-server metadata.
- PKCE S256.
- Refresh tokens / offline access so clients survive access-token expiry.
- Durable client/token state across redeploys.
- Exact allowlist for the owner account.
- Credentials stay server-side and are never returned by tools.
- No new secret-in-path endpoints. Existing secret paths are migration-only.

Upstream provider OAuth (for example Google Tasks) is distinct from MCP-client OAuth. Store upstream credentials only in Dokploy secrets/runtime state.

## Metadata

Every server/provider declares:

- stable id
- human title
- concise description
- usage instructions
- semantic version
- canonical MCP URL
- website/docs URL
- repository
- owner
- protocol versions/eras
- auth model
- data residency/hosting
- icon
- tool count
- read/write classification

Every tool declares:

- stable machine name
- human title
- description that tells the model when to use it
- strict input schema
- output/structured shape where practical
- `readOnlyHint`
- `destructiveHint`
- `idempotentHint`
- `openWorldHint` when applicable

Do not expose one generic raw HTTP/API tool when a small semantic tool surface can represent the real jobs safely.

## Icons

The registry owns one canonical square icon per MCP. Prefer a provider's official product mark where licensing/use permits; otherwise use a Host4AI-branded service glyph. Requirements:

- square
- PNG and/or SVG source retained
- at least 128x128 raster
- stable public URL
- same icon used in server metadata, hub UI, ChatGPT app setup, Claude connector listing and Ægentica registry
- never rely on a manually uploaded icon as the only copy

## Safety

- Reads may execute directly.
- Writes require server-side write enablement plus explicit user approval for the exact mutation.
- Destructive actions are separately annotated and may have a stronger server-side gate.
- Secrets are redacted recursively from logs/tool output.
- Bound response size, pagination and log tails.
- Set finite upstream timeouts and avoid long synchronous deploy/build calls.

## Health

Every MCP has:

- `/health`: process/router/configuration health; no provider mutation.
- authenticated provider health tool.
- `/deep-health` where practical: one harmless upstream read proving credentials + network + API.
- deployment state from Dokploy.
- CI contract tests before deployment.

A service is **green** only when build/tests pass, Dokploy reports healthy/running, protocol discovery succeeds, `tools/list` succeeds, metadata/icon resolve, OAuth refresh works, and deep health succeeds.

## Compatibility gates

Each production MCP must be tested against:

- modern MCP client using 2026-07-28
- legacy 2025-era MCP client
- ChatGPT custom app/tool scan
- Claude remote custom connector
- Ægentica MCP client in automatic protocol negotiation mode

Compatibility results belong in `mcp-hub/registry.json`, not in memory or chat history.

## Release

1. Change provider code.
2. Run unit/schema/security tests.
3. Build container.
4. Deploy to staging or isolated path.
5. Run protocol + shallow + deep health.
6. Verify ChatGPT, Claude and Ægentica compatibility.
7. Promote canonical route.
8. Update registry automatically.
9. Keep old route only for a defined migration period.

## Versioning

- MCP implementation version uses semver.
- Tool names are stable public API.
- Breaking schema/tool-name changes require a major version or compatibility alias.
- Registry records the git commit and deployed image/build revision.
- Client tool scans may cache schemas, so tool changes must be explicitly refreshed/re-scanned in clients.
