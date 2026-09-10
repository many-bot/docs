---
title: ctx.chat
description: Info about the current chat — id, name, whether it's a group, participants, and admin checks.
sidebar:
  order: 5
---

Info about the current chat (runtime only, already filtered by the allow-list in `manybot.toml`).

```js
ctx.chat.id;                          // string
ctx.chat.name;                        // string
ctx.chat.isGroup;                     // boolean
await ctx.chat.getParticipants();     // [] in private chats — [{ id, isAdmin, isSuperAdmin }] in groups
await ctx.chat.isAdmin(contactId);    // false in private chats
await ctx.chat.isSenderAdmin();       // shortcut for isAdmin(ctx.msg.sender)
await ctx.chat.isBotAdmin();          // check before using ctx.admin.*
ctx.chat.history;                     // WAHistoryArray — chat messages, oldest first
```

`ctx.chat.history` behaves like a normal array (`history[10]`, `.length`, `.map()`, ...) and
has two chainable filters, both returning another `WAHistoryArray`:

```js
ctx.chat.history.last(5);           // last 5 messages
ctx.chat.history.from(contactId);   // only messages from this sender
ctx.chat.history.last(20).from(contactId); // combining both
```

> History is kept in memory, with a cap of **200 messages per chat** — messages older
> than that aren't available.

```js
if (ctx.chat.isGroup) {
  await ctx.send.text(`Hello, group *${ctx.chat.name}*!`);
} else {
  await ctx.send.text(`Hello, ${ctx.msg.senderName}!`);
}

if (ctx.msg.is("ban")) {
  if (!await ctx.chat.isSenderAdmin()) return void await ctx.msg.reply.text("Admins only.");
  // ...
}
```

> `ctx.chat.clearMessages()` exists in the interface, but currently **has no effect** — it only logs
> a warning. Baileys doesn't expose that functionality yet.
