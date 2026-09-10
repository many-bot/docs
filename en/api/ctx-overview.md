---
title: The ctx object
description: The two variants of ctx (setup vs runtime) and where each API is available.
sidebar:
  order: 2
---

Two variants, depending on where it's passed.

## `plugin.setup(ctx)` — once, at startup

```js
export async function setup(ctx) {
  // no ctx.msg, ctx.chat
  // ctx.send only has .to(chatId) — no "current" chat
  await ctx.send.to("5511999999999@c.us").text("Bot online!");
}
```

## `plugin.default(ctx)` — on every message

```js
export default async function (ctx) {
  // everything from setup, plus:
  ctx.msg;   // the message that triggered the handler
  ctx.chat;  // the chat it came from
  ctx.send.text("...");   // shortcut for the current chat (no need for .to())
}
```

## Where each API is available

| API | setup | runtime |
|---|---|---|
| [`ctx.config`](/docs/api/ctx-config/) | ✅ | ✅ |
| [`ctx.i18n`](/docs/api/ctx-i18n/) (and the `ctx.t` shortcut) | ✅ | ✅ |
| [`ctx.utils`](/docs/api/ctx-utils/) | ✅ | ✅ |
| [`ctx.download`](/docs/api/ctx-download/) | ✅ | ✅ |
| [`ctx.scheduler`](/docs/api/ctx-scheduler/) | ✅ | ✅ |
| [`ctx.storage`](/docs/api/ctx-storage/) | ✅ | ✅ |
| [`ctx.plugins`](/docs/api/ctx-plugins/) | ✅ | ✅ |
| [`ctx.log`](/docs/api/ctx-log/) | ✅ | ✅ |
| [`ctx.botId`](/docs/api/ctx-botid/) | ✅ | ✅ |
| [`ctx.contacts`](/docs/api/ctx-contacts/) | ✅ | ✅ |
| [`ctx.me`](/docs/api/ctx-me/) | ✅ | ✅ |
| 🧪 [`ctx.commands`](/docs/api/ctx-commands/) (experimental) | ✅ | ✅ |
| [`ctx.settings`](/docs/api/ctx-settings/) | ⚠️ reduced (only `.global`) | ✅ full |
| [`ctx.admin.add()`](/docs/api/ctx-admin/) | ⚠️ only with `.to(chatId)` | ✅ |
| [`ctx.send.to()`](/docs/api/ctx-send/) | ✅ | ✅ |
| [`ctx.events`](/docs/api/ctx-events/) | ✅ | ❌ |
| [`ctx.send.text/image/...`](/docs/api/ctx-send/) (current chat) | ❌ | ✅ |
| [`ctx.msg`](/docs/api/ctx-msg/) | ❌ | ✅ |
| [`ctx.chat`](/docs/api/ctx-chat/) | ❌ | ✅ |
| [`ctx.admin`](/docs/api/ctx-admin/) (other methods) | ❌ | ✅ |
| [`ctx.poll`](/docs/api/ctx-poll/) | ❌ | ✅ |
| [`ctx.wa`](/docs/api/ctx-wa/) (escape hatch, raw socket/store/msg) | ❌ | ✅ |
| 🧪 [`ctx.session`](/docs/api/ctx-session/) (experimental) | ❌ | ✅ |
| 🧪 [`ctx.runCommand`](/docs/api/ctx-runcommand/) (experimental) | ❌ | ✅ |

> `ctx.events` only exists in setup — registering a listener inside the message handler would create a
> new listener on every message.
>
> `ctx.admin` exists in both, but in setup there's no "current" chat: only `ctx.admin.add(ids).to(chatId)`
> works there (it's the only admin method chainable with `.to()`). The rest (`kick`, `promote`,
> `setSubject`, etc.) require the runtime context and throw an error if called in setup.
>
> `ctx.settings` exists in both, but in setup it only exposes `.global` (bot-wide settings,
> not tied to a chat) — the rest (`.forChat()`, `.link()`, etc.) only make sense at runtime.
>
> 🧪 `ctx.commands`, `ctx.session`, and `ctx.runCommand` are part of the new
> [`commands.yaml`](/docs/commands-yaml/) architecture, still **experimental** and being tested.
