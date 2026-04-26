# Security Policy

This repository contains runbooks and prompts for giving an AI coding agent limited DevOps authority over an OpenClaw deployment.

## Important safety notes

- Do not paste secrets, tokens, private IPs, or production config into issues or PRs.
- Do not grant broad shell access blindly.
- Review all Claude Code permission templates before use.
- Start with the conservative template and promote specific commands only after testing.
- Avoid automatic deletion of unknown data.
- Keep firewall, SSH, auth, token rotation, major updates, and rollbacks behind explicit human approval unless you have a mature site-specific policy.

## Reporting security issues

Please open a private security advisory if available, or contact the repository owner privately. Do not disclose secrets or exploitable deployment details in public issues.
