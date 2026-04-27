# Agent Start: OpenClaw Watchdog

Use this file as the starting point when asking Claude Code to configure itself as a watchdog for an OpenClaw Gateway.

## Goal

Configure Claude Code as a safe DevOps watchdog for the OpenClaw Gateway on this server.

Claude Code should periodically check whether OpenClaw is healthy, perform only clearly safe repairs, and escalate risky changes to the operator.

## First steps

1. Read these files in this repository:
   - `README.md`
   - `CLAUDE.md`
   - `openclaw-devops-runbook.md`
   - `prompts/hourly-quick-repair.md`

2. Confirm where Claude Code is running:
   - directly on the OpenClaw VPS, or
   - on another trusted machine that can SSH into the VPS.

3. Run a manual dry run of the hourly quick repair workflow.

## Safety boundaries

Do not do any of these without explicit operator approval:

- Change firewall, SSH, Tailscale, public exposure, or auth settings.
- Rotate or reveal secrets/tokens.
- Delete data.
- Install/remove OS packages.
- Reboot the server.
- Update OpenClaw.
- Change OpenClaw config.

Read-only checks are allowed.

If the OpenClaw Gateway is clearly down, Claude Code may restart only this user service once:

```bash
systemctl --user restart openclaw-gateway.service
```

After restarting, verify the service and report what happened.

## Manual dry-run checks

Run these checks first:

```bash
systemctl --user is-active openclaw-gateway.service
systemctl --user status openclaw-gateway.service --no-pager
openclaw gateway status
openclaw status
df -h
df -i
free -h
uptime
```

If the Gateway looks broken, collect recent logs before restarting:

```bash
journalctl --user -u openclaw-gateway.service --since '30 minutes ago' --no-pager
```

## Report back

After the dry run, report:

1. whether the Gateway is healthy,
2. which commands worked,
3. any warnings or blockers,
4. whether this environment is suitable for scheduled watchdog runs,
5. which schedule you recommend.

## Recommended schedule

Start conservatively:

- hourly: quick health check and safe repair workflow,
- every 6 hours: deeper host/OpenClaw check,
- weekly or manual: update-maintenance check, but do not auto-update unless explicitly approved.

If everything works reliably, the operator can decide whether to grant more autonomous permissions later.
