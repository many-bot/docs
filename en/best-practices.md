---
title: Plugin best practices
description: Recommendations for plugins to work well on any platform — native dependencies, event loop, persistence, guardOptions, i18n.
sidebar:
  order: 7
---

A collection of general recommendations for writing plugins that work well for whoever installs them —
whether on a Linux server, on Windows, or on an Android phone via Termux.

## Index

- [Native dependencies — be careful, especially with Android in mind](#native-dependencies-be-careful-especially-with-android-in-mind)
- [Don't block the event loop](#dont-block-the-event-loop)
- [Store state where it survives a restart](#store-state-where-it-survives-a-restart)
- [Don't fight guardOptions without a reason](#dont-fight-guardoptions-without-a-reason)
- [Avoid loops with fromMe](#avoid-loops-with-fromme)
- [Treat `null` as a normal response, not an exception](#treat-null-as-a-normal-response-not-an-exception)
- [Keep npm dependencies lean](#keep-npm-dependencies-lean)
- [Don't hardcode a number/JID](#dont-hardcode-a-numberjid)
- [Locale, even if just in your own language](#locale-even-if-just-in-your-own-language)

---

## Native dependencies — be careful, especially with Android in mind

ManyBot runs on Linux, Windows, **and Android (via Termux)** — and each platform has its own
architecture/libc. npm packages that depend on a **compiled native addon** (anything that
uses `node-gyp`, `node-pre-gyp`, `prebuild-install`, etc. under the hood) publish
prebuilt binaries only for the most common OS/architecture combinations — and Termux (Bionic libc, not
glibc) is almost never on that list. `npm install` then falls back to compiling from scratch on the spot,
which:

- requires a build toolchain installed on the device (`clang`, `make`, `python`, headers) —
  none of that comes by default;
- is slow and, on weaker phones, can hang or run out of memory;
- sometimes **just doesn't compile**, because the lib depends on system headers that Bionic
  doesn't have.

This means whoever installs your plugin on a normal Linux server won't notice anything — but
half of your Android user base will have the install failing, and it'll look like
`manyplug install` is broken, when in reality it's the dependency you chose.

**Before adding a dependency to `manyplug.json`, ask: does it have a pure
JS/WASM alternative?** Some common examples:

| Instead of... | Prefer | Why |
|---|---|---|
| `sqlite3` / `better-sqlite3` | [`node:sqlite`](https://nodejs.org/api/sqlite.html) (native to Node, available from 22, stable enough on 24 which ManyBot already requires) | Not a dependency at all — it's already built into the runtime, nothing to compile. |
| `bcrypt` | `bcryptjs` | Pure JS implementation, practically the same API. |
| `sharp` / `canvas` (heavy image processing) | [`jimp`](https://www.npmjs.com/package/jimp) for simple cases (resize, convert, watermark) | Pure JS/WASM, no native addon. If the plugin genuinely needs something at the level of `sharp` (e.g. bulk processing, heavy filters), document that clearly as a requirement in the README instead of assuming it "just installs". |
| Native FFmpeg bindings (`fluent-ffmpeg` calling an embedded native lib, `ffmpeg-static`, etc.) | Call the system's `ffmpeg` binary via `externalDependencies` (see [manyplug.json](/docs/how-to-make-a-plugin/#manyplugjson)) | The system binary already exists as a package on Termux (`pkg install ffmpeg`) and on distros — no need to embed/compile anything, just declare the external dependency. |

In general: **always prefer an external binary declared via `externalDependencies`**
(which the user installs through their own system's package manager — `apt`, `pkg`, etc.) **over
an npm lib that embeds/compiles that binary under the hood**. It's more transparent about
what will actually be installed, and it leaves compatibility to be resolved by each platform's
package manager — which is its job, not your plugin's.

If there's no alternative and the native dependency is truly necessary, at least make that
clearly visible in the plugin's `README.md`, so whoever installs it on Android knows
beforehand that it might cause problems, instead of finding out in the middle of a `manyplug install` that hangs.

---

## Don't block the event loop

Plugins run in sequence — **one stuck plugin delays the response of every other one** for
that message, because the kernel dispatches them one by one. Heavy work done directly in the handler
(parsing a large file, a long synchronous loop, heavy cryptography, resizing an image without
`await`) blocks the entire process, not just your plugin.

- Downloads and the like: use `ctx.download.enqueue()` (a serialized queue, already built for this) instead
  of downloading directly in the handler.
- Anything that legitimately takes more than a few seconds: reply with something like
  "processing..." and do the heavy work afterwards, sending the result when it's done via
  `ctx.send.to(chatId)` — don't leave the handler hanging while waiting.
- Remember that the default `timeout` in `guardOptions` interrupts the plugin if it doesn't finish within 2
  minutes, counting it as a failure (see [guardOptions](/docs/api/plugins-basic/#guardoptions)) —
  that's a sign the task should be async/queued, not a limit to try to squeeze under.

---

## Store state where it survives a restart

A module-scoped `Map()` (a common pattern for "sessions", queues, scoreboards) disappears entirely
when the bot restarts — which happens more often on a phone than on a server
(battery dying, Android killing the process, network switch). If the data genuinely matters
(a game's score, a setting the user adjusted, a pending task queue), persist it with
`ctx.storage`/`ctx.settings` instead of just memory. Reserve runtime memory for state
that's genuinely disposable (a 2-minute session timeout, a very short-lived cache).

---

## Don't fight guardOptions without a reason

`timeout`, `typing`, `cooldown`, and `jitter` exist to mitigate detection/banning of the
WhatsApp account. Disabling all four at once "because it's annoying during testing" is common during
development, but if that ships in the published plugin, everyone who installs it inherits that higher
risk without knowing. Only disable the specific option that genuinely doesn't fit your plugin (e.g.
`typing: false` on a service plugin that never shows "typing" because it doesn't respond
directly to messages) — not the whole set by default.

---

## Avoid loops with fromMe

This is already mentioned in [anatomy of a plugin](/docs/api/plugins-basic/), but it's worth reinforcing: the bot
also receives its own messages. Any handler that reacts to a "loose" message (not just a
prefixed command) without checking `ctx.msg.fromMe` risks replying to itself in a loop.

---

## Treat `null` as a normal response, not an exception

Several APIs return `null` in legitimate, relatively common scenarios, not just on error:
`ctx.msg.getContact()`, `ctx.contacts.get()`, `ctx.msg.getReply()`, `ctx.contacts.getPfpUrl()`.
Assuming the return is always populated and accessing a property directly (`contact.pushname` without
checking `contact` first) is the most common cause of a plugin silently breaking — an unhandled
error is automatically reloaded by the kernel (see
[guardOptions](/docs/api/plugins-basic/#guardoptions)), but if an unhandled `null` throws an error
on every message, your plugin may end up disabled until someone edits the file (triggering a reload)
or restarts the bot.

---

## Keep npm dependencies lean

Every entry in **`package.json`**'s `dependencies` (not to be confused with `manyplug.json`'s
`dependencies`, which is about other plugins — see
[manyplug.json](/docs/how-to-make-a-plugin/#dependencies)) gets installed via `npm` on the machine of
**every person** who installs your plugin — server, desktop, or phone. An extra dependency isn't
just weight: it's one more surface for the install to fail (see the native dependencies section above),
more `manyplug install` time, and one more thing to keep updated. Before adding a lib
to solve something small, consider whether you can do it with what Node/`ctx.utils` already offers.

---

## Don't hardcode a number/JID

Admin numbers, log groups, etc. — if hardcoded in the code, your plugin only works for
you. Read from `ctx.config.get(...)` (a key in the user's `manybot.toml`) or from
`ctx.settings`/`ctx.settings.global` (configurable from WhatsApp itself) instead of writing the
number directly in `index.js`.

---

## Locale, even if just in your own language

It's not mandatory, but error/response messages hardcoded directly in the code (without `ctx.i18n`) become
rework if one day you want to support another language, or if the plugin is used by people who
prefer `en`. See [locale](/docs/how-to-make-a-plugin/#locale).
