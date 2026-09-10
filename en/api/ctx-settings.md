---
title: ctx.settings
description: Per-chat (or global) settings that the user adjusts from WhatsApp itself, including communities.
sidebar:
  order: 15
---

Per-chat (or global) settings storage, persisted to disk — think of it as "preferences
the user adjusts from WhatsApp itself", different from [`ctx.storage`](/docs/api/ctx-storage/)
(the plugin's free-form data).

```js
// in the current chat
ctx.settings.set("welcome", true);
const enabled = ctx.settings.get("welcome", false); // with fallback
ctx.settings.getAll();      // all keys for this chat
ctx.settings.delete("welcome");
ctx.settings.deleteAll();

// bot-wide setting, not tied to any chat
ctx.settings.global.set("maintenanceMode", true);
ctx.settings.global.get("maintenanceMode", false);

// setting for another specific chat
ctx.settings.forChat(otherChatId).set("language", "en");
```

### Communities

Groups that belong to the same WhatsApp community can share settings:

```js
ctx.settings.link(communityId);          // links the current chat to a community
ctx.settings.unlink();                   // removes the current chat's link
ctx.settings.getCommunityId();           // string | null
ctx.settings.getCommunityChats();        // string[] — chats linked to the same community
```

See the full method table in [ctx — options and signatures](/docs/api/ctx-options/#ctxsettings).

> In `setup()`, only `ctx.settings.global` is available — with no current chat, the methods that
> operate on the "current chat" (including `forChat`/`link`/`unlink`) don't make sense there.
