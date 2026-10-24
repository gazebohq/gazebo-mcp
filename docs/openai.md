# OpenAI MCP setup

Connect Gazebo to OpenAI Codex using remote HTTP MCP and an agent bearer token. OpenAI's Responses API and ChatGPT web have separate setup and authentication requirements.

On 25 October 2026, Gazebo passed a direct OpenAI Responses API test with `gpt-5.4-mini`: tool discovery, approval, identity, fictional-key retrieval, denials, audit pagination and revoked-token rejection. Codex CLI, IDE and ChatGPT web were not runtime-tested by that check.

## Codex CLI and IDE

Create a dedicated agent at [Gazebo Agents](https://app.gazebohq.com/agents). Set `GAZEBO_AGENT_TOKEN` in the environment inherited by Codex, then add this to `~/.codex/config.toml` or a trusted project's `.codex/config.toml`:

```toml
[mcp_servers.gazebo]
url = "https://app.gazebohq.com/api/mcp"
bearer_token_env_var = "GAZEBO_AGENT_TOKEN"
```

The setting contains the environment variable's **name**, not its value. Do not put a real token in a committed file. Restart the client after changing its environment.

Run `codex mcp list` to check the configuration and use `/mcp` in the Codex terminal interface to inspect the connection. Ask it to call `get_identity` first, then verify that `get_credential` and `list_audit_events` are available.

Gazebo uses a pre-issued agent token. `codex mcp login gazebo` is an OAuth flow and is not the setup path for this endpoint. See [OpenAI's current MCP configuration guide](https://developers.openai.com/codex/mcp).

## OpenAI Responses API

The Responses API supports remote Streamable HTTP MCP with an `authorization` value. This is a separate cloud-hosted integration, not proof that the Codex client works.

For an initial identity-only check, configure the following tool object in your server-side Responses API request:

```js
{
  type: "mcp",
  server_label: "gazebo",
  server_url: "https://app.gazebohq.com/api/mcp",
  authorization: process.env.GAZEBO_AGENT_TOKEN,
  allowed_tools: ["get_identity"],
  require_approval: "always"
}
```

Supply the raw agent token in `authorization`; do not prepend `Bearer` there. The API constructs the bearer header. Your OpenAI API key authenticates the Responses API request and is separate from the Gazebo agent token.

Choose a model that supports remote MCP. Review any `mcp_approval_request` before sending an approval response. An approval request alone is not a successful tool call: confirm the resulting `mcp_call` has no error and returns the intended identity. See [OpenAI's MCP tool guide](https://platform.openai.com/docs/guides/tools-connectors-mcp).

Only enable `get_credential` after you understand the trust boundary. OpenAI's infrastructure receives tool results, including actual provider keys if credential retrieval is enabled; the key may also enter model context and response history. Begin with a fictional key, never a production credential.

## ChatGPT web and plugins

Do not paste Codex's TOML into ChatGPT web or assume that a ChatGPT subscription provides the same MCP configuration controls. OpenAI's current hosted plugin documentation expects OAuth 2.1 for authenticated MCP servers.

Gazebo's hosted MCP endpoint currently accepts pre-issued agent bearer tokens and does not implement MCP OAuth discovery. Treat authenticated ChatGPT web/plugin integration as **not verified and not supported by these instructions**. Account controls and product versions can change; see [OpenAI's authentication guide](https://developers.openai.com/plugins/build/auth).

## Safe verification

Use a dedicated test agent with a fictional service key. Check identity, allowed retrieval, denied retrieval, audit events and revocation before using real credentials.

The requested HTTP method governs credential retrieval, not subsequent provider calls. Provider key permissions govern those calls. Revoking Gazebo access prevents future retrieval; it does not invalidate a key already returned.
