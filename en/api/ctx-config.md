---
title: ctx.config
description: Read the user's manybot.toml from a plugin.
sidebar:
  order: 9
---

```js
const prefix = ctx.config.get("CMD_PREFIX");
const lang   = ctx.config.get("LANGUAGE", "en"); // with fallback
```

`get(key, defaultValue?)` — reads the user's `manybot.toml`. Default for the 2nd argument: `null`.
