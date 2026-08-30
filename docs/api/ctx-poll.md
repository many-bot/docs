---
title: ctx.poll
description: Cria enquetes no WhatsApp e acompanha os votos em tempo real. Runtime only.
sidebar:
  order: 8.2
---

Cria uma enquete no WhatsApp e mantém a contagem de votos atualizada automaticamente enquanto o
plugin estiver ativo. Runtime only (precisa de um chat de destino).

```js
export default async function (ctx) {
  if (!ctx.msg.is("votacao")) return;

  const poll = await ctx.poll.create("Pizza ou hambúrguer?", ["🍕 Pizza", "🍔 Hambúrguer"], {
    allowMultipleAnswers: false, // padrão
  });

  poll.onVote((results) => {
    ctx.log.info("votos atuais:", results);
  });
}
```

Métodos do handle devolvido por `create()`:

```js
poll.results();   // { "🍕 Pizza": 3, "🍔 Hambúrguer": 1 }
poll.onVote(cb);   // cb(results, raw) chamado a cada mudança de voto
poll.winner();     // nome(s) da(s) opção(ões) líder(es); [] se ninguém votou ainda
poll.close();      // para de rastrear essa enquete (remove do registro interno)
```

`ctx.poll.get(msgId)` recupera um `PollHandle` já criado (por exemplo, depois de um reload do
plugin), usando o id da mensagem da enquete.

```js
const poll = ctx.poll.get(msgId); // PollHandle | null
```

> A decriptação e agregação de votos depende do driver — hoje só o driver Baileys implementa
> esse suporte. Num driver que não implementa, `ctx.poll.create()` continua enviando a enquete
> normalmente, mas os votos não são contabilizados (`onVote` nunca dispara).
