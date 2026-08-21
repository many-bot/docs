---
title: ctx.storage
description: Diretório de dados persistentes do plugin.
sidebar:
  order: 14
---

Diretório de dados persistentes do plugin (`~/.manybot/data/<key>/`), criado automaticamente.

```js
import { readFileSync, writeFileSync, existsSync } from "fs";

const dbPath = ctx.storage.resolve("dados.json"); // cria subpastas se precisar
const dados = existsSync(dbPath) ? JSON.parse(readFileSync(dbPath, "utf-8")) : {};
dados[ctx.msg.sender] = Date.now();
writeFileSync(dbPath, JSON.stringify(dados, null, 2));
```

Veja `dir` e `resolve()` em [ctx — opções e assinaturas](/docs/api/ctx-options/#ctxstorage).

> Sobrevive a reinstalações. `manyplug remove` pergunta antes de apagar (`-Y` pula tudo).
> `resolve()` rejeita tentativas de escapar do diretório (`../`, caminhos absolutos) — sempre
> retorna um caminho dentro de `dir`.
