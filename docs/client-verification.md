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

Run each check in Cursor, Claude Code, Claude Desktop, and Windsurf.

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

- all four current stable clients complete identity and tool-discovery checks;
- credential access and audit retrieval pass in every client that supports remote HTTP servers with bearer headers;
- any unsupported client is documented accurately rather than worked around with an unverified adapter;
- repository instructions match the tested client releases;
- no listing copy claims GET, SSE, sessions, or server streaming.
