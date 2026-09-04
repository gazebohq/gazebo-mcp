# Security policy

## Supported scope

This public repository documents Gazebo's hosted MCP endpoint. It does not contain the production server implementation. Reports are welcome for:

- unsafe or misleading MCP configuration;
- authentication, credential-handling, audit, or transport claims that differ from production;
- repository or documentation changes that could redirect users or expose secrets;
- reproducible interoperability behavior with security impact.

For product vulnerabilities in the hosted Gazebo service, use the same private reporting channels below and do not create a public issue.

## Reporting a vulnerability

Use GitHub's **Report a vulnerability** function in this repository when available. You may alternatively report privately through Gazebo's contact channel at [gazebohq.com/contact](https://gazebohq.com/contact).

Include a concise description, affected behavior, reproduction steps, impact, and a safe way to contact you. Use synthetic or fully redacted data.

Do not disclose vulnerabilities, agent tokens, service credentials, authorization headers, or unredacted protocol traces in a public issue, discussion, pull request, or attachment.

## What to expect

Gazebo will acknowledge a valid private report, assess severity and scope, and coordinate remediation and disclosure when appropriate. Do not publicly disclose an unresolved report without coordination.

## Credential handling

The `get_credential` tool delivers a plaintext credential to the MCP client or agent. Clients and integrations must treat the value as ephemeral and must not persist it in files, environment variables, logs, prompts, transcripts, or long-lived memory.

If a credential or agent token is exposed, revoke or rotate it immediately in Gazebo before reporting the incident. Do not wait for triage before containing an active exposure.

See [THREAT_MODEL.md](THREAT_MODEL.md) for repository trust boundaries and required guarantees.
