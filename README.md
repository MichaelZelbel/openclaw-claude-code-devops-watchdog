# Claude Code OpenClaw DevOps Watchdog

This folder contains a proposed Claude Code DevOps workflow for keeping an OpenClaw Gateway healthy.

Goal: Claude Code runs independently from OpenClaw, checks the OpenClaw VPS/Gateway, performs safe repairs, and escalates anything risky to the operator.

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
