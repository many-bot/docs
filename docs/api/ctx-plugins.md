---
title: ctx.plugins
description: Comunicação entre plugins via API pública — get, require, exists.
sidebar:
  order: 16
---

Comunicação entre plugins via API pública.

```js
// plugins/meu-banco/index.js
export const api = {
  async buscarUsuario(id) { /* ... */ },
};
```

```js
// consumindo
const banco = ctx.plugins.require("meu-banco");   // lança erro se não existir
const stats = ctx.plugins.get("many-stats");       // null se não existir
if (ctx.plugins.exists("many-ai")) { /* feature flag */ }
```

Veja a tabela de métodos (`get`, `require`, `exists`) em
[ctx — opções e assinaturas](/docs/api/ctx-options/#ctxplugins).
