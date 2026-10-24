# Gazebo MCP

Credential retrieval for AI agents via the Model Context Protocol, with scoped identities, audit events and revocable future access.

Gazebo hosts the MCP server. You do **not** need to install or run a local Gazebo server.

- **Endpoint:** `https://app.gazebohq.com/api/mcp`
- **Authentication:** MCP OAuth with explicit key approval, or `Authorization: Bearer <agent-token>`
- **Transport:** Stateless, JSON-only Streamable HTTP over `POST`
- **Create an agent token:** [app.gazebohq.com/agents](https://app.gazebohq.com/agents)
- **Documentation:** [gazebohq.com/docs/mcp](https://gazebohq.com/docs/mcp)

> Gazebo's MCP endpoint accepts scoped agent bearer tokens and resource-bound MCP OAuth access tokens. Account API tokens and browser sessions are rejected.

## Why Gazebo

Long-lived credentials should not be copied into every agent's configuration. Gazebo gives each agent an identity and a policy describing which service credentials it may retrieve and the declared HTTP methods permitted for retrieval. Provider key permissions, not Gazebo, govern subsequent API calls. The agent retrieves an approved credential only when it needs one, while credential-access attempts—including denials—are recorded in the audit trail.

The actual provider credential is delivered to the MCP client or agent runtime and can be accessed by its host. Cloud connectors, gateways and model providers may receive it and may retain tool output according to their settings. Treat it as ephemeral where possible and do not deliberately persist it. Revoking Gazebo access blocks future retrieval; rotate or revoke the provider key to invalidate copies already received.

## Available tools

### `get_identity`

Returns the authenticated agent's identity and access policy. It does not return a credential and does not create a credential-access audit event.

Call this first to confirm that the token resolves to the intended agent and to inspect accessible services and allowed methods.

```text
get_identity()
→ {
    status: "ok",
    agent_id: "...",
    name: "my-agent",
    accessible_services: [
      { service: "github", allowed_methods: ["GET"] }
    ]
  }
```

### `get_credential`

Retrieves an active credential for an allowed service and intended HTTP method. Valid credential-retrieval attempts are audited before a secret is released. Authentication and malformed-request failures are not credential-access events. Audit persistence failure blocks release.

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `service` | string | Yes | Service slug, such as `github` or `stripe` |
| `method` | string | Yes | Intended method: `GET`, `POST`, `PUT`, `PATCH`, or `DELETE` |
| `key_name` | string | For OAuth; optional for manual agent tokens | OAuth requires an explicitly approved named key. Manual agent tokens may omit it to select one deterministic primary active credential. |

```text
get_credential({ service: "github", method: "GET" })
→ "<ephemeral-service-credential>"
```

Do not retry a denied request until the user changes the policy or you choose an allowed method.

### `list_audit_events`

Returns recent credential-access attempts for the authenticated agent.

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `limit` | integer | No | Number of events, from 1 through 50 |
| `service` | string | No | Filter by service |
| `cursor` | string | No | Opaque pagination cursor |

## Recommended flow

1. Call `get_identity` to verify the authenticated agent and policy.
2. Call `get_credential` with the service and actual intended HTTP method.
3. Use the returned credential for the immediate operation, then discard it.
4. Optionally call `list_audit_events` to verify the access event.

## Client configuration

Client configuration changes independently of Gazebo. Use the current client documentation for the exact configuration surface and field names. For clients using manual agent tokens, use:

```text
URL: https://app.gazebohq.com/api/mcp
Authorization: Bearer <your-agent-token>
```

### Cursor

Cursor currently supports project configuration in `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "gazebo": {
      "url": "https://app.gazebohq.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${env:GAZEBO_AGENT_TOKEN}"
      }
    }
  }
}
```

Do not commit a real agent token. Prefer the current client-supported secret or environment-variable mechanism where available.

### OpenAI Codex and Responses API

Codex supports remote HTTP MCP using an environment-based bearer token:

```toml
[mcp_servers.gazebo]
url = "https://app.gazebohq.com/api/mcp"
bearer_token_env_var = "GAZEBO_AGENT_TOKEN"
```

Set the variable in Codex's inherited environment. See [OpenAI setup](docs/openai.md) for configuration, identity-only Responses API guidance and ChatGPT web limitations. Codex configuration is source-reviewed but not runtime-tested. A separate direct OpenAI Responses API test passed with fictional credentials; see the production verification report.

### Claude Desktop

In Settings → Connectors, add a custom connector:

- URL: `https://app.gazebohq.com/api/mcp`
- Authentication: **Sign in now**
- OAuth client: **Use Claude’s published identity**
- Transport: **Streamable HTTP**

Click Add, then Connect. Sign into Gazebo, select only the keys to share, choose an expiry and approve. In Claude, call Get identity first. Use fictional credentials for your first retrieval test. Revoke the grant in Gazebo to block further retrieval. A key already returned remains accessible to the runtime/host until invalidated at the provider.

### Claude Code

Claude Code has a separate configuration surface. Configure remote HTTP with the canonical endpoint and agent authorization header above; follow Anthropic’s current documentation for your release. The Claude Desktop verification does not establish Claude Code compatibility.

### Windsurf

Configure a remote MCP server using the canonical endpoint and authorization header above. Windsurf's configuration field names and locations vary by release, so follow Windsurf's current documentation rather than assuming a particular `url` or `serverUrl` field.

## Verification checklist

After configuring a client:

1. Confirm initialization and tool discovery list all three Gazebo tools.
2. Call `get_identity` and verify the expected agent name and policy.
3. Call `get_credential` for an allowed service and method.
4. Call `list_audit_events` and verify the credential-access event is present.
5. Confirm an account API token, invalid token, and revoked agent token are rejected.

See [docs/client-verification.md](docs/client-verification.md) for the release-test matrix.

## Protocol behavior

Gazebo supports MCP protocol versions `2024-11-05`, `2025-03-26`, and `2025-06-18`. Future version requests negotiate to `2025-06-18`.

The endpoint is stateless, JSON-only Streamable HTTP over `POST`. Accepted notifications return HTTP 202 with no body. Unauthenticated GET requests return HTTP 401 with OAuth discovery information. Authenticated GET and DELETE requests are not supported MCP operations. Unsupported explicit MCP-Protocol-Version headers return HTTP 400. Native clients may omit Origin; untrusted explicit Origins return HTTP 403. Responses use Cache-Control: no-store. Gazebo does not claim SSE, sessions, or server streaming.

Credential denials and missing keys use HTTP 200 with result.isError: true, not REST-style 403/404. Inspect the tool result as well as HTTP status.

## Readiness and registry status

Authenticated production protocol checks passed using an existing fictional-key fixture, including retrieval, denial, audit pagination and revoked-token rejection. See [production verification](docs/production-verification.md). The separate user-performed Claude Desktop OAuth test passed approval, tool discovery, identity, named retrieval and post-revocation authentication rejection; see the client verification matrix for its limits.

The remote-only [server.json](server.json) passes the official registry schema. It has not been submitted. Real-client results and publisher authentication remain required; see [submission checklist](docs/registry-submission.md).

## Support and security

- Product documentation: [gazebohq.com/docs/mcp](https://gazebohq.com/docs/mcp)
- Issues about public documentation or client interoperability: use this repository's issue tracker
- Security vulnerabilities: see [SECURITY.md](SECURITY.md)

## License

The documentation and examples in this repository are licensed under the [Apache License 2.0](LICENSE).
