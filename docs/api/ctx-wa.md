---
title: ctx.wa
description: Escape hatch pro WaContract (interface driver-neutra do WhatsApp), pra quando algo não está coberto pelo resto da API.
sidebar:
  order: 19
---

> [!WARNING]
> Esse endpoint será removido por completo em atualizações futuras, o ManyBot vai se concentrar somente no uso de sua API.

Escape hatch pro `WaContract` — a interface **driver-neutra** que o kernel usa internamente pra
falar com o WhatsApp (implementada tanto pelo driver Baileys quanto por outros drivers) — pra
quando algo que você precisa não está coberto pelo resto da API. Runtime only.

```js
ctx.wa.contract; // instância do WaContract — groupMetadata, sendImage, updateBlockStatus, etc.
ctx.wa.store;    // store interno do ManyBot (cache de contatos/chats)
ctx.wa.msg;      // objeto de mensagem bruto que disparou esse handler
await ctx.wa.downloadMedia(); // helper de download que não passa pelas checagens de ctx.msg
```

`ctx.wa.contract` **não** é o socket cru do Baileys (`@whiskeysockets/baileys`) — é uma interface
própria do ManyBot, com métodos como `groupMetadata`, `sendImage`, `updateBlockStatus`,
`getBusinessProfile`, entre outros. Isso é proposital: o kernel nunca expõe o socket do driver
diretamente pros plugins, justamente pra não travar plugins a uma biblioteca/driver específico.

```js
// exemplo: método do contract sem equivalente em ctx.admin/ctx.chat
export default async function (ctx) {
  if (!ctx.msg.is("perfil-de-negocio")) return;
  const perfil = await ctx.wa.contract.getBusinessProfile(ctx.msg.sender);
  await ctx.msg.reply.text(JSON.stringify(perfil ?? "não é conta business"));
}
```

> **Use com cautela.** Diferente do resto do `ctx`, isso não passa pelas normalizações do
> ManyBot (formato de JID, `guardOptions`, tratamento de erro/reload) — bugs aqui não são
> pega pelo mesmo retry/disable de 3 tentativas de um `default()` normal quebrando, e IDs que
> saem direto do `contract`/`msg` vêm no formato nativo do driver (`@s.whatsapp.net`, não
> `@c.us`). Prefira sempre a API normal (`ctx.msg`, `ctx.chat`, `ctx.admin`, etc.) quando ela
> cobrir o que você precisa.

`ctx.tg` e `ctx.dc` também existem na interface (reservados pra Telegram e Discord), mas hoje são
sempre `null` — o ManyBot só tem driver de WhatsApp implementado.

