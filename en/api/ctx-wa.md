---
title: ctx.wa
description: Escape hatch to the WaContract (driver-neutral WhatsApp interface), for when something isn't covered by the rest of the API.
sidebar:
  order: 19
---

> [!WARNING]
> This endpoint will be fully removed in future updates — ManyBot will focus exclusively on using its own API.

Escape hatch to the `WaContract` — the **driver-neutral** interface the kernel uses internally to
talk to WhatsApp (implemented both by the Baileys driver and by other drivers) — for
when something you need isn't covered by the rest of the API. Runtime only.

```js
ctx.wa.contract; // WaContract instance — groupMetadata, sendImage, updateBlockStatus, etc.
ctx.wa.store;    // ManyBot's internal store (contacts/chats cache)
ctx.wa.msg;      // raw message object that triggered this handler
await ctx.wa.downloadMedia(); // download helper that skips ctx.msg's checks
```

`ctx.wa.contract` is **not** Baileys' raw socket (`@whiskeysockets/baileys`) — it's ManyBot's
own interface, with methods like `groupMetadata`, `sendImage`, `updateBlockStatus`,
`getBusinessProfile`, among others. This is intentional: the kernel never exposes the driver's
socket directly to plugins, precisely so plugins aren't locked to a specific library/driver.

```js
// example: contract method with no equivalent in ctx.admin/ctx.chat
export default async function (ctx) {
  if (!ctx.msg.is("business-profile")) return;
  const profile = await ctx.wa.contract.getBusinessProfile(ctx.msg.sender);
  await ctx.msg.reply.text(JSON.stringify(profile ?? "not a business account"));
}
```

> **Use with caution.** Unlike the rest of `ctx`, this doesn't go through ManyBot's
> normalizations (JID format, `guardOptions`, error handling/reload) — bugs here aren't
> caught by the same 3-attempt retry/disable that a normal `default()` breaking gets, and IDs that
> come straight from `contract`/`msg` are in the driver's native format (`@s.whatsapp.net`, not
> `@c.us`). Always prefer the normal API (`ctx.msg`, `ctx.chat`, `ctx.admin`, etc.) when it
> covers what you need.

`ctx.tg` and `ctx.dc` also exist in the interface (reserved for Telegram and Discord), but today they're
always `null` — ManyBot only has the WhatsApp driver implemented.
