---
title: ctx.me
description: Perfil do próprio bot — nome, recado (about) e foto de perfil.
sidebar:
  order: 7
---

Perfil do próprio bot (setup + runtime).

```js
await ctx.me.setName("ManyBot 🟢");
await ctx.me.setAbout("Online — digite !ajuda para começar.");
await ctx.me.setProfilePic("/tmp/avatar.jpg"); // ou Buffer

// atualizar em runtime conforme estado
if (filaCheia) await ctx.me.setAbout("Ocupado — processando downloads...");
```
