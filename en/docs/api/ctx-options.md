---
title: "ctx: options and signatures"
description: Quick-reference tables — options per send method, and signatures of the utility methods (i18n, scheduler, storage, settings, plugins).
sidebar:
  order: 20
  label: Options and signatures
---

Quick reference for each method's options/signatures, gathered here so the same table doesn't repeat
on every endpoint page. For examples and usage explanations, see the corresponding endpoint
page.

## ctx.send

Options per method — see [ctx.send](/docs/api/ctx-send/) for examples.

| Method | Options |
|---|---|
| `text` | `{ linkPreview?, mentions?: string[] }` |
| `image` / `video` / `gif` | `{ viewOnce?, mentions?: string[] }` |
| `audio` | `{ asVoice? (default true), viewOnce? }` |
| `poll` | `{ allowMultipleAnswers? }` |
| `sticker` / `file` | no extra options |

## ctx.i18n

See [ctx.i18n](/docs/api/ctx-i18n/) for examples.

| Method | Signature | Description |
|---|---|---|
| `t` | `(key, context?)` | Translates a key from the **core** locales. |
| `createT` | `(import.meta.url)` | Returns `{ t, lang }` — `t()` scoped to the plugin's locales, `lang` is the currently active language. |
| `reload` | `()` | Reloads translations from disk. |
| `getCurrentLang` | `()` | `"pt"`, `"en"`, etc. |

## ctx.scheduler

See [ctx.scheduler](/docs/api/ctx-scheduler/) for examples.

| Method | Signature | Description |
|---|---|---|
| `schedule(expression, fn)` | `(string, () => Promise<void>) => { stop(): void }` | Registers a cron task; returns a handle to cancel it. |

## ctx.storage

See [ctx.storage](/docs/api/ctx-storage/) for examples.

| Prop/Method | Description |
|---|---|
| `dir` | Absolute path of the data directory. |
| `resolve(relativePath)` | Resolves a path inside `dir`, creating subfolders as needed. |

## ctx.settings

See [ctx.settings](/docs/api/ctx-settings/) for examples.

| Method | Description |
|---|---|
| `get(key, default?)` / `getAll()` | Reads a key (or all keys) from the current chat. Without a 2nd argument, returns `undefined` (not `null` — unlike `ctx.config.get()`). |
| `set(key, value)` / `delete(key)` / `deleteAll()` | Writes/removes in the current chat. `value` must be JSON-serializable. |
| `global` | Same methods above (`get`/`set`/`delete`/`getAll`/`deleteAll`), but scoped to the whole bot, with no associated chat. |
| `forChat(chatId)` | Same methods above, scoped to another specific chat. |
| `link(communityId)` / `unlink()` | Links/unlinks the current chat to a community. |
| `getCommunityId()` / `getCommunityChats()` | Queries the current link. |

## ctx.plugins

See [ctx.plugins](/docs/api/ctx-plugins/) for examples.

| Method | Description |
|---|---|
| `get(name)` | Public API or `null`. Optional dependency. |
| `require(name)` | Public API or throws. Required dependency. |
| `exists(name)` | `true` if the plugin is active. |
