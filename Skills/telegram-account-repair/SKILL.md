---
name: "telegram-account-repair"
description: "Repair duplicate Telegram account entries in openclaw.json: detect by botToken, merge fields, drop with config.unset, restart and verify."
---

# Telegram Account Repair

Repair `channels.telegram.accounts` entries that duplicate one bot, and change account settings that need a Gateway restart to take effect.

## 路径代号解析（执行前必做）

- `<OPENCLAW_HOME>`：OpenClaw 状态目录（通常是 `%USERPROFILE%\.openclaw`）
- `%USERPROFILE%`：使用者 home 目录

## Trigger

- One bot answers on one account id but stays silent on another
- `channels.telegram.accounts` holds two keys with the same `botToken`
- A new account id, `dmPolicy`, `allowFrom`, or `groupAllowFrom` change did not take effect

## Detect duplicate entries

Group the entries of `channels.telegram.accounts` by `botToken`. Every group holding more than one key is a duplicate pair: the Control UI writes a lowercase-normalized `accountId` with its own field shape (`enabled`, `groups`, `allowFrom`, `groupAllowFrom`), while a hand-written entry keeps the camelCase key and carries `execApprovals`. Both hold the same token, so only the key referenced by `bindings` carries the binding and the approval settings.

To date an entry, list `openclaw.json.bak*`, `openclaw.json.last-good`, and `openclaw.json.pre-update` with their mtimes and diff the account key sets. The oldest backup missing the key shows when it appeared; a differing field set identifies which writer created it.

## Merge one account

1. Back up before touching the file:
   ```powershell
   Copy-Item "$env:USERPROFILE\.openclaw\openclaw.json" "$env:USERPROFILE\.openclaw\openclaw.json.bak.merge<NN>"
   ```
2. Keep the key that `bindings` references. Port the missing fields from the duplicate onto it: `enabled`, `groups: {"*": {"requireMention": true}}`, `allowFrom`, `groupAllowFrom`, and set `dmPolicy: "allowlist"` because `allowFrom` only takes effect under `allowlist`. Keep the existing `botToken`, `groupPolicy`, `streaming`, `execApprovals`.
3. Apply each config path in its own config tool call — one call listing several paths applies only the first one, silently.
4. Drop the duplicate key with `config.unset` on that path, not a `config.set` to null.
5. Restart the Gateway before testing, because merged account fields stay inactive until the Gateway reloads them. Never use `openclaw gateway restart`; a failed restart leaves the bot disconnected. On Windows use the scheduled task that runs the restart script, wait ~15s, then confirm the port's listening PID changed.
6. Verify by reading the file back, because the tool reply reports only what it applied:
   ```
   node -e "const j=require(process.env.USERPROFILE+'/.openclaw/openclaw.json');const a=j.channels.telegram.accounts;console.log(Object.keys(a).length);console.log(JSON.stringify(a['<Key>'],null,1));console.log(j.bindings.map(b=>b.agentId+'->'+b.match.accountId))"
   ```
   Confirm the account count dropped by one, the duplicate key is gone, the `botToken` is byte-identical to the backup, and the bindings list is unchanged. Then send the bot a direct message and confirm it answers, because a correct file does not prove a live bot.

## Adding a new account

Do not improvise the setup here: the token comes from the bot platform, and the account id, `botToken`, and `bindings` entry are plain `openclaw.json` fields covered by the channel's own documentation. Create the account, add the binding, restart the Gateway, then confirm the new bot answers.
