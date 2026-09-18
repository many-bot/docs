---
title: ctx.storage
description: The plugin's persistent data directory.
sidebar:
  order: 14
---

The plugin's persistent data directory (`~/.manybot/data/<key>/`), created automatically.

```js
import { readFileSync, writeFileSync, existsSync } from "fs";

const dbPath = ctx.storage.resolve("data.json"); // creates subfolders if needed
const data = existsSync(dbPath) ? JSON.parse(readFileSync(dbPath, "utf-8")) : {};
data[ctx.msg.sender] = Date.now();
writeFileSync(dbPath, JSON.stringify(data, null, 2));
```

See `dir` and `resolve()` in [ctx — options and signatures](/docs/api/ctx-options/#ctxstorage).

> Survives reinstalls. `manyplug remove` asks before deleting it (`-Y` skips all prompts).
> `resolve()` rejects attempts to escape the directory (`../`, absolute paths) — it always
> returns a path inside `dir`.
