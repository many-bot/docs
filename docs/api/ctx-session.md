---
title: ctx.session
description: Trava exclusiva por chat, escopada ao plugin que a abriu — evita que dois fluxos interativos rodem ao mesmo tempo no mesmo chat. Experimental.
sidebar:
  order: 22
  label: ctx.session 🧪
---

> ⚠️ **Experimental — parte da [nova arquitetura `commands.yaml`](/docs/commands-yaml/) 🧪,
> ainda em testes e não 100% funcional. Sujeito a mudar sem aviso em versões `5.x`.**

Trava exclusiva de chat (runtime only — não existe no `setup()`, já que não há chat atual pra
travar naquele momento), escopada ao chat atual **e** ao plugin que chamou. Serve pra fluxos
interativos (jogos, uma sessão de figurinha com timeout, um prompt de download em várias etapas)
que não podem ter dois plugins concorrendo pelo mesmo chat ao mesmo tempo.

O kernel só controla **quem** segura a trava — todo o estado da sessão em si (timeout, mídia
coletada, de quem é a vez, etc.) continua sendo responsabilidade do plugin.

```js
export default async function (ctx) {
  if (ctx.msg.is("iniciar-jogo")) {
    if (!ctx.session.acquire()) {
      return void await ctx.msg.reply.text("Já tem outra sessão ativa nesse chat.");
    }
    await ctx.msg.reply.text("Sessão iniciada!");
    return;
  }

  if (ctx.session.isMine()) {
    // processa a jogada, mensagem da sessão, etc.
  }

  if (ctx.msg.is("sair")) {
    ctx.session.release();
    await ctx.msg.reply.text("Sessão encerrada.");
  }
}
```

| Método | Descrição |
|---|---|
| `acquire()` | Abre a sessão pra este plugin neste chat. `true` se conseguiu (ou se este mesmo plugin já era o dono — seguro chamar de novo numa mensagem seguinte do mesmo fluxo); `false` se outro plugin já segura a trava. |
| `release()` | Libera a sessão — só tem efeito se este plugin for quem a segura. |
| `isLocked()` | Se o chat atual tem alguma sessão aberta (de qualquer plugin). |
| `isMine()` | Se **este** plugin é quem segura a sessão do chat atual. |
