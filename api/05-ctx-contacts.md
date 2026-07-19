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

> **Retorna `null` pra contato "não confirmado":** `get()`/`getContact()` devolvem `null` (o
> objeto inteiro, não só `pushname: null`) quando o ManyBot ainda não tem nenhum registro desse
> JID — nem no cache de contatos, nem uma confirmação de que a conta existe. Isso é comum logo
> no **primeiro** contato com um `@lid` novo (ex: alguém que acabou de entrar num grupo, ou uma
> auto-conversa). A partir da primeira mensagem que essa pessoa manda, o ManyBot aprende o
> `pushName` dela direto da própria mensagem — não depende só da sincronização de contatos do
> WhatsApp, que pode demorar ou nunca rodar pra um `@lid` isolado. Ou seja: se `get()` te devolver
> `null`, tenta de novo depois que a pessoa mandar pelo menos uma mensagem enquanto o bot estava
> online; não é um erro pra reportar.

> **`getPfpUrl()` não tem cache:** toda chamada bate na rede do WhatsApp (~150-350ms típico) — não
> guarda em memória como o `getParticipants()`/`isAdmin()` de `ctx.chat` fazem. Evite chamar em
> loop (ex: pra cada participante de um grupo grande) sem espaçar as chamadas. Também devolve
> `null` tanto pra "contato sem foto" quanto pra falha de rede/timeout — não dá pra distinguir os
> dois casos pelo retorno.

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
