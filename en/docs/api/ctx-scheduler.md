---
title: ctx.scheduler
description: Recurring cron-based tasks, scoped to the plugin.
sidebar:
  order: 13
---

Schedules recurring cron tasks, scoped to your plugin (setup + runtime, but normally
used in `setup()` — calling it inside the message handler would register a new task on every
message).

```js
export async function setup(ctx) {
  ctx.scheduler.schedule("0 9 * * 1", async () => {
    await ctx.send.to("5511999999999@c.us").text("Good morning! Weekly report:");
  });
}
```

See the `schedule()` signature in [ctx — options and signatures](/docs/api/ctx-options/#ctxscheduler).

`expression` follows standard cron syntax (minute, hour, day of month, month, day of week). `fn`
runs without receiving `ctx` — close over the variables you need, as in the example above.

```js
const task = ctx.scheduler.schedule("*/5 * * * *", async () => { /* ... */ });
task.stop(); // cancels it — useful if the scheduling condition is dynamic
```

> Calling `schedule()` again with the **same** cron expression on the same plugin **replaces** the
> previous task, instead of accumulating two running in parallel — safe to call again on every
> plugin hot-reload. An invalid cron expression doesn't throw: it logs a warning and returns a handle
> that does nothing.
