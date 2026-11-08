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

As of 8 November 2026, direct protocol and Responses API tests, native CLI checks, and user-performed ChatGPT Business web, normal Claude Chat, Claude Desktop, Claude Code desktop, Cursor Desktop and Devin-hosted OAuth runs provide the evidence below. The latest date is the latest recorded client run, not a retest of every client. Coverage differs by client: do not turn these results into universal compatibility or treat missing cases as passed.

| Client | Runtime result |
|---|---|
| Cursor Desktop | macOS, 25 October 2026: DCR-based OAuth approval, three-tool discovery, identity, reported named fictional-key retrieval and reported rejection of a fresh request after the Gazebo revocation step. Exact version, audit-tool use, renewal, timed expiry and web/cloud runtime not verified. |
| Claude Code — terminal/CLI | 2.1.310, Linux, 25 October 2026: authenticated MCP health passed; the same configuration no longer connected after token revocation. CLI tool execution not tested. |
| Claude Code — desktop Code session | macOS, 6 November 2026: OAuth approval, three-tool discovery, identity, reported named fictional-key GET retrieval, matching successful audit retrieval and fresh-request rejection after the Gazebo revocation step passed. Exact app version and raw final error unrecorded; terminal/CLI tool execution, renewal, timed expiry and marketplace-plugin installation unverified. |
| OpenAI Codex CLI / IDE | CLI 0.164.0 app-server, Linux, 25 October 2026: discovery, identity, named fictional-key retrieval, method denial, audit retrieval and revoked-token rejection passed. No model turn, IDE or OAuth test. |
| Claude Chat — normal Chat OAuth flow | 7 November 2026: approved fictional-key OAuth scope, identity, named GET retrieval, matching successful audit retrieval and fresh-request rejection after Gazebo-side grant revocation. Owner-supplied screenshots; exact client version and complete interface/platform matrix unrecorded. Renewal, timed expiry, policy denials and pagination not verified in this run. |
| Claude Desktop remote connector | macOS, 25 October 2026: OAuth approval, three-tool discovery, identity, named credential retrieval, and authentication rejection after Gazebo-side grant revocation passed. Exact client version and audit-tool call not recorded; broader matrix incomplete. |
| Devin hosted | 25 October 2026: Personal OAuth approval, three-tool discovery, identity, reported named fictional-key GET retrieval, audit retrieval showing the matching successful event, and reported fresh-request rejection requiring OAuth after the revocation step. Exact version, renewal, timed expiry and local desktop execution unverified. |
| Legacy Windsurf / Cascade | Not tested; hosted Devin evidence is not evidence for older clients. |
| OpenAI Responses API | Passed 25 October 2026 with gpt-5.4-mini: discovery, approval, identity, fictional-key access, policy denials, audit pagination and revoked-token failure; store:false |
| ChatGPT Business — web | 8 November 2026: OAuth connection, identity, reported named fictional-key GET retrieval and matching successful audit lookup. Following Gazebo-side revocation, a fresh retrieval prompted reconnection. Maintainer-supplied screenshots; exact client version, raw token exchange and final HTTP status unrecorded. Other plans, native mobile, refresh, timed expiry, policy-denial cases and pagination unverified. |

Use only fictional keys for verification. Cloud services and gateways can receive tool results; do not describe every desktop interface as local execution.

## ChatGPT Business web evidence and limits

On 8 November 2026, the maintainer used ChatGPT Business in a web browser and its **Plugins → + → Add custom MCP server** form. The canonical Gazebo endpoint was configured with OAuth, without manually supplied client credentials. Gazebo's consent screen approved only one named fictional service key and set a 24-hour access deadline. The other listed service key remained unchecked.

The conversation reported successful get_identity with the intended approved service/key and expiry. It then reported successful named get_credential retrieval using method GET and a matching list_audit_events result for that connection and service. The credential was not printed, but it necessarily reached OpenAI's runtime and host. Not printing a value does not guarantee retention behavior. No raw credential values, account identifiers or tokens are retained here.

Following the Gazebo revocation step, a fresh retrieval request without reconnecting displayed **Reconnect Gazebo**, with the generic message that the connection had expired. No new credential was shown. This is user-observed revoked-access behavior, not evidence that the selected 24-hour deadline elapsed or a capture of the exact HTTP status.

Evidence consists of maintainer-supplied production consent and conversation screenshots, not raw tool/token traces or independent backend inspection. The screenshots do not identify CIMD versus DCR negotiation. Exact client version, operating system, refresh rotation, timed expiry, unapproved-key/method-denial cases and audit pagination were not established. Other plans and native mobile clients remain unverified. The conversation reported no downstream provider requests; no independent network trace was captured.

Creating a custom connection is separate from public-directory submission. This private fictional-data test is not OpenAI approval, directory acceptance or approval to process real provider keys; directory eligibility remains unresolved. Revocation stops future retrieval, not use of keys already received.

## Claude Desktop evidence and limits

Evidence is the maintainer’s screenshots from an actual production connection, not a mocked client. It used Sign in now, Claude’s published identity (CIMD), and Streamable HTTP. No expiry was selected. Claude reported successful named-key retrieval without displaying the value. After the grant was revoked in Gazebo, a fresh call displayed “Authentication required to use this tool” with a Connect action. The connector remained configured, and no reauthorization was approved during the check.

