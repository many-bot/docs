---
title: ctx.plugins
description: Communication between plugins via a public API — get, require, exists.
sidebar:
  order: 16
---

Communication between plugins via a public API.

```js
// plugins/my-database/index.js
export const api = {
  async findUser(id) { /* ... */ },
};
```

```js
// consuming it
const db = ctx.plugins.require("my-database");   // throws if it doesn't exist
const stats = ctx.plugins.get("many-stats");       // null if it doesn't exist
if (ctx.plugins.exists("many-ai")) { /* feature flag */ }
```

See the method table (`get`, `require`, `exists`) in
[ctx — options and signatures](/docs/api/ctx-options/#ctxplugins).
