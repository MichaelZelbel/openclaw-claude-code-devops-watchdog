# Troubleshooting

## Claude Code terminal connection closed unexpectedly

If the terminal running Claude Code disconnects, first determine what failed:

1. Did the VPS stay healthy?
   - `uptime`
   - `free -h`
   - `df -h`
   - `journalctl -k --since '2 hours ago' --no-pager | grep -Ei 'oom|killed process|segfault|out of memory'`

2. Did SSH or the web terminal disconnect?
   - `journalctl -u ssh --since '2 hours ago' --no-pager`
   - If using a provider browser console, retry over normal SSH. Browser consoles are more fragile than SSH.

3. Did Claude Code exit, or is it still running?
   - `ps aux | grep -Ei '[c]laude|[n]ode.*claude'`

4. Did OpenClaw itself stay healthy?
   - `systemctl --user is-active openclaw-gateway.service`
   - `openclaw gateway status`
   - `openclaw status`

Recommended prevention:

- Run Claude Code inside `tmux` or `screen`, not directly in a provider web terminal.
- Prefer logging in as the OpenClaw OS user instead of switching users mid-session.
- Ensure the OpenClaw CLI is on that user's `PATH`.
- If using a user service from another account, set `XDG_RUNTIME_DIR=/run/user/<uid>` correctly.

Example tmux flow:

```bash
tmux new -s openclaw-watchdog
# run Claude Code inside tmux

# if disconnected later:
tmux attach -t openclaw-watchdog
```

If Claude Code lost its context, restart the dry run from `AGENT_START.md` and mention what step disconnected.

## Gateway is down

- Check service state.
- Read recent logs.
- Restart the user service once.
- Re-check Gateway reachability.
- Escalate if the restart does not repair it.

## Disk is full

- Check `df -h` and `df -i`.
- Identify large logs/caches read-only first.
- Vacuum logs only when they are clearly the cause.
- Never delete unknown OpenClaw state, sessions, credentials, browser profiles, or databases.

## Agent stopped replying after self-modifying its config

Symptom: gateway is `active`, channels (Telegram, web chat) accept messages, but the agent never produces a reply. Logs show repeated lines like:

```
[tools] No callable tools remain after resolving explicit tool allowlist
(tools.allow: <id>); no registered tools matched.
```

Root cause: the agent set `tools.allow` in `~/.openclaw/openclaw.json` to an ID that does not match any registered tool. `tools.allow` is a restrictive whitelist, so the resolved toolset becomes empty and every run fails before it can reply.

Manual recovery:

1. Edit `~/.openclaw/openclaw.json` and remove the `tools.allow` array (keep `tools.profile`).
2. Restart `openclaw-gateway.service`.
3. Wait up to 25 s for warm-up before probing.

For a permanent guardrail that detects and reverses this within minutes, see `docs/agent-self-config-guardrail.md`.

## Browser/CDP works but screenshot fails

Treat this as a warning if snapshot/evaluate/navigation and direct Playwright/CDP checks work. File a bug with a minimal reproduction if needed.
