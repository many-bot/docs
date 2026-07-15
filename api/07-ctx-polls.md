# ctx.poll (runtime only)

Enquetes com tracking automático de voto — sem precisar ouvir eventos manualmente.

```js
export default async function (ctx) {
  if (!ctx.msg.is("enquete")) return;

  const poll = await ctx.poll.create("Próximo tema?", ["Futebol", "Culinária", "Tech"]);

  poll.onVote((results) => {
    ctx.log.info(`Placar: ${JSON.stringify(results)}`);
  });
}
```

```js
// fechar e anunciar vencedor
if (ctx.msg.is("resultado")) {
  const poll = ctx.poll.get(pollMsgId); // guarde o msgId no escopo do plugin
  if (!poll) return void await ctx.msg.reply.text("Nenhuma enquete ativa.");

  const winners = poll.winner();
  const linhas = Object.entries(poll.results()).map(([k, v]) => `${k}: ${v} voto(s)`);
  await ctx.send.text([`*Resultado:* ${winners.join(" / ")}`, ...linhas].join("\n"));

  poll.close(); // libera memória
}
```

```js
// múltiplas respostas
await ctx.poll.create("Quais linguagens você usa?", ["JS", "Python", "Rust", "Go"], {
  allowMultipleAnswers: true,
});
```

| API | Assinatura | Descrição |
|---|---|---|
| `ctx.poll.create(q, opts[], cfg?)` | `Promise<PollHandle>` | Envia e começa a rastrear. |
| `ctx.poll.get(msgId)` | `PollHandle \| null` | Recupera handle ativo entre invocações. |
| `handle.onVote(cb)` | `(results, vote) => void` — encadeável | Chamado a cada mudança de voto. |
| `handle.results()` | `Record<string, number>` | Placar atual. |
| `handle.winner()` | `string[]` | Opção(ões) líder(es); vazio se sem votos. |
| `handle.close()` | `void` | Para o tracking, libera memória. |

> `ctx.send.poll()` vs `ctx.poll.create()`: use `send.poll()` se não precisa rastrear votos
> (retorna só `MessageHandle`). Use `poll.create()` pra `.onVote()`/`.results()`/`.winner()`.
