# Security Policy

AgentAudit is a hackathon project that reads AWS audit events. It is designed to be read-only and to redact account identifiers in everything it shows publicly.

## Reporting a vulnerability

Please use GitHub's private vulnerability reporting (the "Report a vulnerability" button on this repository's Security tab) rather than opening a public issue.

## Scope

- Anything that lets a visitor to the public site modify data or an AWS account
- Anything that exposes unredacted account IDs, ARNs, IP addresses, or credentials
- Secrets accidentally committed to this repository