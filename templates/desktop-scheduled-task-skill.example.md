---
name: openclaw-devops-watchdog
description: Scheduled Claude Code DevOps watchdog for the operator's OpenClaw Gateway. Checks Gateway, host health, browser/CDP, performs safe repairs, and escalates risky changes.
---

# OpenClaw DevOps Watchdog

Use this as the body of a Claude Code Desktop scheduled task skill if you prefer task files over pasting prompts into the UI.

## Instructions

Read the runbook:

`openclaw-devops-runbook.md`

Then run the task matching this scheduled job:

- Hourly quick repair: `prompts/hourly-quick-repair.md`
- Six-hour deep check: `prompts/six-hour-deep-check.md`
- Update maintenance: `prompts/update-maintenance.md`

## Non-negotiables

- You may repair OpenClaw within the runbook boundaries.
- You must not delete unknown data.
- You must not expose tokens or secrets.
- You must not change firewall/SSH/auth/security posture without the operator's approval.
- You must notify the operator when you repair something, cannot repair something, or need approval.
