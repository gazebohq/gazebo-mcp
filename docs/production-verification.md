# Production protocol verification

On 25 October 2026, twenty authenticated HTTP protocol checks passed against https://app.gazebohq.com/api/mcp. Initialization, notification 202, tool discovery, identity, primary/named fictional-key retrieval, service and method denials, missing-key errors, audit outcomes and cursor pagination, token rejection, Origin validation, unsupported protocol headers, GET 405, no-store and post-revocation rejection were checked.

The run created one expiring temporary agent in the dedicated smoke account using its established fictional service key. Only that agent's access policy changed. The agent was revoked, cleanup confirmed zero active test agents remained, and the fictional credential was not sent to any provider API. Existing service credentials were unchanged. Audit events remain and normal access-denied notifications may have been triggered. No tokens, credential responses, account identifiers or raw transcripts are included here.

These are direct JSON-RPC HTTP checks, not actual Claude Code, Codex, Cursor, Windsurf or desktop connector runs. Separate native CLI and user-performed desktop/hosted OAuth checks are recorded in [client verification](client-verification.md); their evidence is not inferred from these HTTP checks. The separate OpenAI Responses API run below passed. The official MCP Registry entry is prepared but not submitted.

## OpenAI Responses API verification

Passed 25 October 2026 using the real OpenAI endpoint `https://api.openai.com/v1/responses`,
model `gpt-5.4-mini`, from the Linux workspace. This used the user's restricted
OpenAI project key, not Replit's AI proxy, and the deployed Gazebo MCP endpoint.
It does not establish Codex or ChatGPT compatibility.

Ten checks passed:

1. Dedicated smoke-account and fictional-service prerequisites.
2. Creation of one expiring temporary agent with GET-only service access.
3. Direct preflight verification of the exact primary and named fictional values
   before allowing OpenAI to retrieve them.
4. OpenAI discovery of all three tools, explicit approval, and correct agent identity.
5. Primary fictional-key retrieval through an actual OpenAI MCP call.
6. Named fictional-key retrieval through an actual OpenAI MCP call.
7. Denied POST retrieval.
8. Denied GET retrieval after disabling service access.
9. Audit success/denial verification across two cursor pages.
10. Agent revocation, failed OpenAI MCP authentication with that token, and direct
    confirmation of HTTP 401 for the same revoked token.

The harness required approval for every tool call and checked the exact tool name
and arguments before approving. It inspected actual `mcp_call` results rather than
accepting a model's claim of success. Fifteen OpenAI requests used 4,667 input tokens
and 321 output tokens. API billing is determined by OpenAI; no cost is inferred here.

All requests used `store: false`. This disables response storage for the API
workflow but is not a claim of zero retention or exemption from OpenAI's policies.
Only fictional provider credentials were sent to OpenAI. The temporary Gazebo agent
was revoked; cleanup verified zero remaining active test agents. Existing services
and credentials were unchanged. Audit records remain and normal notifications may
have been triggered.

Reproduction requires new authorization and both secrets through the workspace's
Secrets tooling:

```sh
node scripts/src/openai-mcp-runtime.mjs --allow-mutations
```

The script requires `OPENAI_API_KEY` and `GAZEBO_SMOKE_ACCOUNT_TOKEN`, caps API
requests, bounds timeouts, and prints only check statuses, usage counts and cleanup
results. No token, raw tool output, account identifier or response ID is recorded.
Revoke the temporary OpenAI key in its provider dashboard after verification.
