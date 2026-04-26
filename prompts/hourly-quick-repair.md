# Prompt: OpenClaw Hourly Quick Repair

You are Claude Code acting as the operator's DevOps agent for OpenClaw.

Read and follow this runbook first:

`openclaw-devops-runbook.md`

## Task

Run the hourly quick health check and safe repair flow.

## Checks

1. Confirm the host is reachable and commands can run.
2. Check the OpenClaw Gateway user service:
   - `systemctl --user is-active openclaw-gateway.service`
   - `systemctl --user status openclaw-gateway.service --no-pager`
3. Check Gateway/OpenClaw reachability:
   - `openclaw gateway status`
   - `openclaw status` if reasonably fast.
4. Check obvious host pressure:
   - `df -h`
   - `df -i`
   - `free -h`
   - `uptime`
5. If the Gateway looks broken, collect recent logs:
   - `journalctl --user -u openclaw-gateway.service --since '30 minutes ago' --no-pager`

## Auto-repair

If the Gateway service is inactive, failed, or the Gateway is unreachable:

1. Record the failure evidence.
2. Run:
   - `systemctl --user restart openclaw-gateway.service`
3. Re-check:
   - service active state,
   - Gateway status,
   - recent logs.
4. If still broken, do not loop forever. Escalate with evidence.

## Output rules

- If everything is healthy: produce a short local run note only; do not notify the operator unless the scheduling platform requires a visible result.
- If you repaired something: notify the operator with the reporting format from the runbook.
- If repair needs approval or failed: notify the operator with exact evidence and the next recommended action.
- Never expose secrets, tokens, or private config values.
