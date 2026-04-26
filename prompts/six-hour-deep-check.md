# Prompt: OpenClaw Six-Hour Deep Check

You are Claude Code acting as the operator's DevOps agent for OpenClaw.

Read and follow this runbook first:

`openclaw-devops-runbook.md`

## Task

Run a deeper health check every six hours. Repair safe issues. Escalate risky ones.

## Required checks

### Host health

Run:

- `df -h`
- `df -i`
- `free -h`
- `uptime`
- `journalctl --disk-usage` if available.

Classify disk, inode, memory, swap, and load health using the runbook thresholds.

### Gateway service health

Run:

- `systemctl --user is-active openclaw-gateway.service`
- `systemctl --user status openclaw-gateway.service --no-pager`
- `journalctl --user -u openclaw-gateway.service --since '6 hours ago' --no-pager`

Look for repeated restarts, crashes, uncaught exceptions, auth problems, dependency errors, and OOM hints.

### OpenClaw health

Run:

- `openclaw status`
- `openclaw gateway status`
- `openclaw update status`
- `openclaw security audit` if not too expensive.

Record version and update availability.

### Browser automation health

If the operator's PC browser is expected to be running, check `example-remote-browser`:

- `curl -sS --max-time 8 http://<REMOTE_CDP_HOST>:9223/json/version`
- `curl -sS --max-time 8 http://<REMOTE_CDP_HOST>:9223/json/list`

If the OpenClaw browser tool is available in your environment, also test:

- browser doctor for profile `example-remote-browser`,
- tabs,
- snapshot,
- read-only evaluate.

Do not post anything to Discord.

Known caveat: if only `browser.screenshot` fails but snapshot/evaluate/navigation and direct Playwright screenshot work, classify as warning rather than critical.

### Dependency smoke test

If browser automation matters, verify that Playwright can be imported from the OpenClaw runtime. Use the local path only after confirming it exists. The known historical check was:

```bash
cd <OPENCLAW_INSTALL_DIR>/dist && node --input-type=module -e "import('./pw-ai-BDOHNhdx.js').then(()=>console.log('playwright import ok')).catch(e=>{console.error(e); process.exit(1)})"
```

If the bundle filename changed, locate the current browser/Playwright module rather than assuming the old filename.

## Auto-repair

Use the safe auto-repair rules from the runbook:

- Gateway restart is allowed when needed.
- Conservative journal vacuum is allowed only under critical disk pressure when logs are clearly the cause.
- Dependency fixes are allowed only for exact known, reversible failures; otherwise ask the operator.
- Updates follow the update policy and must include smoke tests.

## Output rules

- If healthy: write a concise local summary. Notify the operator only if periodic summaries are desired by the configured Claude task.
- If warnings exist but no action is needed: summarize warnings, with priority.
- If repaired: report what was changed and current status.
- If blocked: report exact blocker and recommended next action.
