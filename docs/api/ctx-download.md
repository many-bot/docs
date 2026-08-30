---
title: ctx.download
description: Fila serializada pra downloads pesados, pra não travar o event loop.
sidebar:
  order: 12
---

Fila serializada pra downloads pesados — **não** baixe direto no handler, isso trava o event
loop e atrasa outras mensagens.

```js
export default async function (ctx) {
  if (!ctx.msg.is("video")) return;
  const url = ctx.msg.args[0];
  if (!url) return void await ctx.msg.reply.text("Informe uma URL.");

  await ctx.msg.reply.text("Baixando, aguarde...");

  ctx.download.enqueue(
    async () => {
      const filePath = await baixarVideo(url);
      await ctx.send.video(filePath);
    },
    async (err) => {
      ctx.log.error(`Download falhou: ${err.message}`);
      await ctx.msg.reply.text("Falha no download.");
    }
  );
}
```

Só um job roda por vez. `errorFn` (segundo argumento) é opcional — se omitido, o erro ainda é
logado via `logger.warn` (não é engolido silenciosamente), mas o recomendado é sempre passar os
dois pra dar feedback ao usuário.
