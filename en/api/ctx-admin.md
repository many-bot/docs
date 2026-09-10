---
title: ctx.admin
description: Group administration actions — kick, add, promote, demote, subject, description, photo, and invite link.
sidebar:
  order: 6
---

Group administration actions (runtime only). Check `ctx.chat.isGroup` and
`ctx.chat.isBotAdmin()` before calling — see [ctx.chat](/docs/api/ctx-chat/).

```js
await ctx.admin.kick(memberIds);              // string | string[]
await ctx.admin.add(memberIds);               // current group
await ctx.admin.add(memberIds).to(groupId);   // another group by ID — chainable
await ctx.admin.promote(memberIds);
await ctx.admin.demote(memberIds);
await ctx.admin.setSubject("New Name");
await ctx.admin.setDescription("...");
await ctx.admin.setProfilePic("/tmp/photo.jpg"); // or Buffer
const link = await ctx.admin.getInviteLink();          // current group
const otherLink = await ctx.admin.getInviteLink(groupId); // another group, by ID
await ctx.admin.revokeInvite();
```

> `kick`/`add`/`promote`/`demote` **throw an error** if WhatsApp rejects the operation for any
> of the given IDs (e.g. no permission, already a member, contact's privacy settings) — they don't fail
> silently. Wrap them in `try/catch` if you want to handle that instead of letting it propagate to the
> plugin's error guard (see [guardOptions](/docs/api/plugins-basic/#guardoptions)).

> Only `add()` is chainable with `.to(groupId)` to target another group. `getInviteLink()` also
> works in `setup()` **if** you pass an explicit `groupId` (it doesn't depend on a current chat). The
> rest (`kick`, `promote`, `setSubject`, etc.) always operate on the current chat and require runtime
> context.

Standard admin command guard:

```js
export default async function (ctx) {
  if (!ctx.chat.isGroup) return;
  if (!ctx.msg.is("kick")) return;
  if (!await ctx.chat.isSenderAdmin()) return void await ctx.msg.reply.text("Admins only.");
  if (!await ctx.chat.isBotAdmin())    return void await ctx.msg.reply.text("I need to be an admin.");

  await ctx.admin.kick(ctx.msg.args[0]);
  await ctx.send.text("Removed.");
}
```

> **kick ≠ ban:** WhatsApp has no native ban. Combine `kick` with `ctx.contacts.block` to
> prevent re-entry.
