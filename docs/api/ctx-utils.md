---
title: ctx.utils
description: Utilidades gerais — hoje, apenas emptyFolder.
sidebar:
  order: 11
---

```js
ctx.utils.emptyFolder(DOWNLOADS_DIR); // apaga conteúdo sem remover a pasta
```

> Só apaga **arquivos diretamente dentro** da pasta — não é recursivo, subpastas (e o conteúdo
> delas) ficam intactas. Lança erro se a pasta não existir; garanta que ela existe antes (ou
> envolva em `try/catch`).
