# Claude Code OpenClaw DevOps Watchdog

This folder contains a proposed Claude Code DevOps workflow for keeping an OpenClaw Gateway healthy.

Goal: Claude Code runs independently from OpenClaw, checks the OpenClaw VPS/Gateway, performs safe repairs, and escalates anything risky to the operator.

## TL;DR

1. Install OpenClaw on your VPS.
2. Install Claude Code on the same VPS, or on a trusted machine that can SSH into the VPS.
3. Open Claude Code and use this prompt:

```text
Configure yourself as the OpenClaw watchdog for this server.

Start here:
https://raw.githubusercontent.com/MichaelZelbel/openclaw-claude-code-devops-watchdog/main/AGENT_START.md
```

That starter file contains the safety boundaries, dry-run checks, and scheduling recommendations.

## Quick start

1. Clone or copy this repository onto the machine where Claude Code will run.
2. Make sure Claude Code can reach the OpenClaw VPS, either by running directly on the VPS or by SSHing into it.
3. Open this repository in Claude Code.
4. Start with a manual dry run before scheduling anything.

Suggested first prompt for Claude Code:

```text
You are setting up this repository as an OpenClaw DevOps watchdog.

Read README.md, CLAUDE.md, and openclaw-devops-runbook.md first.

Then do a manual safe dry run of the hourly quick repair workflow from prompts/hourly-quick-repair.md.

Important boundaries:
- Do not change firewall, SSH, Tailscale, secrets, auth, config, packages, or OS settings.
- Do not delete data.
- Do not update OpenClaw yet.
- Only run read-only checks unless the OpenClaw Gateway is clearly down.
- If the Gateway is down, you may restart only the user service openclaw-gateway.service once, then verify and report.

After the dry run, report:
1. whether the Gateway is healthy,
2. which commands worked,
3. any warnings that should be documented,
4. whether this machine is suitable for scheduled watchdog runs,
5. the exact scheduled task prompt/frequency you recommend.
```

## Manual VPS validation

Before enabling a scheduled watchdog, verify the basic commands on the VPS:

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

Expected healthy signs:

- `openclaw-gateway.service` is `active`.
- `openclaw gateway status` reports the Gateway runtime as running and the connectivity probe as ok.
- Disk usage is comfortably below warning levels, ideally below 80%.
- Memory and load are not under sustained pressure.

If `openclaw status` reports warnings, review them before giving Claude Code more autonomous permissions. Warnings are not always incidents, but they are useful setup notes.

## Recommended scheduling model

Use Claude Code Scheduled Tasks / Routines, not OpenClaw cron, for this watchdog. The point is that Claude Code should still be able to diagnose and repair OpenClaw if the OpenClaw Gateway is down.

Recommended tasks:

1. `openclaw-hourly-quick-repair`
   - Frequency: hourly
   - Prompt: `prompts/hourly-quick-repair.md`
   - Purpose: fast health check, safe repair, incident report only when needed.

2. `openclaw-six-hour-deep-check`
   - Frequency: every 6 hours
   - Prompt: `prompts/six-hour-deep-check.md`
   - Purpose: deeper Gateway, host, browser/CDP, update-readiness, disk/RAM/log health.

3. Optional `openclaw-update-maintenance`
   - Frequency: weekly or manual
   - Prompt: `prompts/update-maintenance.md`
   - Purpose: safe OpenClaw update flow with pre/post smoke tests and explicit rollback boundaries.

## Which Claude Code scheduling option?

- Claude Code Desktop Scheduled Tasks: best if Claude Code runs on a machine with direct SSH/local access to the VPS and configurable task permissions. Minimum interval is documented as 1 minute.
- Claude Code Cloud Routines: good if the routine has a secure connector/environment that can reach the VPS. Minimum interval is documented as 1 hour. Do not use cloud routines for repair unless secure VPS access is deliberately configured.
- `/loop`: useful for temporary incident work, not recommended as the long-term watchdog.

## Files

- `openclaw-devops-runbook.md` — operational policy and escalation rules.
- `prompts/hourly-quick-repair.md` — main hourly scheduled prompt.
- `prompts/six-hour-deep-check.md` — deep scheduled prompt.
- `prompts/update-maintenance.md` — update prompt.
- `templates/claude-settings.conservative.example.json` — stricter permission template.
- `templates/claude-settings.autonomous-devops.example.json` — more autonomous template after the operator explicitly approves it.
- `templates/desktop-scheduled-task-skill.example.md` — optional Claude Code Desktop scheduled task SKILL.md wrapper.

## Safety principle

Claude Code may repair OpenClaw, but it must not silently perform destructive, security-sensitive, or broad system changes. Restarting the Gateway is safe. Deleting unknown files, rotating secrets, changing firewall/SSH, and major updates are not.
