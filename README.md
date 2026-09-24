# Host4AI MCP Hub

Canonical inventory and operating standard for Riki's custom Model Context Protocol servers.

## Roles

- **mcp-hub**: source-of-truth registry, health/compatibility metadata and operational docs.
- **mcp-template**: starter for new standalone MCP servers.
- **connectors**: existing shared provider runtime; migrate providers toward the hub standard without breaking live clients.
- **Dokploy / connectors / production**: canonical production deployment location.
- **mcp.host4ai.se**: canonical public hostname for remote MCP endpoints.

Each provider remains a separate MCP surface even when multiple providers share code or a repository.

See `docs/STANDARD.md` and `registry.json`.