The existing smoke-test service was used; this review did not inspect the credential plaintext. No secret values or account identifiers are retained here. This run does not prove expiry, refresh rotation, audit retrieval or every negative-policy case in Claude.

## Claude Chat OAuth evidence and limits

On 7 November 2026, the maintainer exercised the production remote connector in a normal Claude Chat conversation, separate from the earlier desktop Code-session test. Only a named fictional service key was approved through OAuth. Owner-supplied screenshots show get_identity returning the approved service/key scope, successful get_credential with the explicitly approved key_name and declared method GET, and list_audit_events reporting the matching successful retrieval. Each tool was exercised through Claude’s approval interface.

After the Gazebo grant was revoked, a fresh retrieval failed and Claude displayed a reconnect prompt. The maintainer did not reconnect. The written response did not print the fictional value, but the credential was delivered to the runtime and its host; asking not to display or save it does not guarantee host retention behavior.

This evidence establishes the observed client flow, not independent database/log inspection or a raw protocol trace. Exact client version and the full interface/platform matrix were not recorded. Token renewal, timed expiry, policy denials, audit pagination and other Claude interfaces remain outside this run. It does not verify Claude Code terminal/CLI tool execution or ChatGPT web. Revocation stops future retrieval, not use of a provider key already received; invalidate existing copies at the provider.

## Claude Code desktop evidence and limits

The maintainer used the **Code** tab in Claude's macOS desktop app, selected **This computer** and a separate empty test folder, then reconnected the existing Gazebo custom connector through OAuth. Only the fictional test service/key was approved, with no expiry. This is a Code-session test, distinct from normal Chat mode and the earlier terminal/CLI health check.

Connector settings displayed all three tools with **Needs approval**, while the session also displayed an Auto-mode notice. This does not establish that every call prompted for manual approval. get_identity returned the approved service/key metadata. get_credential with an explicitly named fictional key and method GET reported success without displaying the value. list_audit_events returned one successful GET event matching that key and the same connection, timestamped 2026-11-05 22:19:19 UTC (6 November in Melbourne).

After the Gazebo grant-revocation step, a fresh get_credential attempt visibly showed **Failed**. Claude reported that the connection had been invalidated and required reconnection, and stated it did not reconnect, sign in or approve new access.

Evidence consists of the maintainer's production UI screenshots and agent summaries, not raw protocol traces or independent inspection of the key. The credential necessarily reached the runtime; a prompt not to display or save it is not a technical retention guarantee. Exact app version, CLI tool execution, timed expiry, renewal, policy-denial cases, audit pagination and marketplace installation were not tested in this run. A local Code session does not establish local-only handling by the account's remote connector. The broader directory release gate is unchanged by this evidence update.

## Automated native CLI evidence

Claude Code 2.1.310 and Codex CLI 0.164.0 were run in isolated temporary homes on Linux against the production endpoint using a newly created, expiring dedicated test agent. The existing test credential was compared to the expected fictional value before passing it to either client. No unrelated project secrets were inherited by the vendor processes.

Claude Code `mcp list` reported a connected authenticated server, then no connected server after revocation. This is a health check, not proof of credential tool execution in Claude Code.

Codex’s native app-server initialized an ephemeral thread and exposed all three Gazebo tools. Its `mcpServer/tool/call` interface successfully executed identity, named retrieval and audit retrieval, correctly returned a tool error for a disallowed DELETE method, and rejected fresh credential retrieval after the agent token was revoked. No model API request was needed; this proves the native client MCP path, not a model-driven task or Codex IDE behavior. OAuth and audit pagination were not tested in these CLI runs.

Temporary agents were removed and temporary local credential/configuration directories deleted after each attempt. No token, provider-key value or raw tool result was retained in this report.

## Cursor Desktop evidence and limits

The maintainer used the URL-only remote configuration on macOS. The first attempt failed because Gazebo lacked dynamic client registration; after DCR was added and published, Authenticate opened Gazebo's selected-key approval and all three tools became available. The session returned identity and reported successful named retrieval of the fictional fixture. Cursor initially requested manual tool approval because an earlier identity-only instruction conflicted with retrieval; the maintainer approved the intended retrieval call. After the instruction to revoke the grant in Gazebo, the session reported a fresh request rejected, without reconnecting. Screenshots show client summaries, not the raw final server error or independent inspection of the key. No expiry was selected; the exact app version, audit retrieval and refresh were not recorded.

## Devin hosted evidence and limits

The maintainer configured HTTP, Personal OAuth, authorization code, Automatic execution from Devin's servers and the discovered resource. No client ID, client secret or authorization header was manually supplied. The approval used the existing fictional-key fixture. Web validation discovered all three tools but only displayed schemas. A hosted agent session returned scoped identity, reported successful named GET retrieval and called list_audit_events, returning the successful event under the same connection. After the Gazebo revocation instruction, a fresh request was reported rejected with OAuth required and no value returned; no new authorization was approved.

This evidence consists of production UI screenshots and agent summaries, not raw token exchanges or an independent inspection of the key. The connection had no expiry. Exact client version, timed expiry, refresh rotation, manual-token setup and local desktop execution were not tested. The initial session was blocked by free credits; later hosted calls executed successfully. Personal authentication is not a guarantee that other organization members cannot interact with a session.
