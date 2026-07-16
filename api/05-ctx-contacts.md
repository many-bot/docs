# ctx.contacts

Informações de contatos (setup + runtime).

```js
const contact = await ctx.contacts.get(contactId);        // objeto normalizado | null
const picUrl  = await ctx.contacts.getPfpUrl(id);   // string | null
const path    = await ctx.contacts.getPfpPath(id, destPath); // baixa pra disco
const about   = await ctx.contacts.getAbout(id);           // string | null
await ctx.contacts.block(id);
await ctx.contacts.unblock(id);
```

```js
const contact = await ctx.contacts.get("5511999999999@s.whatsapp.net");
if (contact) ctx.log.info(`Pushname: ${contact.pushname}`);

const destPath = ctx.storage.resolve(`pfp_${contact.number}.jpg`);
const saved = await ctx.contacts.getPfpPath(contact.id, destPath);
if (saved) await ctx.send.image(saved, "Foto de perfil.");
```

> **`@lid`:** em grupos recentes o WhatsApp pode devolver IDs `@lid` (opacos, por privacidade).
> Dentro de um handler de mensagem, prefira `ctx.msg.getContact()` — ele resolve isso sozinho.

## Objeto de contato normalizado

Mesma shape em `ctx.contacts.get()` e `ctx.msg.getContact()`:

```ts
{
  id: string;             // "5511999999999@s.whatsapp.net"
  number: string;          // "5511999999999"
  pushname: string | null;
  name: string | null;     // salvo na sua agenda
  shortName: null;
  isBusiness: boolean;
  isEnterprise: boolean;
  isBlocked: boolean;
  isMe: boolean;
  isWAAccount: boolean;
  isUser: boolean;
  isGroup: boolean;
  mention: { text: string; mentions: string[] }; // spread nas opções de envio
}
```

> `shortName`, `isBusiness`, `isEnterprise` e `isBlocked` hoje são sempre `null`/`false` — o
> ManyBot ainda não deriva esses dados de verdade a partir do WhatsApp. Não confie neles pra
> decisões (ex: não use `isBlocked` pra saber se um contato te bloqueou).

```js
// Mencionar um contato
const contact = await ctx.msg.getContact();
await ctx.msg.reply.text(`oi ${contact.mention.text}`, contact.mention);
```
