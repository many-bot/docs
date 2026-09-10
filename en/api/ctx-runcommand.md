---
title: ctx.runCommand
description: Invokes another registered command through the same kernel pipeline (permission, arguments, subcommand, crash alert). Experimental.
sidebar:
  order: 23
  label: ctx.runCommand 🧪
---

> ⚠️ **Experimental — part of the [new `commands.yaml` architecture](/docs/commands-yaml/) 🧪,
> still being tested and not 100% functional. Subject to change without notice in `5.x` versions.**

Invokes another registered command through the **same pipeline** used for real messages:
permission check → subcommand routing → required-argument validation →
handler dispatch → crash capture. Runtime only.

```js
const result = await ctx.runCommand("sticker", "https://example.com/image.jpg");

switch (result.status) {
  case "executed":
    // ran normally
    break;
  case "permission_denied":
  case "argument_missing":
  case "unknown_sub":
    if (result.suggestedReply) await ctx.send.text(result.suggestedReply);
    break;
  case "no_dispatch":
    // invocation doesn't exist, or it's a fixed-text command (no plugin) — nothing was dispatched
    break;
}
```

- `invocation` — the command or alias, without the prefix (e.g. `"sticker"`, not `"!sticker"`).
- `rawArgs` — the rest of the line, unparsed (optional).

Runs with a `ctx` scoped to the plugin that **owns** the target command (its own `storage`, `plugins`,
`guardOptions`) — not the caller's context. Same principle as
[`ctx.plugins.require()`](/docs/api/ctx-plugins/), but for commands instead of the public API
exported by a plugin.

| Field | Description |
|---|---|
| `status` | `"executed"` \| `"permission_denied"` \| `"argument_missing"` \| `"unknown_sub"` \| `"no_dispatch"` |
| `sentReply` | Text the kernel itself already sent to the current chat during the pipeline (e.g. a permission warning), or `null`. |
| `suggestedReply` | Suggested text for the caller to decide whether to send, when the kernel didn't send anything on its own, or `null`. |

> Fixed-text commands (`text:`, without `plugin:`) and unknown invocations resolve with
> `status: "no_dispatch"` instead of throwing an error.
