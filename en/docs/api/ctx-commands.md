---
title: ctx.commands
description: Read-only query into the central commands registry — exists, desc, manual, list, isMenuAlias. Experimental.
sidebar:
  order: 21
  label: ctx.commands 🧪
---

> ⚠️ **Experimental — part of the [new `commands.yaml` architecture](/docs/commands-yaml/) 🧪,
> still being tested and not 100% functional. Subject to change without notice in `5.x` versions.**

Read-only query into the central commands registry (setup + runtime). Lets a plugin
check whether another command exists, or read its description/manual, without needing the
owning plugin's `ctx.plugins.require()` — mainly meant for AI/menu plugins, which
mention commands and can't just make up (hallucinate) whether they exist or not.

```js
if (ctx.commands.exists("sticker")) {
  await ctx.send.text("Yes, that command exists!");
}

const description = ctx.commands.desc("sticker");        // string | null, in the current language
const descriptionEn = ctx.commands.desc("sticker", "en"); // string | null, forcing a language

const manual = ctx.commands.manual("sticker"); // falls back to desc if there's no manual

const all = ctx.commands.list(); // every registered top-level command, one item per stable id
// [{ id, cmd, aliases, category, desc }, ...]

ctx.commands.isMenuAlias("help"); // true if "help" is one of the menu command's own aliases
```

> Queries use the word (`cmd` or alias) exactly as declared in
> [`commands.yaml`](/docs/commands-yaml/) — no implicit case normalization.

> `list()` returns only **top-level** commands (one item per stable id) — it doesn't expand
> subcommands individually.
