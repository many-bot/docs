---
title: ctx.scheduler
description: Tarefas recorrentes via cron, escopadas ao plugin.
sidebar:
  order: 13
---

Agenda tarefas recorrentes via cron, escopadas ao seu plugin (setup + runtime, mas normalmente
usado no `setup()` — chamar dentro do handler de mensagem registraria uma tarefa nova a cada
mensagem).

```js
export async function setup(ctx) {
  ctx.scheduler.schedule("0 9 * * 1", async () => {
    await ctx.send.to("5511999999999@c.us").text("Bom dia! Relatório semanal:");
  });
}
```

Veja a assinatura de `schedule()` em [ctx — opções e assinaturas](/docs/api/ctx-options/#ctxscheduler).

`expression` segue a sintaxe padrão de cron (minuto, hora, dia do mês, mês, dia da semana). `fn`
roda sem receber `ctx` — feche sobre as variáveis que precisar, como no exemplo acima.

```js
const tarefa = ctx.scheduler.schedule("*/5 * * * *", async () => { /* ... */ });
tarefa.stop(); // cancela — útil se a condição de agendar for dinâmica
```

> Chamar `schedule()` de novo com a **mesma** expressão cron no mesmo plugin **substitui** a
> tarefa anterior, em vez de acumular duas rodando em paralelo — seguro chamar de novo a cada
> hot-reload do plugin. Expressão cron inválida não lança erro: loga um aviso e retorna um handle
> que não faz nada.
