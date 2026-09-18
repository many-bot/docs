---
title: ctx.botId
description: The bot's own ID, string | null while the client is still initializing.
sidebar:
  order: 18
---

```js
ctx.log.info(`Bot running as: ${ctx.botId}`);
```

`string | null` — can be `null` if the client hasn't finished initializing yet
(automatically emits a warning in the log in that case).
