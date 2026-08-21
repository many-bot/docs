---
title: ctx.admin
description: Ações de administração de grupo — kick, add, promote, demote, subject, descrição, foto e convite.
sidebar:
  order: 6
---

Ações de administração de grupo (runtime only). Cheque `ctx.chat.isGroup` e
`ctx.chat.isBotAdmin()` antes de chamar — veja [ctx.chat](/docs/api/ctx-chat/).

```js
await ctx.admin.kick(memberIds);              // string | string[]
await ctx.admin.add(memberIds);               // grupo atual
await ctx.admin.add(memberIds).to(groupId);   // outro grupo por ID — encadeável
await ctx.admin.promote(memberIds);
await ctx.admin.demote(memberIds);
await ctx.admin.setSubject("Novo Nome");
await ctx.admin.setDescription("...");
await ctx.admin.setProfilePic("/tmp/foto.jpg"); // ou Buffer
const link = await ctx.admin.getInviteLink();          // grupo atual
const outroLink = await ctx.admin.getInviteLink(groupId); // outro grupo, por ID
await ctx.admin.revokeInvite();
```

> `kick`/`add`/`promote`/`demote` **lançam erro** se o WhatsApp rejeitar a operação pra qualquer
> um dos IDs informados (ex: sem permissão, já é membro, privacidade do contato) — não falham
> silenciosamente. Envolva em `try/catch` se quiser tratar isso ao invés de deixar propagar pro
> guard de erro do plugin (veja [guardOptions](/docs/api/plugins-basic/#guardoptions)).

> Só `add()` é encadeável com `.to(groupId)` pra mirar outro grupo. `getInviteLink()` também
> funciona no `setup()` **se** você passar um `groupId` explícito (não depende de chat atual). Os
> demais (`kick`, `promote`, `setSubject`, etc.) sempre operam no chat atual e exigem contexto de
> runtime.

Guard padrão de comando admin:

```js
export default async function (ctx) {
  if (!ctx.chat.isGroup) return;
  if (!ctx.msg.is("kick")) return;
  if (!await ctx.chat.isSenderAdmin()) return void await ctx.msg.reply.text("Só admins.");
  if (!await ctx.chat.isBotAdmin())    return void await ctx.msg.reply.text("Preciso ser admin.");

  await ctx.admin.kick(ctx.msg.args[0]);
  await ctx.send.text("Removido.");
}
```

> **kick ≠ ban:** WhatsApp não tem ban nativo. Combine `kick` com `ctx.contacts.block` pra
> impedir re-entrada.
