---
title: ctx.log
description: Logging with consistent formatting — info, warn, error, success.
sidebar:
  order: 17
---

```js
ctx.log.info("Starting processing...");
ctx.log.warn("API key not configured");
ctx.log.error(`Failed: ${err.message}`);
ctx.log.success("Sticker sent!");
```

Prefer `ctx.log` over `console.log` — it keeps consistent formatting with the rest of the bot.
