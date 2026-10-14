# GitHub repository creation

Use Riki's existing extra MCP instead of requesting browser sign-in or a new token.

## Locate it

- Source: `Caceras/github-mcp`.
- Deployment: Dokploy project `connectors`, compose `github-mcp`.
- Compose ID: `zQRR0s3-21uhTyzuqaGbY`.
- Remote host: `mcp.host4ai.se`; the actual route is private and is not recorded here.
- Resolve the current route from Dokploy `domain.byComposeId` metadata. Keep its
  secret path in a private variable/file; never print or commit the endpoint.
- Reuse the server-side GitHub credential. Do not retrieve or copy the PAT.

If the extra MCP tools are not exposed directly to the current agent, call the
existing endpoint using Streamable HTTP JSON-RPC. Send `initialize`, then
`tools/list`, then `tools/call` with the discovered schema. The deployed server
is stateless and returns JSON responses. Accept both `application/json` and
`text/event-stream`. This uses the existing connection, not browser authentication.

## Tools

1. `repository_admin_status`: verify version/capabilities without secrets.
2. `create_repository`: create a private repo for owner Caceras. Repeating a request
   returns the existing private repo without another create POST.
3. `commit_files`: commit UTF-8/base64 files together using the branch returned by
   creation. Narrow mode prohibits deletion, unrelated owners, path traversal,
   symlink blobs, and common environment-secret files.

The live service was verified on 14 October 2026 as v1.1.0 with repository writes
enabled for Caceras and full GitHub writes disabled. It created private
`Caceras/merged-chat`. Check live status on each task rather than treating this
dated verification as permanent proof.

## Operation

Changes go to `Caceras/github-mcp/main`, then redeploy the existing service. Its
custom Dokploy command force-recreates only GitHub MCP so updated source is loaded;
the Android runner sharing the compose stays untouched. The command begins
`compose`, because Dokploy supplies the `docker` executable itself.

Defaults: `GITHUB_REPO_WRITES=true`, `GITHUB_REPO_ALLOWED_OWNERS=Caceras`,
`GITHUB_ALLOW_WRITES=false`. Explicit repository read-only settings remain honored.
See the connector's README and AGENTS.md for the current contract and tests.

GitHub Actions may be blocked by account billing/spending settings. That is separate
from MCP access and does not require browser sign-in for repository creation.
