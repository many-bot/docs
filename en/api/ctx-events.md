---
title: ctx.events
description: Subscribe to raw WhatsApp events (messages, chats, contacts, groups, connection) outside the normal message cycle.
sidebar:
  order: 8.1
---

Subscribes to raw WhatsApp events directly on the `WaContract` — useful for reacting to things that don't
arrive as a regular message (group update, contact, connection status, etc.). Setup
only: subscribe in `plugin.setup(ctx)`, not in `default(ctx)`.

```js
export async function setup(ctx) {
  const unsubscribe = ctx.events.on("group-participants.update", (payload) => {
    ctx.log.info("participants changed", payload);
  });

  // call unsubscribe() if you need to stop listening before the plugin reloads
}
```

`ctx.events.once(event)` returns a Promise that resolves on the next time the event fires (and
removes the subscription on its own afterward):

```js
const payload = await ctx.events.once("connection.update");
```

You can only subscribe to a fixed set of events — passing any other name throws an
explicit error listing this set:

- `messages.upsert`
- `messages.update`
- `messages.delete`
- `messaging-history.set`
- `chats.upsert` / `chats.update` / `chats.delete`
- `contacts.upsert` / `contacts.update`
- `group-participants.update`
- `groups.upsert` / `groups.update`
- `group.join-request`
- `blocklist.set` / `blocklist.update`
- `connection.update`

> If you need an event outside this list, open an issue — adding a new event is a
> contract change (`WaContract`), not just a plugin API change.

Every subscription made via `ctx.events.on(...)` is automatically removed when the plugin is
reloaded or disabled — there's no need to clean it up manually when the plugin shuts down.
