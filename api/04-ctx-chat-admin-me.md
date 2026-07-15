# ctx.chat

Informações do chat atual (runtime only, já filtrado pela lista de permitidos do `manybot.toml`).

```js
ctx.chat.id;                          // string
ctx.chat.name;                        // string
ctx.chat.isGroup;                     // boolean
await ctx.chat.getParticipants();     // [] em privado
await ctx.chat.isAdmin(contactId);    // false em privado
await ctx.chat.isSenderAdmin();       // atalho pra isAdmin(ctx.msg.sender)
await ctx.chat.isBotAdmin();          // cheque antes de usar ctx.admin.*
```

```js
if (ctx.chat.isGroup) {
  await ctx.send.text(`Olá, grupo *${ctx.chat.name}*!`);
} else {
  await ctx.send.text(`Olá, ${ctx.msg.senderName}!`);
}

if (ctx.msg.is("banir")) {
  if (!await ctx.chat.isSenderAdmin()) return void await ctx.msg.reply.text("Só admins.");
  // ...
}
```

> `ctx.chat.clearMessages()` existe na interface, mas atualmente **não tem efeito** — só registra
> um aviso no log. O Baileys ainda não expõe essa funcionalidade.

---

# ctx.admin

Ações de administração de grupo (runtime only). Cheque `ctx.chat.isGroup` e
`ctx.chat.isBotAdmin()` antes de chamar.

```js
await ctx.admin.kick(memberIds);              // string | string[]
await ctx.admin.add(memberIds);               // grupo atual
await ctx.admin.add(memberIds).to(groupId);   // outro grupo por ID — encadeável
await ctx.admin.promote(memberIds);
await ctx.admin.demote(memberIds);
await ctx.admin.setSubject("Novo Nome");
await ctx.admin.setDescription("...");
await ctx.admin.setProfilePic("/tmp/foto.jpg"); // ou Buffer
const link = await ctx.admin.getInviteLink();
await ctx.admin.revokeInvite();
```

> Só `add()` é encadeável com `.to(groupId)` pra mirar outro grupo — é o único método admin
> disponível no `setup()` (onde não existe um chat "atual"). Os demais (`kick`, `promote`,
> `setSubject`, etc.) sempre operam no chat atual e exigem contexto de runtime.

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

---

# ctx.me

Perfil do próprio bot (setup + runtime).

```js
await ctx.me.setName("ManyBot 🟢");
await ctx.me.setAbout("Online — digite !ajuda para começar.");
await ctx.me.setProfilePic("/tmp/avatar.jpg"); // ou Buffer

// atualizar em runtime conforme estado
if (filaCheia) await ctx.me.setAbout("Ocupado — processando downloads...");
```
