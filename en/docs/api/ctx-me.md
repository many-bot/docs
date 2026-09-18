---
title: ctx.me
description: The bot's own profile — name, about, and profile picture.
sidebar:
  order: 7
---

The bot's own profile (setup + runtime).

```js
await ctx.me.setName("ManyBot 🟢");
await ctx.me.setAbout("Online — type !help to get started.");
await ctx.me.setProfilePic("/tmp/avatar.jpg"); // or Buffer

// update at runtime based on state
if (queueFull) await ctx.me.setAbout("Busy — processing downloads...");
```
