---
title: ctx.chat
description: Informações do chat atual — id, nome, se é grupo, participantes, checagem de admin, e busca de outros chats/mensagens por ID.
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
await ctx.chat.getChat(jid);          // busca OUTRO chat pelo JID — objeto igual a este, ou null
await ctx.chat.getMsg(msgId);         // busca uma mensagem antiga pelo ID — objeto igual ao ctx.msg, ou null
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

### Buscando outro chat: `getChat()`

`ctx.chat` sempre descreve o chat de onde a mensagem atual veio. Pra olhar um chat **diferente**
(ex. um grupo que apareceu numa lista, um JID que seu plugin guardou antes, etc.) use
`getChat(jid)`:

```js
const outro = await ctx.chat.getChat("120363999999999@g.us");
if (!outro) return; // grupo inválido, ou o bot não é (mais) membro dele

console.log(outro.name, outro.isGroup);
const participantes = await outro.getParticipants();
const ehAdmin = await outro.isAdmin(contactId);
```

O objeto devolvido tem **exatamente a mesma forma** do `ctx.chat` — inclusive `history`,
`getParticipants()`, `isAdmin()`, `isBotAdmin()` e o próprio `getChat()` (dá pra encadear). A
única exceção é `isSenderAdmin()`: ela continua respondendo sobre quem enviou a mensagem
**atual** (a que disparou o handler), já que não existe outro "remetente" pra perguntar quando o
chat buscado não é o chat de origem.

`getChat()` nunca lança erro — devolve `null` quando o JID é um grupo inválido ou inacessível
(o bot não é membro, por exemplo). Um JID de conversa privada (`@c.us`) sempre resolve com
sucesso, já que não depende de nenhuma chamada de rede pra ser validado.

### Buscando uma mensagem antiga: `getMsg()`

Se seu plugin guardou o `id` de uma mensagem (`ctx.msg.id`) e precisa recuperá-la
depois pra reagir, responder ou só ler o conteúdo de novo, use `getMsg()`.

```js
const antiga = await ctx.chat.getMsg("3EB0...");
if (!antiga) return; // ID desconhecido, ou já saiu da janela de 200 msgs/chat

await antiga.reply.text("Ainda sobre isso:");
```

`getMsg()` resolve o chat automaticamente — não importa se a mensagem é deste chat ou de outro,
o único dado necessário é o ID. O objeto devolvido tem a mesma forma do `ctx.msg`
(veja [ctx.msg](/docs/api/ctx-msg/)), incluindo `reply`, `downloadMedia()`, `getContact()` etc.

> Assim como `ctx.chat.history`, `getMsg()` só enxerga o que ainda está na memória, então não
> persiste se reiniciar o bot.

> O teto de **200 mensagens por chat** vale aqui também. Um ID de mensagem muito antiga, já evictada,
> devolve `null`.

