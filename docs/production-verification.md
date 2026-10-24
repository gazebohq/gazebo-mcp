# Production protocol verification

On 25 October 2026, twenty authenticated HTTP protocol checks passed against https://app.gazebohq.com/api/mcp. Initialization, notification 202, tool discovery, identity, primary/named fictional-key retrieval, service and method denials, missing-key errors, audit outcomes and cursor pagination, token rejection, Origin validation, unsupported protocol headers, GET 405, no-store and post-revocation rejection were checked.

The run created one expiring temporary agent in the dedicated smoke account using its established fictional service key. Only that agent's access policy changed. The agent was revoked, cleanup confirmed zero active test agents remained, and the fictional credential was not sent to any provider API. Existing service credentials were unchanged. Audit events remain and normal access-denied notifications may have been triggered. No tokens, credential responses, account identifiers or raw transcripts are included here.

These are direct JSON-RPC HTTP checks, not actual Claude Code, Codex, Cursor, Windsurf or desktop connector runs. The vendor-client matrix remains pending. The official MCP Registry entry is prepared but not submitted.
