# MCP client verification

Complete this matrix against current stable client releases before submitting Gazebo to public MCP directories. Record the exact client version and test date in the pull request or issue that performs the verification.

## Canonical connection details

- Endpoint: `https://app.gazebohq.com/api/mcp`
- Header: `Authorization: Bearer <agent-token>`
- Authentication: Gazebo agent bearer token only
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

Run each check in Cursor, Claude Code, OpenAI Codex, Claude Desktop, and Windsurf. Track OpenAI Responses API separately as a cloud runtime; ChatGPT web authenticated plugins currently require OAuth and are not covered by Gazebo's bearer-token setup.

| Check | Expected result |
|---|---|
| Initialize | Client connects successfully and negotiates a supported protocol version |
| Tool discovery | `get_identity`, `get_credential`, and `list_audit_events` are visible |
| `get_identity` | Returns the intended agent identity and expected policy without a credential |
| Allowed credential access | `get_credential` succeeds for the test service and allowed method |
| Default key selection | Omitting `key_name` returns one deterministic primary active credential |
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

As of 25 October 2026, vendor documentation is reviewed, but no actual vendor-client run is recorded. Do not turn production HTTP smoke-test results into client passes.

| Client | Runtime result |
|---|---|
| Cursor | Pending |
| Claude Code | Pending |
| OpenAI Codex CLI / IDE | Pending |
| Claude Desktop remote connector | Pending; verify account controls and cloud hosting |
| Windsurf / current Cascade product | Pending; verify exact product/version because vendor docs redirect |
| OpenAI Responses API | Pending; separate API account and cloud-runtime test |
| ChatGPT web / hosted plugins | Not supported by current setup; OAuth is not implemented |

Use only fictional keys for verification. Cloud services and gateways can receive tool results; do not describe every desktop interface as local execution.
