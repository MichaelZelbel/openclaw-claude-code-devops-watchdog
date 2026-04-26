# Troubleshooting

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

## Browser/CDP works but screenshot fails

Treat this as a warning if snapshot/evaluate/navigation and direct Playwright/CDP checks work. File a bug with a minimal reproduction if needed.
