# Local Cron + Telegram Watchdog

This is the recommended production architecture for a VPS-local OpenClaw watchdog.

## Why local cron?

Claude Code `/schedule` and Claude Cloud Routines run remote agents in Anthropic's cloud. They do **not** automatically run on the VPS and cannot access local files, local services, local environment variables, `systemctl --user`, or `openclaw gateway status` unless you deliberately provide a connector such as SSH or MCP.

For a watchdog whose job is to repair a local OpenClaw Gateway, the safest default is therefore:

```text
local cron/systemd timer on the VPS
  -> local check/repair scripts
  -> Telegram notification only when something meaningful happens
```

Telegram works well because it provides phone notifications without giving a remote cloud agent SSH access to the VPS.

## Recommended layout

Example paths:

```text
/opt/openclaw-watchdog/
  quick-check.sh
  restart-gateway.sh
  deep-check.sh
  notify-telegram.sh
  logs/
```

Use whichever directory is appropriate for your server, but keep scripts readable and auditable.

## Environment file

Store notification secrets outside the repository, for example:

```bash
sudo install -d -m 700 /etc/openclaw-watchdog
sudoedit /etc/openclaw-watchdog/env
sudo chmod 600 /etc/openclaw-watchdog/env
```

Example variables:

```bash
TELEGRAM_BOT_TOKEN="..."
TELEGRAM_CHAT_ID="..."
```

Do not commit tokens, chat IDs, private hostnames, or production-specific config to this repository.

## Minimal notifier shape

`notify-telegram.sh` should send a concise message and fail closed if notification variables are missing.

Example behavior:

```bash
./notify-telegram.sh "OpenClaw watchdog: Gateway was restarted and is healthy again."
```

Implementation can use Telegram's `sendMessage` endpoint, but keep the token in `/etc/openclaw-watchdog/env`, not in the script.

## Cron shape

Start conservatively:

```cron
# quick check every 5 minutes
*/5 * * * * /opt/openclaw-watchdog/quick-check.sh >> /opt/openclaw-watchdog/logs/quick-check.log 2>&1

# deeper check every 6 hours
0 */6 * * * /opt/openclaw-watchdog/deep-check.sh >> /opt/openclaw-watchdog/logs/deep-check.log 2>&1
```

The quick check should be quiet when healthy. Notify only when:

- a repair was performed,
- repair failed,
- an approval is needed,
- disk/RAM/load crosses serious thresholds,
- OpenClaw is down or unreachable after one restart attempt.

## Safety boundaries

Local cron may perform only pre-approved safe actions:

- read service status,
- read recent logs,
- run OpenClaw status/probe commands,
- restart `openclaw-gateway.service` once when clearly down,
- verify and notify.

Do not let cron silently:

- change firewall, SSH, Tailscale, public exposure, auth, or secrets,
- install/remove packages,
- reboot the host,
- delete unknown data,
- update OpenClaw,
- modify OpenClaw config.

## When to use Claude Code Scheduled Tasks instead

Use Claude Code `/schedule` or Cloud Routines only if you deliberately configure one of these:

1. a secure SSH path from the remote agent to the VPS,
2. an MCP/connector that can safely run the needed local commands,
3. a public status endpoint with strong authentication.

These are advanced options. They add security and reliability tradeoffs. The local cron + Telegram architecture should be the default starting point.
