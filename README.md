# Gazebo MCP

Scoped credential access for AI agents via the Model Context Protocol, with auditable credential-access attempts and immediate revocation.

Gazebo hosts the MCP server. You do **not** need to install or run a local Gazebo server.

- **Endpoint:** `https://app.gazebohq.com/api/mcp`
- **Authentication:** `Authorization: Bearer <agent-token>`
- **Transport:** Stateless, JSON-only Streamable HTTP over `POST`
- **Create an agent token:** [app.gazebohq.com/agents](https://app.gazebohq.com/agents)
- **Documentation:** [gazebohq.com/docs/mcp](https://gazebohq.com/docs/mcp)

> Gazebo's MCP endpoint accepts scoped **agent bearer tokens only**. Account API tokens and browser sessions are rejected.

## Why Gazebo

Long-lived credentials should not be copied into every agent's configuration. Gazebo gives each agent an identity and a policy describing which services and HTTP methods it may access. The agent retrieves an approved credential only when it needs one, while credential-access attempts—including denials—are recorded in the audit trail.

The credential value is delivered to the MCP client or agent. Treat it as ephemeral: do not persist it in files, environment variables, logs, prompts, transcripts, or long-lived memory.

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

Retrieves an active credential for an allowed service and intended HTTP method. Every credential-access attempt is audited before a secret is released.

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `service` | string | Yes | Service slug, such as `github` or `stripe` |
| `method` | string | Yes | Intended method: `GET`, `POST`, `PUT`, `PATCH`, or `DELETE` |
| `key_name` | string | No | Named credential; omit it to return exactly one deterministic primary active credential |

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

Client configuration changes independently of Gazebo. Use the current client documentation for the exact configuration surface and field names. Every client needs the same two values:

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
        "Authorization": "Bearer <your-agent-token>"
      }
    }
  }
}
```

Do not commit a real agent token. Prefer the current client-supported secret or environment-variable mechanism where available.

### Claude Code and Claude Desktop

Claude Code and Claude Desktop have separate MCP setup surfaces. Configure a remote HTTP MCP server using the canonical endpoint and authorization header above; do not reuse one product's configuration instructions for the other. Follow Anthropic's current documentation for the installed release.

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

The endpoint is stateless, JSON-only Streamable HTTP over `POST`. Gazebo does not claim support for `GET`, SSE, sessions, or server streaming.

## Support and security

- Product documentation: [gazebohq.com/docs/mcp](https://gazebohq.com/docs/mcp)
- Issues about public documentation or client interoperability: use this repository's issue tracker
- Security vulnerabilities: see [SECURITY.md](SECURITY.md)

## License

The documentation and examples in this repository are licensed under the [Apache License 2.0](LICENSE).
