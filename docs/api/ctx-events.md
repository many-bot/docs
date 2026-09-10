---
title: ctx.events
description: Assina eventos brutos do WhatsApp (mensagens, chats, contatos, grupos, conexão) fora do ciclo normal de mensagem.
sidebar:
  order: 8.1
---

Assina eventos brutos do WhatsApp diretamente no `WaContract` — útil pra reagir a coisas que não
chegam como uma mensagem comum (atualização de grupo, contato, status de conexão, etc.). Setup
only: assine em `plugin.setup(ctx)`, não em `default(ctx)`.

```js
export async function setup(ctx) {
  const unsubscribe = ctx.events.on("group-participants.update", (payload) => {
    ctx.log.info("participantes mudaram", payload);
  });

  // chame unsubscribe() se precisar parar de escutar antes do plugin recarregar
}
```

`ctx.events.once(evento)` devolve uma Promise que resolve no próximo disparo do evento (e some a
inscrição sozinha depois):

```js
const payload = await ctx.events.once("connection.update");
```

Só é possível assinar um conjunto fixo de eventos — passar qualquer outro nome lança um erro
explícito citando essa lista:

- `messages.upsert`
- `messages.update`
- `messages.delete`
- `messaging-history.set`
- `chats.upsert` / `chats.update` / `chats.delete`
- `contacts.upsert` / `contacts.update`
- `group-participants.update`
- `groups.upsert` / `groups.update`
- `group.join-request`
- `blocklist.set` / `blocklist.update`
- `connection.update`

> Se precisar de um evento fora dessa lista, abra uma issue — adicionar um evento novo é uma
> mudança de contrato (`WaContract`), não só da API de plugin.

Toda inscrição feita via `ctx.events.on(...)` é automaticamente removida quando o plugin é
recarregado ou desativado — não é preciso limpar manualmente no encerramento do plugin.

