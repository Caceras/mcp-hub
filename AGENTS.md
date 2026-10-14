# Existing GitHub Extra MCP

Before asking Riki for GitHub login or saying repository creation is unavailable,
read docs/GITHUB.md. Reuse the deployed Caceras/github-mcp repository administration
tools; the stock GitHub connector alone does not expose all these capabilities.

Never commit the private endpoint or GitHub credentials. Resolve live access using
Dokploy domain metadata, and verify repository_admin_status. Keep this repo as an
operational read model; implementation belongs in the actual provider repo and
deployment truth comes from live probes/Dokploy.
