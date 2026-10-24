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

Authorization is a required, secret, user-supplied **whole header value**.
No default or fixed token is included. Do not replace the placeholder with
any real credential. Downstream directories may not support this auth mode.

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
7. Fetch the registry record and compare the endpoint, version and secret-header
   declaration. Registry discovery is not security certification or a guarantee
   of inclusion in downstream marketplaces.

Sources:

- https://modelcontextprotocol.io/registry/quickstart
- https://modelcontextprotocol.io/registry/authentication
- https://modelcontextprotocol.io/registry/remote-servers
- https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json
