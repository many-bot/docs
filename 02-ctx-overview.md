# O objeto `ctx`

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
| [`ctx.config`](/docs/08-ctx-utilities/#ctxconfig) | ✅ | ✅ |
| [`ctx.i18n`](/docs/08-ctx-utilities/#ctxi18n) (e o atalho `ctx.t`) | ✅ | ✅ |
| [`ctx.utils`](/docs/08-ctx-utilities/#ctxutils) | ✅ | ✅ |
| [`ctx.download`](/docs/08-ctx-utilities/#ctxdownload) | ✅ | ✅ |
| [`ctx.scheduler`](/docs/08-ctx-utilities/#ctxscheduler) | ✅ | ✅ |
| [`ctx.storage`](/docs/08-ctx-utilities/#ctxstorage) | ✅ | ✅ |
| [`ctx.plugins`](/docs/08-ctx-utilities/#ctxplugins) | ✅ | ✅ |
| [`ctx.log`](/docs/08-ctx-utilities/#ctxlog) | ✅ | ✅ |
| [`ctx.botId`](/docs/08-ctx-utilities/#ctxbotid) | ✅ | ✅ |
| [`ctx.contacts`](/docs/05-ctx-contacts/) | ✅ | ✅ |
| [`ctx.me`](/docs/04-ctx-chat-admin-me/#ctxme) | ✅ | ✅ |
| [`ctx.settings`](/docs/08-ctx-utilities/#ctxsettings) | ⚠️ reduzido (só `.global`) | ✅ completo |
| [`ctx.admin.add()`](/docs/04-ctx-chat-admin-me/#ctxadmin) | ⚠️ só com `.to(chatId)` | ✅ |
| [`ctx.send.to()`](/docs/03-ctx-messaging/#ctxsend) | ✅ | ✅ |
| [`ctx.events`](/docs/06-ctx-events/) | ✅ | ❌ |
| [`ctx.send.text/image/...`](/docs/03-ctx-messaging/#ctxsend) (chat atual) | ❌ | ✅ |
| [`ctx.msg`](/docs/03-ctx-messaging/#ctxmsg) | ❌ | ✅ |
| [`ctx.chat`](/docs/04-ctx-chat-admin-me/#ctxchat) | ❌ | ✅ |
| [`ctx.admin`](/docs/04-ctx-chat-admin-me/#ctxadmin) (demais métodos) | ❌ | ✅ |
| [`ctx.poll`](/docs/07-ctx-polls/) | ❌ | ✅ |
| [`ctx.wa`](/docs/08-ctx-utilities/#ctxwa) (escape hatch, socket/store/msg crus) | ❌ | ✅ |

> `ctx.events` só existe no setup — registrar listener dentro do handler de mensagem criaria um
> listener novo a cada mensagem.
>
> `ctx.admin` existe em ambos, mas no setup não há um chat "atual": só `ctx.admin.add(ids).to(chatId)`
> funciona ali (é o único método admin encadeável com `.to()`). Os demais (`kick`, `promote`,
> `setSubject`, etc.) exigem o contexto de runtime e lançam erro se chamados no setup.
>
> `ctx.settings` existe em ambos, mas no setup só expõe `.global` (configurações do bot inteiro,
> sem chat associado) — o restante (`.forChat()`, `.link()`, etc.) só faz sentido em runtime.
