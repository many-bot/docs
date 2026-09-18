---
title: ctx.i18n
description: Translations — t()/createT() scoped to the plugin, per-language locales, and the ctx.t shortcut.
sidebar:
  order: 10
---

```js
export default async function (ctx) {
  const { t } = ctx.i18n.createT(import.meta.url); // t() scoped to your plugin

  if (ctx.msg.is("hi")) {
    await ctx.send.text(t("welcome", { name: ctx.msg.senderName }));
  }
}
```

Expected structure:

```
plugins/my-plugin/
  index.js
  locale/
    en.json   → { "welcome": "Welcome, {{name}}!" }
    pt.json
```

See the available methods (`t`, `createT`, `reload`, `getCurrentLang`) in
[ctx — options and signatures](/docs/api/ctx-options/#ctxi18n).

> No translation for the configured language → falls back to `en.json` automatically.
>
> `ctx.t` is a direct shortcut to the core's own `t` (equivalent to `ctx.i18n.t`) — useful if your
> plugin only needs the core translations and has no locale of its own.
>
> **`LANGUAGE` doesn't reload on its own:** unlike other `manybot.toml` keys, the language is
> loaded once per process. Changing `LANGUAGE` in the file doesn't change what `t()`/`ctx.t`
> translate at runtime — you need to call `ctx.i18n.reload()` (from some plugin) or restart the bot.
