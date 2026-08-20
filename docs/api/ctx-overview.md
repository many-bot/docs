---
title: O objeto ctx
description: As duas variantes do ctx (setup vs runtime) e onde cada API está disponível.
sidebar:
  order: 2
---

Duas variantes, dependendo de onde ele é passado.

## `plugin.setup(ctx)` — uma vez, na inicialização

```js
export async function setup(ctx) {
  // sem ctx.msg, ctx.chat
  // ctx.send só tem .to(chatId) — não tem chat "atual"
  await ctx.send.to("5511999999999@c.us").text("Bot online!");
}
```

## `plugin.default(ctx)` — a cada mensagem

```js
export default async function (ctx) {
  // tudo do setup, mais:
  ctx.msg;   // a mensagem que disparou o handler
  ctx.chat;  // o chat de onde ela veio
  ctx.send.text("...");   // atalho pro chat atual (sem precisar de .to())
}
```

## Onde cada API está disponível

| API | setup | runtime |
|---|---|---|
| [`ctx.config`](/docs/api/ctx-utilities/#ctxconfig) | ✅ | ✅ |
| [`ctx.i18n`](/docs/api/ctx-utilities/#ctxi18n) (e o atalho `ctx.t`) | ✅ | ✅ |
| [`ctx.utils`](/docs/api/ctx-utilities/#ctxutils) | ✅ | ✅ |
| [`ctx.download`](/docs/api/ctx-utilities/#ctxdownload) | ✅ | ✅ |
| [`ctx.scheduler`](/docs/api/ctx-utilities/#ctxscheduler) | ✅ | ✅ |
| [`ctx.storage`](/docs/api/ctx-utilities/#ctxstorage) | ✅ | ✅ |
| [`ctx.plugins`](/docs/api/ctx-utilities/#ctxplugins) | ✅ | ✅ |
| [`ctx.log`](/docs/api/ctx-utilities/#ctxlog) | ✅ | ✅ |
| [`ctx.botId`](/docs/api/ctx-utilities/#ctxbotid) | ✅ | ✅ |
| [`ctx.contacts`](/docs/api/ctx-contacts/) | ✅ | ✅ |
| [`ctx.me`](/docs/api/ctx-chat-admin-me/#ctxme) | ✅ | ✅ |
| [`ctx.settings`](/docs/api/ctx-utilities/#ctxsettings) | ⚠️ reduzido (só `.global`) | ✅ completo |
| [`ctx.admin.add()`](/docs/api/ctx-chat-admin-me/#ctxadmin) | ⚠️ só com `.to(chatId)` | ✅ |
| [`ctx.send.to()`](/docs/api/ctx-messaging/#ctxsend) | ✅ | ✅ |
| [`ctx.events`](/docs/api/ctx-events/) | ✅ | ❌ |
| [`ctx.send.text/image/...`](/docs/api/ctx-messaging/#ctxsend) (chat atual) | ❌ | ✅ |
| [`ctx.msg`](/docs/api/ctx-messaging/#ctxmsg) | ❌ | ✅ |
| [`ctx.chat`](/docs/api/ctx-chat-admin-me/#ctxchat) | ❌ | ✅ |
| [`ctx.admin`](/docs/api/ctx-chat-admin-me/#ctxadmin) (demais métodos) | ❌ | ✅ |
| [`ctx.poll`](/docs/api/ctx-polls/) | ❌ | ✅ |
| [`ctx.wa`](/docs/api/ctx-utilities/#ctxwa) (escape hatch, socket/store/msg crus) | ❌ | ✅ |

> `ctx.events` só existe no setup — registrar listener dentro do handler de mensagem criaria um
> listener novo a cada mensagem.
>
> `ctx.admin` existe em ambos, mas no setup não há um chat "atual": só `ctx.admin.add(ids).to(chatId)`
> funciona ali (é o único método admin encadeável com `.to()`). Os demais (`kick`, `promote`,
> `setSubject`, etc.) exigem o contexto de runtime e lançam erro se chamados no setup.
>
> `ctx.settings` existe em ambos, mas no setup só expõe `.global` (configurações do bot inteiro,
> sem chat associado) — o restante (`.forChat()`, `.link()`, etc.) só faz sentido em runtime.
