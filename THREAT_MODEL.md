# Threat model

## Scope

This repository documents interoperability with Gazebo's hosted MCP endpoint at `https://app.gazebohq.com/api/mcp`. It contains no hosted server source, package, executable client, dependency manifest, or production credential.

The main risks are misleading security claims, unsafe client configuration, accidental disclosure during interoperability testing, unauthorized documentation changes, and supply-chain risk from repository automation.

## Assets

- **Agent bearer tokens** authenticate an agent to Gazebo and must never enter this repository, its issues, pull requests, Actions logs, or artifacts.
- **Retrieved service credentials** are plaintext values delivered to an MCP client or agent. They must remain ephemeral and must never be committed or attached to reports.
- **Security contract** includes the canonical endpoint, agent-only authentication boundary, tool behavior, transport limitations, audit semantics, and credential-handling guidance.
- **Repository integrity** matters because clients and directory reviewers may treat this repository as authoritative setup guidance.

## Trust boundaries

- **Contributor to repository:** Pull requests, issues, attachments, and suggested configurations are untrusted until reviewed.
- **Repository to MCP client:** Users may copy configuration from this repository into software that can expose values through logs, transcripts, prompts, or memory.
- **MCP client to Gazebo:** The client presents an agent bearer token and receives tool results, including plaintext credentials from `get_credential`.
- **Gazebo to third-party directory:** Directory copy may drift from production behavior or omit security caveats.
- **GitHub automation to repository:** Any future workflow or third-party Action would gain a supply-chain position and must use minimal permissions and immutable references.

## Threats and required guarantees

### Spoofing

A malicious endpoint or copied configuration could impersonate Gazebo and collect agent tokens.

- Documentation MUST use exactly `https://app.gazebohq.com/api/mcp`.
- Examples MUST use HTTPS and MUST NOT introduce mirrors, proxies, alternate hosts, or self-hosted server claims.
- Authentication guidance MUST say Gazebo agent bearer tokens only.

### Tampering

An unauthorized or insufficiently reviewed change could weaken warnings, change the endpoint, or misstate tool behavior.

- The default branch MUST reject force pushes and deletion.
- Changes MUST arrive through pull requests with resolved review conversations.
- Security-sensitive paths MUST have declared ownership.
- A second trusted maintainer SHOULD be added before requiring approving reviews; until then, the repository must not claim two-person review enforcement.

### Repudiation

Public guidance or listing copy could change without a clear review trail.

- Changes MUST retain Git commit and pull-request history.
- Client-verification evidence MUST record exact version, operating system, date, and redacted result.
- Directory claims MUST trace back to the canonical repository and current production behavior.

### Information disclosure

Tokens, credentials, headers, or unredacted client output could be committed or posted publicly.

- Secret scanning and push protection MUST remain enabled.
- Tests MUST use dedicated agents and non-production credentials.
- Issues and pull requests MUST prohibit tokens, credentials, authorization headers, and unredacted logs or transcripts.
- Suspected exposure MUST trigger immediate revocation or rotation before investigation continues.
- Security reports MUST use private vulnerability reporting or Gazebo's private contact channel.

### Denial of service

This documentation repository does not execute requests against Gazebo. Automated client tests added in the future could create uncontrolled request volume.

- The repository MUST NOT contain unattended tests using production credentials.
- Any future automated interoperability check MUST use bounded requests, dedicated test identities, and explicit rate limits.

### Elevation of privilege

A workflow, Action, or compromised maintainer could gain write access or publish malicious guidance.

- GitHub Actions MUST remain disabled while the repository has no required automation.
- If Actions are introduced, workflows MUST use minimal explicit permissions, immutable commit-SHA pins for third-party Actions, no pull-request secret exposure, and protected environments for privileged operations.
- Administrative access MUST remain limited to trusted Gazebo organization maintainers with strong account security.

## Release gate

Gazebo must not be submitted to MCP directories until current stable Cursor, Claude Code, Claude Desktop, and Windsurf releases have been tested according to `docs/client-verification.md`. Unsupported client behavior must be documented rather than hidden behind an unverified adapter.
