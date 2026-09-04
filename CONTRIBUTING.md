# Contributing

Thanks for helping improve Gazebo MCP interoperability and documentation.

This repository documents Gazebo's hosted MCP endpoint; it is not the source repository for the hosted service. Good contributions include:

- corrections for current MCP client configuration;
- reproducible client interoperability reports;
- documentation fixes and clarifications;
- redacted protocol traces that demonstrate a standards issue without exposing secrets.

## Before opening a pull request

1. Confirm the behavior against the current stable client release.
2. Include the client name, exact version, operating system, and test date.
3. Remove all tokens, credentials, authorization headers, and sensitive output.
4. Keep these canonical facts unchanged unless Gazebo production behavior has changed:
   - endpoint: `https://app.gazebohq.com/api/mcp`;
   - agent bearer tokens only;
   - stateless JSON-only Streamable HTTP over `POST`;
   - no claimed GET, SSE, session, or server-streaming support.

For vulnerabilities or credential exposure, do not open a public issue. Follow [SECURITY.md](SECURITY.md).
