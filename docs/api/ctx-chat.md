---
title: ctx.chat
description: Informações do chat atual — id, nome, se é grupo, participantes e checagem de admin.
sidebar:
  order: 5
---

Informações do chat atual (runtime only, já filtrado pela lista de permitidos do `manybot.toml`).

```js
ctx.chat.id;                          // string
ctx.chat.name;                        // string
ctx.chat.isGroup;                     // boolean
await ctx.chat.getParticipants();     // [] em privado — [{ id, isAdmin, isSuperAdmin }] em grupo
await ctx.chat.isAdmin(contactId);    // false em privado
await ctx.chat.isSenderAdmin();       // atalho pra isAdmin(ctx.msg.sender)
await ctx.chat.isBotAdmin();          // cheque antes de usar ctx.admin.*
ctx.chat.history;                     // WAHistoryArray — mensagens do chat, mais antigas primeiro
```

`ctx.chat.history` se comporta como um array normal (`history[10]`, `.length`, `.map()`, ...) e
tem dois filtros encadeáveis, ambos retornando outro `WAHistoryArray`:

```js
ctx.chat.history.last(5);           // últimas 5 mensagens
ctx.chat.history.from(contactId);   // só mensagens desse remetente
ctx.chat.history.last(20).from(contactId); // combinando os dois
```

> O histórico é mantido em memória, com um teto de **200 mensagens por chat** — mensagens mais
> antigas que isso não ficam disponíveis.

```js
if (ctx.chat.isGroup) {
  await ctx.send.text(`Olá, grupo *${ctx.chat.name}*!`);
} else {
  await ctx.send.text(`Olá, ${ctx.msg.senderName}!`);
}

if (ctx.msg.is("banir")) {
  if (!await ctx.chat.isSenderAdmin()) return void await ctx.msg.reply.text("Só admins.");
  // ...
}
```

> `ctx.chat.clearMessages()` existe na interface, mas atualmente **não tem efeito** — só registra
> um aviso no log. O Baileys ainda não expõe essa funcionalidade.
