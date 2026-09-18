---
title: ctx.poll
description: Create WhatsApp polls and track votes in real time. Runtime only.
sidebar:
  order: 8.2
---

Creates a WhatsApp poll and keeps the vote count updated automatically while the
plugin is active. Runtime only (needs a destination chat).

```js
export default async function (ctx) {
  if (!ctx.msg.is("vote")) return;

  const poll = await ctx.poll.create("Pizza or burger?", ["🍕 Pizza", "🍔 Burger"], {
    allowMultipleAnswers: false, // default
  });

  poll.onVote((results) => {
    ctx.log.info("current votes:", results);
  });
}
```

Methods on the handle returned by `create()`:

```js
poll.results();   // { "🍕 Pizza": 3, "🍔 Burger": 1 }
poll.onVote(cb);   // cb(results, raw) called on every vote change
poll.winner();     // name(s) of the leading option(s); [] if no one has voted yet
poll.close();      // stops tracking this poll (removes it from the internal registry)
```

`ctx.poll.get(msgId)` retrieves an already-created `PollHandle` (for example, after a
plugin reload), using the poll message's id.

```js
const poll = ctx.poll.get(msgId); // PollHandle | null
```

> Vote decryption and aggregation is driver-dependent — currently only the Baileys driver implements
> this support. On a driver that doesn't implement it, `ctx.poll.create()` still sends the poll
> normally, but votes aren't counted (`onVote` never fires).
