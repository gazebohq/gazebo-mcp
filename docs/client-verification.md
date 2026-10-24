# MCP client verification

Complete this matrix against current stable client releases before submitting Gazebo to public MCP directories. Record the exact client version and test date in the pull request or issue that performs the verification.

## Canonical connection details

- Endpoint: `https://app.gazebohq.com/api/mcp`
- Header: `Authorization: Bearer <agent-token>`
- Authentication: scoped agent bearer token, or MCP OAuth with explicit selected-key approval
- Transport: Stateless, JSON-only Streamable HTTP over `POST`

Never commit real tokens, credentials, response secrets, or unredacted transcripts.

## Test fixture

Use a dedicated test agent with:

- a recognizable non-production name;
- one connected test service;
- an access policy that permits at least `GET`;
- one method that is intentionally denied;
- a token that can be revoked after the test.

## Required checks

Run each check in Cursor, Claude Code, OpenAI Codex, Claude Desktop, and Windsurf. Track OpenAI Responses API separately as a cloud runtime; ChatGPT web is a separate hosted client; the Claude OAuth pass does not establish its compatibility.

| Check | Expected result |
|---|---|
| Initialize | Client connects successfully and negotiates a supported protocol version |
| Tool discovery | `get_identity`, `get_credential`, and `list_audit_events` are visible |
| `get_identity` | Returns the intended agent identity and expected policy without a credential |
| Allowed credential access | `get_credential` succeeds for the test service and allowed method |
| Default key selection | Manual agent tokens: omitting `key_name` selects one primary credential. OAuth: require an approved named key instead. |
| Named credential access | A valid `key_name` returns only that active credential |
| Denied method | Returns a structured denial and creates a denied audit event |
| Audit retrieval | `list_audit_events` finds the success and denial; cursor pagination works |
| Invalid agent token | Connection or request is rejected with `401` |
| Account API token | Rejected at the MCP boundary with `401` and JSON-RPC authentication error |
| Revoked agent token | Previously valid token is rejected after revocation |
| Notification | `notifications/initialized` is accepted without a response body |

## Evidence to retain

Retain only non-sensitive evidence:

- client name and exact version;
- operating system;
- test date;
- pass/fail result for each check;
- redacted error messages for failures;
- any client-specific configuration caveats.

Do not retain agent tokens, retrieved credentials, authorization headers, or unredacted client logs.

## Exit criteria for directory submissions

Directory submission can begin when:

- the current stable clients listed above complete identity and tool-discovery checks;
- credential access and audit retrieval pass in every client that supports remote HTTP servers with bearer headers;
- any unsupported client is documented accurately rather than worked around with an unverified adapter;
- repository instructions match the tested client releases;
- no listing copy claims GET, SSE, sessions, or server streaming.

## Current evidence

As of 25 October 2026, a direct OpenAI Responses API run and a user-performed Claude Desktop OAuth flow passed the checks listed below. Other desktop/CLI runs remain pending. Do not turn HTTP smoke tests or a single-client pass into universal compatibility.

| Client | Runtime result |
|---|---|
| Cursor | Pending |
| Claude Code | 2.1.310, Linux, 25 October 2026: authenticated MCP health passed; the same configuration no longer connected after token revocation. Tool execution not tested. |
| OpenAI Codex CLI / IDE | CLI 0.164.0 app-server, Linux, 25 October 2026: discovery, identity, named fictional-key retrieval, method denial, audit retrieval and revoked-token rejection passed. No model turn, IDE or OAuth test. |
| Claude Desktop remote connector | macOS, 25 October 2026: OAuth approval, three-tool discovery, identity, named credential retrieval, and authentication rejection after Gazebo-side grant revocation passed. Exact client version and audit-tool call not recorded; broader matrix incomplete. |
| Windsurf / current Cascade product | Pending; verify exact product/version because vendor docs redirect |
| OpenAI Responses API | Passed 25 October 2026 with gpt-5.4-mini: discovery, approval, identity, fictional-key access, policy denials, audit pagination and revoked-token failure; store:false |
| ChatGPT web / hosted plugins | Not runtime-tested. MCP OAuth is implemented, but compatibility must be tested independently. |

Use only fictional keys for verification. Cloud services and gateways can receive tool results; do not describe every desktop interface as local execution.

## Claude Desktop evidence and limits

Evidence is the maintainer’s screenshots from an actual production connection, not a mocked client. It used Sign in now, Claude’s published identity (CIMD), and Streamable HTTP. No expiry was selected. Claude reported successful named-key retrieval without displaying the value. After the grant was revoked in Gazebo, a fresh call displayed “Authentication required to use this tool” with a Connect action. The connector remained configured, and no reauthorization was approved during the check.

The existing smoke-test service was used; this review did not inspect the credential plaintext. No secret values or account identifiers are retained here. This run does not prove expiry, refresh rotation, audit retrieval or every negative-policy case in Claude.

## Automated native CLI evidence

Claude Code 2.1.310 and Codex CLI 0.164.0 were run in isolated temporary homes on Linux against the production endpoint using a newly created, expiring dedicated test agent. The existing test credential was compared to the expected fictional value before passing it to either client. No unrelated project secrets were inherited by the vendor processes.

Claude Code `mcp list` reported a connected authenticated server, then no connected server after revocation. This is a health check, not proof of credential tool execution in Claude Code.

Codex’s native app-server initialized an ephemeral thread and exposed all three Gazebo tools. Its `mcpServer/tool/call` interface successfully executed identity, named retrieval and audit retrieval, correctly returned a tool error for a disallowed DELETE method, and rejected fresh credential retrieval after the agent token was revoked. No model API request was needed; this proves the native client MCP path, not a model-driven task or Codex IDE behavior. OAuth and audit pagination were not tested in these CLI runs.

Temporary agents were removed and temporary local credential/configuration directories deleted after each attempt. No token, provider-key value or raw tool result was retained in this report.
