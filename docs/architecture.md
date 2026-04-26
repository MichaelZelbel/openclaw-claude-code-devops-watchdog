# Architecture

The pattern is intentionally simple:

1. Claude Code runs independently from OpenClaw.
2. A scheduled Claude Code task reads this repository's runbook and prompt.
3. It checks the OpenClaw host, Gateway service, browser/CDP health, and update status.
4. It performs narrowly scoped safe repairs.
5. It escalates risky changes to a human operator.

This avoids relying on OpenClaw's own scheduler to repair OpenClaw when the Gateway itself is down.
