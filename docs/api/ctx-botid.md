---
title: ctx.botId
description: ID do próprio bot, string | null enquanto o client ainda inicializa.
sidebar:
  order: 18
---

```js
ctx.log.info(`Bot rodando como: ${ctx.botId}`);
```

`string | null` — pode ser `null` se o client ainda não tiver terminado de inicializar
(emite warning automático no log nesse caso).
