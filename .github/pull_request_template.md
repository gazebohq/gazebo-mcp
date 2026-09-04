## Summary

Describe the user-visible or security-contract change.

## Verification

- [ ] I verified the canonical endpoint remains `https://app.gazebohq.com/api/mcp`.
- [ ] Authentication guidance still requires a Gazebo agent bearer token.
- [ ] Transport guidance claims only stateless JSON Streamable HTTP over POST.
- [ ] Tool names, parameters, audit behavior, and credential handling match production.
- [ ] I included exact client versions and dates for client-specific configuration changes.
- [ ] I removed tokens, credentials, authorization headers, and unredacted logs or transcripts.
- [ ] I reviewed the security impact and updated `THREAT_MODEL.md` if a trust boundary changed.
