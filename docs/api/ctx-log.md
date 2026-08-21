---
title: ctx.log
description: Logging com formato consistente — info, warn, error, success.
sidebar:
  order: 17
---

```js
ctx.log.info("Iniciando processamento...");
ctx.log.warn("API key não configurada");
ctx.log.error(`Falha: ${err.message}`);
ctx.log.success("Sticker enviado!");
```

Prefira `ctx.log` a `console.log` — mantém formato consistente com o resto do bot.
