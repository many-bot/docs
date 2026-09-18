---
title: ctx.download
description: Serialized queue for heavy downloads, so the event loop doesn't block.
sidebar:
  order: 12
---

Serialized queue for heavy downloads — **don't** download directly in the handler, that blocks the event
loop and delays other messages.

```js
export default async function (ctx) {
  if (!ctx.msg.is("video")) return;
  const url = ctx.msg.args[0];
  if (!url) return void await ctx.msg.reply.text("Provide a URL.");

  await ctx.msg.reply.text("Downloading, please wait...");

  ctx.download.enqueue(
    async () => {
      const filePath = await downloadVideo(url);
      await ctx.send.video(filePath);
    },
    async (err) => {
      ctx.log.error(`Download failed: ${err.message}`);
      await ctx.msg.reply.text("Download failed.");
    }
  );
}
```

Only one job runs at a time. `errorFn` (second argument) is optional — if omitted, the error is
still logged via `logger.warn` (it isn't silently swallowed), but it's recommended to always pass
both, to give the user feedback.
