---
title: Configuration (manybot.toml)
description: All manybot.toml keys — CLIENT_ID, CMD_PREFIX, CHATS, LANGUAGE, PHONE_NUMBER, LOGIN_METHOD — and what reloads on its own.
sidebar:
  order: 2
---

On the first run, ManyBot creates `~/.manybot/manybot.toml` (on Windows,
`C:\Users\YourUser\.manybot\manybot.toml`) with commented default values.
Edit this file to configure the bot — there's no `manybot config` command,
it's just a text file.

> 🧪 Commands have their own file, `~/.manybot/commands.yaml` — separate from this one and in YAML,
> not TOML. It's the new commands architecture, still **experimental**: see
> [`commands.yaml`](/docs/commands-yaml/).

## Available keys

| Key          | Default      | Description                                                                 |
|----------------|-------------|----------------------------------------------------------------------------|
| `CLIENT_ID`    | `"manybot"` | Session identifier — separates login data under `~/.manybot/sessions/<CLIENT_ID>`. Run two ManyBot instances (two different numbers) on the same machine using a distinct `CLIENT_ID` in each `manybot.toml`. |
| `CMD_PREFIX`   | `"!"`       | Command prefix read by `ctx.msg.is()`/`ctx.msg.command`. Can be more than one character (e.g. `"bot "`). |
| `CHATS`        | `[]`        | List of allowed JIDs. Empty = **no restriction**, the bot responds in any conversation it's part of. See [below](#chats--restricting-where-the-bot-responds). |
| `LANGUAGE`     | `"en"`      | Interface and translation language (`t()`, `ctx.i18n`, `ctx.t`). Currently `en`, `es`, and `pt`. |
| `PHONE_NUMBER` | `""`        | Number (with country code) used to connect via pairing code. Only read if `LOGIN_METHOD = "phone"`. |
| `LOGIN_METHOD` | `""`        | `"qr"` or `"phone"`. Empty = asks interactively on first connection and saves the choice here automatically. |
| `EXCLUDE_CHATS` | `[]`       | List of JIDs to **ignore**, even if they pass the `CHATS` filter. Useful for excluding a specific chat without restricting everything else. |
| `SECURITY_LEVEL` | `"medium"` | `"low"` / `"medium"` / `"high"` — how cautious the bot is to avoid looking automated. Higher levels make the bot slower (fewer simultaneous chats, longer delays), but reduce the risk of WhatsApp flagging the account. Invalid value falls back to the default. |
| `LOG_LEVEL` | `"normal"` | `"normal"` (everything) / `"clean"` (hides routine noise, keeps success/warnings/errors) / `"minimal"` (only warnings and errors). Invalid value falls back to the default. |
| `OWNER_NUMBER` | `""`     | Number (or JID) treated as the bot's owner — this is the value the `owner: true` permission in [`commands.yaml`](/docs/commands-yaml/) checks against. Empty string is treated as "not configured". |
| `ADMIN_JID` | `""`        | Number/JID that receives alerts via WhatsApp when the bot starts up (critical alerts, update notices). Empty disables this channel — log and OS notifications keep working regardless. |
| `SMTP_HOST` / `SMTP_PORT` / `SMTP_SEC` / `SMTP_USER` / `SMTP_PASS` / `SMTP_FROM` / `SMTP_TO` | `""` / `587` / `"starttls"` / `""` / `""` / `""` / `""` | Optional email alert channel — leave `SMTP_HOST` blank to disable. `SMTP_SEC`: `"starttls"` (port 587, upgrades after connecting), `"ssl"` (port 465, encrypted from the start) or `"none"`. |
| `SMTP_INSECURE` | `false`  | Skips TLS certificate validation — needed for local SMTP proxies with a self-signed certificate (e.g. Proton Mail Bridge, Mailhog, Mailpit). Leave `false` for a real remote provider. |
| `UPDATE_CHECK_ENABLED` | `true` | Checks npm for a new ManyBot version: on startup, and then every `UPDATE_CHECK_INTERVAL_HOURS`. |
| `UPDATE_CHECK_INTERVAL_HOURS` | `24` | Interval (in hours) between update checks, when `UPDATE_CHECK_ENABLED = true`. |
| `STATUS_ENABLED` | `true`  | Enables a local HTTP page showing whether the bot is online or offline. |
| `STATUS_PORT` | `8080`     | Port for the status page, when `STATUS_ENABLED = true`. |

> The driver keys (`driver_primary`, `driver_baileys_enabled`, `driver_fallback_cooldown_ms`,
> `driver_verify_window_ms`) control connection driver selection/fallback. Today only the
> Baileys driver exists, so in practice only the defaults apply — no need to touch them.

> These are the keys ManyBot itself reads. Plugins can define and read
> additional keys in the same file via `ctx.config.get("MY_KEY")` — see
> [ctx.config](/docs/api/ctx-config/).

## What reloads on its own

ManyBot watches `manybot.toml` and applies most changes **without
restarting** — including `CMD_PREFIX` and `CHATS`, read directly from the file on every
message.

`LANGUAGE` is the exception: the translation system loads the language once
and doesn't watch this file. Changing `LANGUAGE` at runtime doesn't change the language of
already-loaded text — you need to restart the bot, or have a plugin call
`ctx.i18n.reload()` manually (see [ctx.i18n](/docs/api/ctx-i18n/)).

## CHATS — restricting where the bot responds

By default (`CHATS = []`) ManyBot processes messages from **any** chat —
group or private — the account is part of. To restrict it to a
specific list, fill `CHATS` with the desired JIDs:

```toml
CHATS = ["5511999999999@c.us", "120363000000000000@g.us"]
```

### Finding a chat's JID

Run the bot with the `--getid` flag:

```bash
manybot --getid
```

This opens a separate connection (doesn't interfere with the bot already running), syncs
the chat list, and shows an interactive selection — use the arrow keys and space to
mark one or more chats, Enter to confirm. The selected JIDs are
copied to the clipboard (and also printed in the terminal), ready
to paste into `CHATS`.

> Groups use the `@g.us` suffix; private chats use `@c.us` — this is the **conversation**
> format ManyBot exposes in `ctx.chat.id` and in this file's `CHATS`. **Person** identity
> (`ctx.msg.sender`, `id` in contact objects) is different: it uses `@lid`, not `@c.us` — see the
> full note in [ctx.msg](/docs/api/ctx-msg/).

## Legacy files

Older versions used `manybot.conf`/`manyplug.conf` (a custom format, not
TOML). On the first run after updating, ManyBot automatically migrates the `.conf`
to `.toml` and renames the original to `.conf.bak`. These
`.conf` files are frozen — they don't receive new keys — so there's no
reason to create one from scratch today.

## See also

ManyPlug has its own file, `~/.manybot/manyplug.toml` (list of
active plugins and CLI preferences) — see
[Configuration](/docs/manyplug-cli/#configuration) on the ManyPlug CLI page.
