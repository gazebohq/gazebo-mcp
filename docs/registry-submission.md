# Official MCP Registry entry — prepared, not submitted

`../server.json` describes the existing hosted service. There is no local MCP
package to install, so `packages` is deliberately omitted. The SDK and CLI
are not MCP server packages.

The proposed name is `io.github.gazebohq/gazebo`. Confirm that the maintainer
can authenticate the `gazebohq` GitHub organization namespace using the
registry publisher's supported login before submission. A connected Replit
GitHub integration is not a registry publisher login.

The version matches the current MCP initialization `serverInfo.version`.
Do not reuse an already-published version for changed registry metadata;
check registry versioning rules before future updates.

The optional `repository` field is intentionally omitted: the public
`gazebo-mcp` repository contains integration documentation, not the hosted
server's source code. It remains linked from the product documentation.

OAuth-capable clients discover authentication from the endpoint's HTTP 401
challenge and authorization-server metadata. No manually supplied header is
required by this listing. This does not make the endpoint unauthenticated:
Gazebo still requires a valid MCP OAuth access token or scoped agent token.
Manual bearer-token setup remains documented separately for clients that need it.

## Readiness update — 25 October 2026

- The registry search returned no existing `io.github.gazebohq/gazebo` entry.
  This is an availability check, not a name reservation.
- Claude Desktop on macOS completed OAuth approval, three-tool discovery,
  identity lookup, named credential retrieval and post-revocation rejection.
  The exact Claude version and audit-tool call remain unrecorded.
- Cursor Desktop completed DCR OAuth, identity, named test-key retrieval and reported fresh-request rejection after the revocation step.
- Devin hosted completed Personal OAuth, discovery and all three tools, including the matching successful audit event; a fresh request then required OAuth after the revocation step.
- Codex CLI 0.164.0 native app-server tool execution passed separately with a manual token; Claude Code 2.1.310 has connection-health evidence only.
- The original all-client release gate remains unresolved: Claude Code tool execution, legacy Windsurf/Cascade and some per-client cases are unverified. Submitting based only on verified paths requires an explicit maintainer decision, not a silent gate change.
- Publisher GitHub authentication and the final publish have not happened.

## Submission gate

1. Record authenticated production protocol results in `production-verification.md`.
2. Complete the real-client checks, including Codex, in the public repository's
   client verification matrix. Source reviews are not client runs.
3. Merge the reviewed public repository documentation changes.
4. Recheck the current registry schema, namespace ownership and name availability.
5. Use the official publisher's interactive GitHub authentication as the owner.
   Keep any publisher credentials out of this repository and chat.
6. Validate with the current official publisher, then publish this exact reviewed
   metadata only once the gate is satisfied.
7. Fetch the registry record and compare the endpoint, version and absence of
   a mandatory manual header. Registry discovery is not security certification or a guarantee
   of inclusion in downstream marketplaces.

Sources:

- https://modelcontextprotocol.io/registry/quickstart
- https://modelcontextprotocol.io/registry/authentication
- https://modelcontextprotocol.io/registry/remote-servers
- https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json
