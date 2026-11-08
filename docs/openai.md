# OpenAI MCP setup

Connect ChatGPT Business on web to Gazebo through OAuth, or use scoped agent tokens with Codex and the OpenAI Responses API. These are separate clients with different setup and authentication requirements.

On 25 October 2026, Gazebo passed a direct OpenAI Responses API test with `gpt-5.4-mini`: tool discovery, approval, identity, fictional-key retrieval, denials, audit pagination and revoked-token rejection. That check does not establish Codex or ChatGPT compatibility. A separate native Codex CLI 0.164.0 app-server run on Linux passed discovery, identity, fictional-key retrieval, method denial, audit retrieval and revoked-token rejection. It did not exercise a model turn, OAuth or an IDE. A separate ChatGPT Business web OAuth test on 8 November 2026 covered identity, fictional-key retrieval, matching audit lookup and a reconnect requirement after revocation. Other plans and token refresh remain unverified.

## Codex CLI and IDE

Create a dedicated agent at [Gazebo Agents](https://app.gazebohq.com/agents). Set `GAZEBO_AGENT_TOKEN` in the environment inherited by Codex, then add this to `~/.codex/config.toml` or a trusted project's `.codex/config.toml`:

```toml
[mcp_servers.gazebo]
url = "https://app.gazebohq.com/api/mcp"
bearer_token_env_var = "GAZEBO_AGENT_TOKEN"
```

The setting contains the environment variable's **name**, not its value. Do not put a real token in a committed file. Restart the client after changing its environment.

Run `codex mcp list` to check the configuration and use `/mcp` in the Codex terminal interface to inspect the connection. Ask it to call `get_identity` first, then verify that `get_credential` and `list_audit_events` are available.

This guide and the verified Codex CLI run use a pre-issued agent token. `codex mcp login gazebo` is a separate OAuth flow: Gazebo implements discovery and public CIMD/DCR clients, but Codex OAuth has not been verified. Do not substitute it for this tested manual-token setup without testing. See [OpenAI's current MCP configuration guide](https://developers.openai.com/codex/mcp).

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

## ChatGPT Business on web

**User-tested on 8 November 2026:** OAuth connection, `get_identity`, named fictional-key `get_credential` retrieval and matching `list_audit_events` lookup succeeded. After Gazebo-side revocation, a fresh retrieval prompted reconnection.

### Setup

Use ChatGPT in a web browser and a Business workspace where you have permission to add custom MCP servers. Account controls and labels can change; see [OpenAI's custom MCP guide](https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt).

1. Open **Plugins → + → Add custom MCP server**.
2. Enter **Gazebo** as the name. An optional description is “Retrieve approved service credentials for your AI agents and review credential-access history.”
3. Choose **Server URL**, enter `https://app.gazebohq.com/api/mcp`, and select **OAuth**.
4. Leave advanced OAuth settings at their discovered defaults initially. Do not paste a Gazebo agent token into a client-secret field or select unauthenticated access. Gazebo supports public-client registration through CIMD or DCR and S256 PKCE; no manual client secret was needed for the recorded test.
5. Review the risk notice and select **Create as a plugin**. Continue to Gazebo when prompted, confirm the `app.gazebohq.com` destination and sign in.
6. For an initial test, use a dedicated fictional-data account. Select only its nonfunctional test key, choose an access deadline and approve. The recorded run selected one fictional key and a 24-hour expiry.
7. Return to a ChatGPT conversation with Gazebo enabled. Call `get_identity` first, then retrieve the explicitly approved `key_name` and check its matching audit event. To test revocation, revoke only this connection in Gazebo and request a fresh retrieval without reconnecting.

These instructions are separate from Codex's TOML and the Responses API bearer-token example above. See [OpenAI's authentication guide](https://developers.openai.com/plugins/build/auth) for current OAuth requirements.

### Test scope and trust boundary

The recorded test used ChatGPT Business on web with OAuth and an inert fictional credential. The maintainer supplied consent and conversation screenshots, not raw token exchanges or an independently inspected backend trace. ChatGPT's generic “expired” reconnect notice followed the revocation step; it was not a test of the selected 24-hour expiry elapsing.

Other plans, native mobile clients, token refresh, timed expiry, policy denials and audit pagination remain unverified in this ChatGPT run. See the [client verification record](client-verification.md#chatgpt-business-web-evidence-and-limits) for evidence and limitations.

OpenAI's runtime and host receive retrieved credentials even when the response does not print them. Revoking Gazebo access stops future retrieval, not use of a key already received. Creating a custom connection is separate from a public-directory submission. This private fictional-data test is not OpenAI endorsement, directory acceptance or approval to process real provider keys; directory eligibility remains unresolved under OpenAI's restricted-data rules. Review the [plugin guidelines](https://developers.openai.com/plugins/plugin-guidelines) and applicable terms.

## Safe verification

Use a dedicated test agent with a fictional service key. Check identity, allowed retrieval, denied retrieval, audit events and revocation before using real credentials.

The requested HTTP method governs credential retrieval, not subsequent provider calls. Provider key permissions govern those calls. Revoking Gazebo access prevents future retrieval; it does not invalidate a key already returned.
