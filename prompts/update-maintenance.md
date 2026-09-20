# Prompt: OpenClaw Update Maintenance

You are Claude Code acting as the operator's DevOps agent for OpenClaw.

Read and follow this runbook first:

`openclaw-devops-runbook.md`

## Task

Check whether OpenClaw should be updated. If the update is safe under the runbook policy, perform it and verify. If not, ask the operator.

## Pre-update checks

1. Record current OpenClaw version and the version on offer:
   - `openclaw --version`
   - `openclaw status`
   - `openclaw update status`
   - `openclaw update --dry-run` (prints the current version, the target version and the planned actions, and changes nothing)
2. Record Gateway service state:
   - `systemctl --user is-active openclaw-gateway.service`
   - `systemctl --user status openclaw-gateway.service --no-pager`
3. Record host health:
   - `df -h`
   - `df -i`
   - `free -h`
4. Confirm the correct update command from local help/docs. Do not guess update commands.
5. Read the release notes of every release between the installed version and the target, not only the newest one: <https://github.com/openclaw/openclaw/releases>. OpenClaw uses calendar versions (2026.9.5), so the number does not tell you whether a jump is major. The notes do: look for entries marked "(breaking)" and for migrations that need `openclaw doctor --fix`. Example: 2026.8.1 lists two breaking migrations, so any update that crosses it is not routine.
6. Take a snapshot before any update, with the state folder and the installed version in it:
   - `tar -czf ~/openclaw-state-$(date +%Y%m%d-%H%M).tar.gz -C ~ .openclaw`
   - keep the output of `openclaw --version` next to it. Reinstalling that version and unpacking the archive is the rollback path.
7. Decide whether this is safe automatic maintenance or requires the operator approval.

## Automatic update allowed only if

- It stays on the same channel, and no release between the installed version and the target lists a breaking change or a migration.
- The update command is confirmed.
- No OS reboot is required.
- No firewall/SSH/token/auth changes are involved.
- No major config migration is expected.
- A rollback/recovery path is clear.

## Requires the operator approval

- A channel switch, or an update that crosses a release with a breaking change or a migration (see pre-update check 5).
- OS package upgrades, kernel updates, or reboot.
- Config schema migration with unclear impact.
- Auth/security/token/firewall/SSH changes.
- Rollback/downgrade involving possible data migration.

## Post-update smoke tests

After updating, run:

1. `openclaw --version` and `openclaw status`: the version is the target from the dry run.
2. `openclaw update status`: it no longer offers the version you just installed.
3. `systemctl --user is-active openclaw-gateway.service`
4. `openclaw gateway status`
5. Gateway reachability check.
6. Playwright/browser runtime import check if applicable.
7. `example-remote-browser` CDP `/json/version` if the operator's PC browser is expected to be running.
8. Browser doctor/snapshot/evaluate if available.
9. Verify critical scheduled jobs still exist.

## If update breaks something

1. Do not panic-loop.
2. Collect logs and exact error messages.
3. Try safe repair once if clearly indicated.
4. If rollback is risky, ask the operator.
5. If rollback is safe and explicitly allowed by the runbook/current policy, do it and verify.

## Output rules

Always notify the operator after an update attempt, successful or failed, with:

- old version,
- new version,
- commands used,
- smoke test results,
- remaining warnings,
- whether any approval is needed.
