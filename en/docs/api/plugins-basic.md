---
title: Anatomy of a plugin
description: The default function, the setup function, and the anti-detection/anti-ban layer (guardOptions) that wraps every plugin.
sidebar:
  order: 1
---

```js
// plugins/my-plugin/index.js

export async function setup(ctx) {
  ctx.log.info("my-plugin initialized!");
}

export default async function (ctx) {
  if (ctx.msg.is("hi")) {
    await ctx.send.text("Hello, world!");
  }
}
```

- `default` — called on every message received in an allowed chat.
- `setup` (optional) — called once, when the bot starts up.
- No central routing: every active plugin receives every message and decides on its own whether to act.

See the difference between setup and runtime `ctx` in [ctx overview](/docs/api/ctx-overview/).

---

## guardOptions

Controls the `pluginGuard` — the bot's anti-detection/anti-ban layer. Export `guardOptions` to
adjust it per plugin:

```js
export const guardOptions = {
  timeout: false,   // disables the 2min timeout
  typing:  false,   // disables the "typing..." indicator
  cooldown: false,  // disables the minimum interval between sends
  jitter:  false,   // disables the random delay before sending
};
```

| Option     | Default | What it does                                                        |
|------------|---------|-----------------------------------------------------------------------|
| `timeout`  | `true`  | Stops the plugin if it doesn't finish within 2 minutes — treated as the error below. |
| `typing`   | `true`  | Shows "typing…" while the plugin processes the message.               |
| `cooldown` | `true`  | Minimum interval between sends to the same chat.                      |
| `jitter`   | `true`  | Random delay before sending, to simulate human behavior.              |

> **Unhandled error (or `timeout: true` triggering):** the kernel catches it, logs a warning, and
> **reloads the plugin** — it stays active and tries again on the next message. Only after
> **3 consecutive failures** is the plugin actually disabled. In other words, a single error doesn't
> take your plugin down; failing every time does.

> `typing: false` only turns off the continuous indicator above — a brief indicator proportional to
> the content size still appears on every send (`ctx.send`/`ctx.msg.reply`), regardless of
> this option. For `ctx.send.audio()`/`ctx.msg.reply.audio()` that indicator is "recording
> audio…" (`recording`), different from the "typing…" (`typing`) used for text and other media.

> This **does not** prevent WhatsApp bans — it only mitigates some detection effects. See the
> [Terms of Use](/docs/terms-and-privacy/).
