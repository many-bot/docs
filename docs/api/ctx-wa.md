---
title: ctx.wa
description: Escape hatch pro socket cru do Baileys, pra quando algo não está coberto pelo resto da API.
sidebar:
  order: 19
---

Escape hatch pro socket cru do Baileys (a biblioteca por trás da conexão com o WhatsApp) — pra
quando algo que você precisa não está coberto pelo resto da API. Runtime only.

```js
ctx.wa.sock;    // instância do socket Baileys (@whiskeysockets/baileys) — API completa da lib
ctx.wa.store;   // store interno do ManyBot (cache de contatos/chats)
ctx.wa.msg;     // objeto de mensagem bruto do Baileys que disparou esse handler
await ctx.wa.downloadMedia(); // helper de download que não passa pelas checagens de ctx.msg
```

```js
// exemplo: método do Baileys sem equivalente em ctx.admin/ctx.chat
export default async function (ctx) {
  if (!ctx.msg.is("perfil-de-negocio")) return;
  const perfil = await ctx.wa.sock.getBusinessProfile(ctx.msg.sender);
  await ctx.msg.reply.text(JSON.stringify(perfil ?? "não é conta business"));
}
```

> **Use com cautela.** Diferente do resto do `ctx`, isso não passa pelas normalizações do
> ManyBot (formato de JID, `guardOptions`, tratamento de erro/reload) — bugs aqui não são
> pega pelo mesmo retry/disable de 3 tentativas de um `default()` normal quebrando, e IDs que
> saem direto do `sock`/`msg` vêm no formato nativo do Baileys (`@s.whatsapp.net`, não `@c.us`).
> Prefira sempre a API normal (`ctx.msg`, `ctx.chat`, `ctx.admin`, etc.) quando ela cobrir o que
> você precisa.

`ctx.tg` e `ctx.dc` também existem na interface (reservados pra Telegram e Discord), mas hoje são
sempre `null` — o ManyBot só tem driver de WhatsApp implementado.
