# Host4AI MCP Hub

> **TRANSITIONAL READ MODEL — not a source of production truth.**
>
> Desired deployment state belongs in `Caceras/host4ai/ops`; implementation
> belongs in `Caceras/connectors`; live status must come from probes and
> Dokploy. This repository currently holds useful design history and standards,
> but its hand-maintained `registry.json` must not be treated as authoritative.
> The target is to generate the private MCP catalog inside Host4AI using the
> official MCP Registry metadata model, then archive or reduce this repository.

Canonical inventory and operating standard for Riki's custom Model Context Protocol servers.

## Roles

- **mcp-hub**: source-of-truth registry, health/compatibility metadata and operational docs.
- **mcp-template**: starter for new standalone MCP servers.
- **connectors**: existing shared provider runtime; migrate providers toward the hub standard without breaking live clients.
- **Dokploy / connectors / production**: canonical production deployment location.
- **mcp.host4ai.se**: canonical public hostname for remote MCP endpoints.

Each provider remains a separate MCP surface even when multiple providers share code or a repository.

See `docs/STANDARD.md` and `registry.json`.
