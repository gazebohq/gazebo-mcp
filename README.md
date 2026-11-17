# Gazebo MCP

Credential retrieval or proxied service requests for AI agents via the Model Context Protocol, with explicit permissions, audit metadata and revocable future access.

Gazebo hosts the MCP server. You do **not** need to install or run a local Gazebo server.

- **Endpoint:** `https://app.gazebohq.com/api/mcp`
- **Authentication:** MCP OAuth (CIMD or Dynamic Client Registration) with purpose-specific approval, or `Authorization: Bearer <agent-token>`
- **Transport:** Stateless, JSON-only Streamable HTTP over `POST`
- **Create an agent token:** [app.gazebohq.com/agents](https://app.gazebohq.com/agents)
- **Documentation:** [gazebohq.com/docs/mcp](https://gazebohq.com/docs/mcp)

> Gazebo's MCP endpoint accepts scoped agent bearer tokens and resource-bound MCP OAuth access tokens. Account API tokens and browser sessions are rejected.

## Why Gazebo

The credential-release model in this section describes **retrieval mode**.
For forwarding without returning the provider key, see [proxy access](#choose-retrieval-or-proxy).

Long-lived credentials should not be copied into every agent's configuration. Gazebo gives each agent an identity and a policy describing which service credentials it may retrieve and the declared HTTP methods permitted for retrieval. Provider key permissions, not Gazebo, govern subsequent API calls. The agent retrieves an approved credential only when it needs one, while credential-access attempts—including denials—are recorded in the audit trail.

The actual provider credential is delivered to the MCP client or agent runtime and can be accessed by its host. Cloud connectors, gateways and model providers may receive it and may retain tool output according to their settings. Treat it as ephemeral where possible and do not deliberately persist it. Revoking Gazebo access blocks future retrieval; rotate or revoke the provider key to invalidate copies already received.

## Choose retrieval or proxy

Proxy access requires a proxy-enabled Gazebo deployment. Proxy tools are not
advertised when proxy access is disabled.

Access mode is fixed when a manual agent is created. OAuth connections request
one purpose: `gazebo.credentials.read` for retrieval or
`gazebo.proxy.execute` for proxy access. Do not request both together.
Registration alone grants neither, and retrieval permission never authorizes
proxy execution.

- **Retrieval:** the existing credential tools release an approved provider key.
  The runtime and its host receive it. Provider permissions govern later use;
  revocation stops future retrieval, not use of copies already obtained.
- **Proxy:** `list_proxy_capabilities` returns approved versioned connections and
  APIs; `execute_proxy_request` forwards one permitted request without returning
  the provider key. Responses can still contain sensitive provider data.

For proxy access, select the intended account/connection, API and methods during
approval. Discover current bindings before composing a request, then preserve
its contract version, connection ID/revision, profile ID/version and response
mode. The [example request](examples/proxy-request.json) uses fictional bindings:
replace them with current discovery values before explicitly submitting it.
The Resend path is an example, not a Resend-only API.

Inspect gateway and provider status separately. There are no automatic retries,
silent permission substitutions or fallbacks to credential release. Method
filters are not semantic read-only guarantees: GET can mutate data, and GraphQL
can use POST for queries and mutations. Provider permissions still apply.
Revocation stops future requests, not ones already dispatched.

See the [shared contract and limits](https://gazebohq.com/docs/api/agent-access#proxy-requests)
and [explicit Cursor OAuth configuration](https://gazebohq.com/docs/mcp/cursor).
Proxy audit records contain metadata and outcomes, not payloads or authentication
headers. Authentication rejection can happen before a proxy request ID exists.

## Available tools

### Tool metadata and permissions

| Tool | Display title | Read-only | Destructive | Idempotent | Open-world |
|---|---|---|---|---|---|
| `get_identity` | View connection identity and access | Yes | No | Yes | No |
| `get_credential` | Retrieve a service credential | No | No | No | No |
| `list_audit_events` | List credential-access events | Yes | No | Yes | No |
| `list_proxy_capabilities` | Discover permitted proxy requests | Yes | No | Yes | No |
| `execute_proxy_request` | Make an approved proxy request | No | Yes | No | Yes |

Tools are mode-specific, not all exposed together. Retrieval exposes `get_identity`, `get_credential` and `list_audit_events`; proxy exposes `get_identity` and the two proxy tools. Credential retrieval is not marked read-only because it audits attempts and releases sensitive values, but does not modify provider data. Proxy execution is destructive/open-world because permitted requests can change or delete external provider data; discovery is metadata only. MCP hints do not grant permission or guarantee client confirmation. Clients may cache tool metadata and need a refresh after a hosted deployment.

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

Retrieves an active credential for an allowed service and intended HTTP method. The method argument is checked against Gazebo's release policy; this tool does not execute that HTTP method or any provider request. Provider permissions govern subsequent use of the returned key. Valid credential-retrieval attempts are audited before a secret is released. Authentication and malformed-request failures are not credential-access events. Audit persistence failure blocks release.

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `service` | string | Yes | Service slug, such as `github` or `stripe` |
| `method` | string | Yes | Intended method: `GET`, `POST`, `PUT`, `PATCH`, or `DELETE` |
| `key_name` | string | For OAuth; optional for manual agent tokens | OAuth requires an explicitly approved named key. Manual agent tokens may omit it to select one deterministic primary active credential. |

```text
get_credential({ service: "github", key_name: "token", method: "GET" })
→ "<actual-stored-provider-key>"
```

Do not retry a denied request until the user changes the policy or you choose an allowed method.

### `list_audit_events`

Returns recent credential-access attempts for the authenticated agent, not subsequent provider API activity. No credential values are returned.

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `limit` | integer | No | Number of events, from 1 through 50 |
| `service` | string | No | Filter by service |
| `cursor` | string | No | Opaque pagination cursor |

## Recommended retrieval flow

1. Call `get_identity` to verify the authenticated agent and policy.
2. Call `get_credential` with the service, approved key name and actual intended HTTP method. OAuth requires the approved key name.
3. Use the returned credential for the immediate operation, then discard it.
4. Optionally call `list_audit_events` to verify the access event.

## Client configuration

The existing client summary below describes **retrieval** coverage. The
separate Cursor proxy OAuth result and its limits are recorded in the
[compatibility guide](docs/client-verification.md#proxy-oauth-evidence).

**Tested retrieval paths include Cursor Desktop, normal Claude Chat, Claude Desktop, ChatGPT Business on web, Claude Code’s desktop Code session, Devin hosted, Codex CLI and the OpenAI Responses API.** The latest recorded retrieval test is the ChatGPT Business web OAuth flow on 8 November 2026; coverage differs by runtime. Cursor proxy OAuth testing on 16 November is recorded separately above. Other ChatGPT plans and token refresh, Claude Code terminal/CLI tool execution and legacy Windsurf/Cascade remain unverified. The ChatGPT fictional-data test does not establish public-directory eligibility. See the [compatibility guide](docs/client-verification.md) for tested authentication methods, environments and detailed results.

Client configuration changes independently of Gazebo. Use the current client documentation for the exact configuration surface and field names. For clients using manual agent tokens, use:

```text
URL: https://app.gazebohq.com/api/mcp
Authorization: Bearer <your-agent-token>
```

### Cursor

For explicit retrieval on Cursor desktop, configure project `.cursor/mcp.json` or global `~/.cursor/mcp.json`:

```json
{"mcpServers":{"gazebo":{"url":"https://app.gazebohq.com/api/mcp","auth":{"CLIENT_ID":"YOUR_PUBLIC_CLIENT_ID","scopes":["gazebo.credentials.read"]}}}}
```

Use the [Cursor guide](https://gazebohq.com/docs/mcp/cursor) to register a public client and copy its returned client ID; no client secret is required. Use Cursor's Authenticate/connect action, select only the keys to share, choose an access deadline and approve. OAuth retrieval must name the approved `key_name`. For proxy instead, use a separate entry with `auth.scopes: ["gazebo.proxy.execute"]` and approve connection/API/method permissions, not raw-key release. Desktop and cloud/web callbacks and sign-ins differ. Dynamic registration remains supported for clients requesting one purpose correctly.

For the manual agent-token alternative:

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

Set the variable in Codex's inherited environment. See [OpenAI setup](docs/openai.md) for configuration, identity-only Responses API guidance and ChatGPT Business web OAuth setup. Codex CLI 0.164.0 was verified on Linux through its native app-server MCP interface: discovery, identity, fictional-key retrieval, method denial, audit retrieval and revoked-token rejection. This was a programmatic client run, not a model-driven task or IDE test. A separate direct OpenAI Responses API test passed with fictional credentials; see the production verification report.

### ChatGPT Business — web

In ChatGPT web, open **Plugins → + → Add custom MCP server**, name it **Gazebo**, enter `https://app.gazebohq.com/api/mcp` and select **OAuth**. Leave advanced OAuth settings at their discovered defaults initially; no manual client secret was needed in the recorded test. Review the risk notice, create the connection and continue to Gazebo to approve only the intended service keys. Start with a nonfunctional fictional key and a bounded access deadline.

The 8 November 2026 user-run test covered identity, named fictional-key GET retrieval and matching audit lookup. After Gazebo-side revocation, a fresh retrieval prompted reconnection. Other plans, native mobile, refresh and timed expiry remain unverified; this is not directory acceptance or approval to process real provider keys. See [ChatGPT setup](docs/openai.md#chatgpt-business-on-web) and [verification evidence](docs/client-verification.md#chatgpt-business-web-evidence-and-limits).

### Claude Desktop and normal Chat

In Settings → Connectors, add a custom connector:

- URL: `https://app.gazebohq.com/api/mcp`
- Authentication: **Sign in now**
- OAuth client: **Use Claude’s published identity**
- Transport: **Streamable HTTP**

Click Add, then Connect. Sign into Gazebo, select only the keys to share, choose an expiry and approve. In Claude, call Get identity first. Use fictional credentials for your first retrieval test. Revoke the grant in Gazebo to block further retrieval. A key already returned remains accessible to the runtime/host until invalidated at the provider.

Normal Chat OAuth was separately verified on 7 November 2026: identity, approved fictional-key retrieval, matching audit retrieval and rejection after grant revocation. This does not establish every Claude interface, platform or version. See the [Chat verification evidence](docs/client-verification.md#claude-chat-oauth-evidence-and-limits).

### Claude Code

In Claude's **Code** tab, start a local session in a separate test folder and open **+ → Connectors**. Add or reconnect Gazebo using the canonical endpoint, sign in with OAuth and approve only the service keys the connection needs. Start with get_identity; OAuth retrieval must name the approved key_name. This desktop connector has passed identity, fictional-key retrieval, matching audit retrieval and rejection after revocation. See the [compatibility guide](docs/client-verification.md#claude-code-desktop-evidence-and-limits) for the tested environment and limits.

Terminal configuration is separate: configure remote HTTP with the canonical endpoint and agent authorization header above, following Anthropic's current documentation. The earlier Linux CLI run covered connection health only, not tool execution. Desktop OAuth results do not verify CLI behavior or local-only credential handling.

### Devin hosted and legacy Windsurf

In Devin's custom MCP settings use HTTP, the canonical endpoint, OAuth and Personal access. Leave execution at Automatic (Devin servers), grant type Authorization code, client ID/secret blank, scope `gazebo.credentials.read offline_access`, resource Default (discovered), and headers empty. Leave the supplied callback URL unchanged. Install and connect, then approve selected keys in Gazebo. Personal authentication does not guarantee session privacy; other organization members may interact with sessions.

The web **Test tools** panel validates the server and displays discovered schemas; the tested panel does not execute calls. Use **Use MCP** to start a hosted session, which needs available agent usage quota. Web and Desktop settings differ. Start with `get_identity`, then use a fictional key for named retrieval and verify the audit event. Revoke the grant in Gazebo and make a fresh request without reconnecting.

This hosted path passed the checks described in the verification matrix. It does not verify local desktop execution or legacy Windsurf/Cascade. Legacy clients may support manual bearer headers, but follow documentation for the installed product/version instead of assuming a configuration path.

## Verification checklist

### Credential retrieval

After configuring a retrieval connection:

1. Confirm initialization and discovery list `get_identity`, `get_credential` and `list_audit_events`.
2. Call `get_identity` and verify the expected agent name and policy.
3. Call `get_credential` for an allowed service and method.
4. Call `list_audit_events` and verify the credential-access event is present.
5. Confirm an account API token, invalid token, and revoked agent token are rejected.

### Proxy access

Use a proxy-enabled deployment and a separately approved proxy connection:

1. Confirm discovery lists `get_identity`, `list_proxy_capabilities` and `execute_proxy_request`, not the credential-retrieval tools.
2. Call `get_identity` to check the intended agent, then `list_proxy_capabilities` for its current connection/profile references and permitted methods.
3. Explicitly execute one known low-impact request using those exact references and a permitted method. An HTTP method alone does not establish that an operation is harmless.
4. Check the gateway/tool result and provider status separately, and match the returned `request_id` in Gazebo Logs.
5. Revoke the specific test grant or agent, then confirm a fresh attempt is rejected without reconnecting or approving access again. Authentication rejection can occur before a proxy request ID exists.

See [docs/client-verification.md](docs/client-verification.md) for the release-test matrix.

## Protocol behavior

Gazebo supports MCP protocol versions `2024-11-05`, `2025-03-26`, and `2025-06-18`. Future version requests negotiate to `2025-06-18`.

The endpoint is stateless, JSON-only Streamable HTTP over `POST`. Accepted notifications return HTTP 202 with no body. Unauthenticated GET requests return HTTP 401 with OAuth discovery information. Authenticated GET and DELETE requests are not supported MCP operations. Unsupported explicit MCP-Protocol-Version headers return HTTP 400. Native clients may omit Origin; untrusted explicit Origins return HTTP 403. Responses use Cache-Control: no-store. Gazebo does not claim SSE, sessions, or server streaming.

Credential denials and missing keys use HTTP 200 with result.isError: true, not REST-style 403/404. Inspect the tool result as well as HTTP status.

## Registry status

The remote-only [server.json](server.json) passes the official Registry schema. Publication is pending; see the [submission checklist](docs/registry-submission.md). Protocol results and client-by-client evidence live in the [compatibility guide](docs/client-verification.md).

## Support and security

- Product documentation: [gazebohq.com/docs/mcp](https://gazebohq.com/docs/mcp)
- Issues about public documentation or client interoperability: use this repository's issue tracker
- Security vulnerabilities: see [SECURITY.md](SECURITY.md)

## License

The documentation and examples in this repository are licensed under the [Apache License 2.0](LICENSE).
