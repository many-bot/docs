# Padrões comuns

### Comando simples

```js
export default async function (ctx) {
  if (!ctx.msg.is("oi")) return;
  await ctx.msg.reply.text(`Olá, ${ctx.msg.senderName}!`);
}
```

### Ignorar o bot e mídia

```js
export default async function (ctx) {
  if (ctx.msg.fromMe) return;
  if (ctx.msg.type !== "chat") return; // só texto
  // ...
}
```

### Estado por chat (sessões)

```js
const sessoes = new Map();
const TIMEOUT = 2 * 60 * 1000;

export default async function (ctx) {
  const { msg, chat } = ctx;

  if (msg.is("iniciar")) {
    const timeout = setTimeout(() => sessoes.delete(chat.id), TIMEOUT);
    sessoes.set(chat.id, { autor: msg.sender, timeout });
    await msg.reply.text("Sessão iniciada! Você tem 2 minutos.");
    return;
  }

  if (msg.is("finalizar")) {
    const sessao = sessoes.get(chat.id);
    if (!sessao) return void await msg.reply.text("Nenhuma sessão ativa.");
    clearTimeout(sessao.timeout);
    sessoes.delete(chat.id);
    await msg.reply.text("Sessão encerrada.");
  }
}
```

### Setup com notificação

```js
const ADMIN = "5511999999999@s.whatsapp.net";

export async function setup(ctx) {
  ctx.log.success("meu-plugin carregado.");
  await ctx.send.to(ADMIN).text("Bot online! 🟢");
}

export default async function (ctx) {
  if (ctx.msg.is("status")) await ctx.send.text("Tudo funcionando.");
}
```

### Dados persistentes

```js
import { readFileSync, writeFileSync, existsSync } from "fs";

export default async function (ctx) {
  if (!ctx.msg.is("pontos")) return;

  const dbPath = ctx.storage.resolve("pontos.json");
  const db = existsSync(dbPath) ? JSON.parse(readFileSync(dbPath, "utf-8")) : {};
  db[ctx.msg.sender] = (db[ctx.msg.sender] ?? 0) + 1;
  writeFileSync(dbPath, JSON.stringify(db, null, 2));

  await ctx.msg.reply.text(`Você tem ${db[ctx.msg.sender]} ponto(s)!`);
}
```

### i18n própria

```js
export default async function (ctx) {
  const { t } = ctx.i18n.createT(import.meta.url);
  if (!ctx.msg.is("ajuda")) return;
  await ctx.send.text(t("ajuda.texto"));
}
```

### Restrito a admins

```js
export default async function (ctx) {
  if (!ctx.chat.isGroup) return;
  if (!ctx.msg.is("limpar")) return;

  if (!await ctx.chat.isAdmin(ctx.msg.sender)) {
    return void await ctx.msg.reply.text("Só admins podem usar esse comando.");
  }

  ctx.utils.emptyFolder("./downloads");
  await ctx.msg.reply.text("Cache limpo.");
}
```

### Lembrete agendado

```js
export async function setup(ctx) {
  ctx.scheduler.schedule("0 9 * * *", async () => {
    for (const chatId of chatsInscritos) {
      await ctx.send.to(chatId).text("Bom dia! ☀️");
    }
  });
}
```
