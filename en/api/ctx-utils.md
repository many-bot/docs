---
title: ctx.utils
description: General utilities — today, just emptyFolder.
sidebar:
  order: 11
---

```js
ctx.utils.emptyFolder(DOWNLOADS_DIR); // deletes contents without removing the folder
```

> Only deletes **files directly inside** the folder — it's not recursive, subfolders (and their
> contents) are left intact. Throws an error if the folder doesn't exist; make sure it exists first (or
> wrap it in `try/catch`).
