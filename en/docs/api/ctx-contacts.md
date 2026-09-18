---
title: ctx.contacts
description: Contact information — get, getPfpUrl, getPfpPath, getAbout, block/unblock, and the normalized contact shape.
sidebar:
  order: 8
---

Contact information (setup + runtime).

```js
const contact = await ctx.contacts.get(contactId);        // normalized object | null
const picUrl  = await ctx.contacts.getPfpUrl(id);   // string | null
const path    = await ctx.contacts.getPfpPath(id, destPath); // downloads to disk
const about   = await ctx.contacts.getAbout(id);           // string | null
await ctx.contacts.block(id);
await ctx.contacts.unblock(id);
```

```js
const contact = await ctx.contacts.get("5511999999999@c.us");
if (contact) ctx.log.info(`Pushname: ${contact.pushname}`);

const destPath = ctx.storage.resolve(`pfp_${contact.numberRaw ?? contact.id}.jpg`);
const saved = await ctx.contacts.getPfpPath(contact.id, destPath);
if (saved) await ctx.send.image(saved, "Profile picture.");
```

> **`@lid`:** in recent groups WhatsApp may return `@lid` IDs (opaque, for privacy).
> Inside a message handler, prefer `ctx.msg.getContact()` — it resolves this on its own.

> **Returns `null` for an "unconfirmed" contact:** `get()`/`getContact()` return `null` (the
> whole object, not just `pushname: null`) when ManyBot doesn't have any record of that
> JID yet — neither in the contacts cache, nor confirmation the account exists. This is common right
> at the **first** contact with a new `@lid` (e.g. someone who just joined a group, or a
> self-chat). From the first message that person sends onward, ManyBot learns their
> `pushName` directly from the message itself — it doesn't depend solely on WhatsApp's contact
> sync, which can be slow or never run for an isolated `@lid`. In other words: if `get()` returns
> `null`, try again after the person sends at least one message while the bot was
> online; it's not an error to report.

> **`getPfpUrl()` has no cache:** every call hits WhatsApp's network (~150-350ms typical) — it isn't
> kept in memory the way `getParticipants()`/`isAdmin()` in `ctx.chat` are. Avoid calling it in a
> loop (e.g. for every participant of a large group) without spacing out the calls. It also returns
> `null` both for "contact with no picture" and for a network failure/timeout — you can't
> distinguish the two cases from the return value.

## Normalized contact object

Same shape in `ctx.contacts.get()` and `ctx.msg.getContact()`:

```ts
{
  id: string | null;              // canonical @lid JID, or "5511...@g.us" for a group; null if LID unknown
  number: string | null;          // E.164, e.g. "+5511999999999"
  numberRaw: string | null;       // digits only, no "+"
  numberPretty: string | null;    // formatted, e.g. "+55 11 99999 9999"
  country: string | null;         // ISO 3166-1 alpha-2, e.g. "BR"
  countryCallingCode: string | null; // country code, e.g. "55"
  pushname: string | null;
  name: string | null;     // saved in your address book
  shortName: null;
  isBusiness: boolean;
  isEnterprise: boolean;
  isBlocked: boolean;
  isMe: boolean;
  isWAAccount: boolean;
  isUser: boolean;
  isGroup: boolean;
  mention: { text: string; mentions: string[] }; // spread into send options
}
```

> **`id`/`number` can now be `null`:** since the LID-aware migration (Baileys v7), `id` is the
> contact's `@lid` JID — `null` when ManyBot hasn't learned that person's LID yet. `number` and the
> other phone fields (`numberRaw`, `numberPretty`, `country`, `countryCallingCode`) come from
> [`libphonenumber-js`](https://www.npmjs.com/package/libphonenumber-js) and are also `null`
> when unresolved or invalid. Always treat these fields as possibly `null` before
> using them — including `id`, which used to always be a string. See also the `sender`/`senderPn`
> note in [ctx.msg](/docs/api/ctx-msg/).
>
> `shortName`, `isEnterprise`, and `isBlocked` are currently always `null`/`false` — ManyBot doesn't yet
> derive that data for real from WhatsApp. Don't rely on them for decisions (e.g. don't use
> `isBlocked` to know whether a contact blocked you). `isBusiness` is different: it **is** actually
> checked against WhatsApp (an extra network call, only for individual contacts) — you can
> trust it.

```js
// Mention a contact
const contact = await ctx.msg.getContact();
await ctx.msg.reply.text(`hi ${contact.mention.text}`, contact.mention);
```
