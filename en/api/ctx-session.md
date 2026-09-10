---
title: ctx.session
description: Exclusive per-chat lock, scoped to the plugin that opened it — prevents two interactive flows from running at the same time in the same chat. Experimental.
sidebar:
  order: 22
  label: ctx.session 🧪
---

> ⚠️ **Experimental — part of the [new `commands.yaml` architecture](/docs/commands-yaml/) 🧪,
> still being tested and not 100% functional. Subject to change without notice in `5.x` versions.**

Exclusive chat lock (runtime only — it doesn't exist in `setup()`, since there's no current chat to
lock at that point), scoped to the current chat **and** the plugin that called it. Meant for
interactive flows (games, a sticker session with a timeout, a multi-step download prompt)
that can't have two plugins competing for the same chat at the same time.

The kernel only controls **who** holds the lock — all of the session's actual state (timeout, collected
media, whose turn it is, etc.) remains the plugin's responsibility.

```js
export default async function (ctx) {
  if (ctx.msg.is("start-game")) {
    if (!ctx.session.acquire()) {
      return void await ctx.msg.reply.text("There's already another active session in this chat.");
    }
    await ctx.msg.reply.text("Session started!");
    return;
  }

  if (ctx.session.isMine()) {
    // process the move, session message, etc.
  }

  if (ctx.msg.is("exit")) {
    ctx.session.release();
    await ctx.msg.reply.text("Session ended.");
  }
}
```

| Method | Description |
|---|---|
| `acquire()` | Opens the session for this plugin in this chat. `true` if it succeeded (or if this same plugin already owned it — safe to call again on a later message in the same flow); `false` if another plugin already holds the lock. |
| `release()` | Releases the session — only has an effect if this plugin is the one holding it. |
| `isLocked()` | Whether the current chat has any open session (from any plugin). |
| `isMine()` | Whether **this** plugin is the one holding the current chat's session. |
