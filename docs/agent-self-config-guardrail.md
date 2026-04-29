# Guardrail: agent self-modified `tools.allow` allowlist

OpenClaw lets the agent itself edit its own configuration via `openclaw config set`. That is convenient, but it also means the agent can disable itself if it draws a wrong conclusion about its own setup. This document describes one specific failure mode that has been observed in production, and a watchdog pattern that detects and reverses it within minutes.

## The failure mode

The agent diagnoses a perceived problem (for example, a missing Telegram capability) and decides the fix is to set an explicit tool allowlist:

```jsonc
// ~/.openclaw/openclaw.json
"tools": {
  "profile": "coding",
  "allow": ["<some-tool-id-the-agent-invented>"]
}
```

In OpenClaw, `tools.allow` is a **restrictive whitelist** against the active `tools.profile`, not an additive extension. If the listed IDs do not match any tool the active plugins register, the resolved toolset is empty. From that point on, every agent run fails immediately with:

```
[tools] No callable tools remain after resolving explicit tool allowlist
(tools.allow: <id>); no registered tools matched.
```

The gateway still starts cleanly, the channel layer (Telegram, web chat, etc.) still accepts incoming messages, but the agent cannot produce any reply because it has zero callable tools. To an outside observer it looks like the agent silently went deaf.

Two factors make this hard to recover from automatically:

1. **The symptom only appears on the next agent run.** The gateway start does not exercise the toolset, so the broken configuration is recorded as a successful start.
2. **OpenClaw's own `openclaw.json.last-good` rotation considers any successful start as good.** That means the broken config is promoted to `last-good` after the first restart, and the built-in rollback is no longer useful.

If the agent is in a loop — for example, it restored its previous "fix" because the operator reverted it — the breakage will reappear minutes after every manual repair until the loop is broken externally.

## Detection

The guardrail uses two independent triggers and treats either as cause for rollback.

### T1: reactive log scan

Search the journal for the literal pattern `No callable tools remain` over a recent window (10 minutes is a good default). One or more hits means the agent is currently failing every run with this exact error.

```bash
journalctl --since "10 minutes ago" --no-pager \
  | grep -c "No callable tools remain"
```

### T2: preventive config drift

Compare `tools.allow` in the live config against a pinned known-good copy that the agent cannot read or write:

```bash
LIVE=/home/<openclaw-user>/.openclaw/openclaw.json
PINNED=/etc/openclaw-watchdog/known-good-config.json

live_allow=$(python3 -c "import json; print(json.dumps(json.load(open('$LIVE')).get('tools',{}).get('allow', [])))")
pinned_allow=$(python3 -c "import json; print(json.dumps(json.load(open('$PINNED')).get('tools',{}).get('allow', [])))")
```

The pinned file should be `root:root mode 600`, stored under `/etc/` rather than the OpenClaw user's home, so the agent has no path to overwrite it even with elevated tools.

T2 fires before any user notices the breakage, because drift is detected on schedule, not on first failed run.

## Rollback

Rollback steps, in order:

1. If the live file has the immutable attribute (`chattr +i`), lift it for the duration of the rollback. This handles the case where the file was previously locked as a stop-gap.
2. Snapshot the bad live config to a timestamped path, for forensics.
3. Copy the pinned known-good over the live config. Restore ownership and mode (`chown <openclaw-user>:<openclaw-user>`, `chmod 600`).
4. Re-apply `chattr +i` only if it was set before the rollback. Do not add the lock if it was not there.
5. Restart the gateway service.
6. Probe with a retry loop. OpenClaw's warm-up is roughly 10–25 s, so a single sleep-then-probe is too tight. Loop with a longer total budget (60 s is usually plenty) and accept the first probe that reports `Connectivity probe: ok`.

After a successful rollback, send a single Telegram alert to the operator. Stay silent on healthy runs.

## Why a separate "pinned" file

OpenClaw's bundled `openclaw.json.last-good` is rotated on every successful configuration load. A configuration that disables the toolset still loads successfully; only the next agent run exposes the problem. The bundled rollback therefore promotes broken configs into the "good" slot.

A pinned known-good owned by `root` and stored outside the OpenClaw user's home avoids this and gives you a deterministic baseline. Pair it with a small helper that the operator runs manually after intentional config changes, so the baseline tracks intent rather than drifting silently.

## Scheduling

A 5-minute cron interval is appropriate for this guardrail. The hourly quick-check used elsewhere in this repo is too slow for self-inflicted regressions: the agent can clobber the config minutes after a manual repair, and an hour of unanswered messages is a real outage.

```
*/5 * * * * /root/openclaw-watchdog/cron-toolset-guard.sh
```

The cron entry should be silent on healthy and notify only on rollback or rollback failure, matching the rest of the watchdog's reporting discipline.

## What this guardrail does *not* cover

- Other forms of agent self-sabotage (for example, disabling all plugins, removing channel allowlists, or rewriting model routing). Each needs its own pinned-baseline + drift detector if it has been observed in your environment.
- Legitimate operator changes to `tools.allow`. If you intentionally narrow the allowlist, update the pinned known-good first, otherwise the next guard run will undo your change.
- Cases where the symptom is something other than `No callable tools remain`. T1 only fires on that exact log line.

## Related operational rules

- `tools.allow` should normally be empty or absent. The default `tools.profile` is the right knob for most setups.
- If the agent insists a Telegram capability is missing, look at `channels.telegram.actions.*` first, not `tools.allow`. Topic creation and editing are channel-layer flags, not agent tools.
- Gateway warm-up is the single most common false-positive in restart verification. Use a probe loop with a 60 s budget rather than a fixed sleep.
