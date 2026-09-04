# Security policy

## Reporting a vulnerability

Do not disclose vulnerabilities, agent tokens, service credentials, authorization headers, or unredacted protocol traces in a public issue.

Report security concerns privately through Gazebo's contact channel at [gazebohq.com/contact](https://gazebohq.com/contact). Include a concise description, reproduction steps, and impact. Use synthetic or fully redacted data.

## Credential handling

The `get_credential` tool delivers a plaintext credential to the MCP client or agent. Clients and integrations must treat the value as ephemeral and must not persist it in files, environment variables, logs, prompts, transcripts, or long-lived memory.

If a credential or agent token is exposed, revoke or rotate it immediately in Gazebo before reporting the incident.
