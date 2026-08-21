---
title: ctx.config
description: Leitura do manybot.toml do usuário a partir de um plugin.
sidebar:
  order: 9
---

```js
const prefix = ctx.config.get("CMD_PREFIX");
const lang   = ctx.config.get("LANGUAGE", "pt"); // com fallback
```

`get(key, defaultValue?)` — lê o `manybot.toml` do usuário. Padrão do 2º argumento: `null`.
