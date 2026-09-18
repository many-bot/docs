---
title: commands.yaml
description: ManyBot's new command architecture — identity, permissions, arguments, loading indicator, and menu centralized in a single file. Experimental.
sidebar:
  order: 6.5
  label: commands.yaml 🧪
---

> ⚠️ **Experimental — under testing, not 100% functional yet.** `commands.yaml` is ManyBot's new
> command architecture. The core is already implemented and covered by tests
> (parser, registry, permissions, automatic deprecation, native menu, function chaining,
> loading indicator), but two points remain deliberately open: (1) the kernel
> still **doesn't enforce** that every command be in `commands.yaml` — a plugin with no
> entry at all keeps working the old way, and this should only become mandatory once the YAML
> becomes the primary path; (2) there's still no warning when the same function is
> referenced in the YAML **and** called directly by another plugin (dual use/conflict). Other
> than that, the schema may gain new fields or change behavior without notice across `5.x`
> versions — it's not recommended to depend on it in production yet. Once it matures enough, this
> architecture is meant to **replace** the current commands/plugins system (reading
> `ctx.msg.command`, manual menu, etc.) in **ManyBot 6.0**. Until then, the two models coexist:
> a plugin can keep deciding on its own whether it responds to a message (as today), or have its
> identity/permissions declared here.

## What changes

Today, a plugin decides on its own whether a message is "its" command (`ctx.msg.is("sticker")`),
handles its own permission checking, and there's no automatic menu — every bot needs a
manually configured menu plugin. With `commands.yaml`:

- **Identity and aliases** (`cmd`, `aliases`) move out of the plugin's code and into the YAML.
- **Permissions** (admin, owner, group/DM only, allowed chat list, cooldown,
  whitelist/blacklist) are applied by the kernel *before* the plugin runs — the plugin doesn't need
  to check this itself.
- **Description and manual** (`desc`, `manual`) feed a **native menu**, generated automatically
  as commands are registered.
- **"Processing..." indicator** (`loading:`) is also decided by the kernel — reaction,
  "typing...", "recording audio...", spinner via message edit, or none.
- **Required arguments** can be declared (`arguments:`) and the kernel validates before
  calling the plugin, responding on its own with a usage hint (generated automatically from the
  declared types) when something's missing.
- **One function or several in sequence** (`function:`/`functions:`) — the kernel can call more
  than one plugin function, in order, for the same command.
- **Renaming or removing** a command is detected automatically (history is saved to
  disk, survives restarts) and can notify whoever tries the old name for a
  transition period, instead of simply not responding anymore.

`commands.yaml` lives at `~/.manybot/commands.yaml`, alongside `manybot.toml` — but, unlike
the bot's general configuration (which is TOML), this one is YAML.

## File structure

By default, commands are declared **directly at the root** of the file — the only reserved
top-level keys are `defaults`, `menu`, `categories`, `manuals`, `import`, `loading_presets`,
`loading`, `prefix`, `notify_changes`, `notify_period_days`, `deprecation_message`,
`permission_messages`, and `commands`; any other first-level key is interpreted as
a command, and the key's name becomes that command's stable internal `id`.

It's also accepted to wrap commands under a `commands:` key — both forms are equivalent and
can even be combined (one key inside `commands:` and another at the root), as long as they don't
collide:

```yaml
defaults:
  notifyPeriodDays: 7

commands:
  sticker:
    cmd: sticker
    plugin: many-media
```

If the same key appears in both places (root and inside `commands:`), the collision is reported
as an error in the log and the version inside `commands:` is ignored.

`prefix:` at the root is accepted by the parser but is currently **informational only** — the
actual prefix used at runtime still comes from `manybot.toml`. Don't expect changing `prefix:` here
to change the bot's prefix.

## Minimal example

```yaml
sticker:
  cmd: sticker
  aliases: [f, sticker]
  plugin: many-media
  category: media
  desc: "Turns an image/video into a sticker"
```

## Structure of a command

```yaml
<stable-id>:
  cmd: <typed word>
  aliases: [<list>]
  plugin: <plugin-name>          # OR text: "fixed response" (no plugin)
  function: <function-name>      # or functions: [fn1, fn2, ...] — see "Function chaining"
  category: <category-id>
  group: <group-id>              # 🚧 accepted by the schema, but currently has no effect on the menu — see note below
  desc: "short description"      # optional — overrides the plugin's desc
  manual: "long text"            # optional — falls back to manuals.<id> if omitted
  deprecatedMessage: "..."       # optional — deprecation message specific to this command
  notifyChanges: true            # optional — overrides defaults.notifyChanges just for this command
  loading: <preset-or-inline>    # optional — "processing..." indicator, see its own section
  permissions: { ... }
  messages: { ... }
  arguments: [ ... ]             # or args: [...]
  subcommands: [ ... ]
```

The top-level key (`sticker` in the example) is a **stable internal id**, decoupled from `cmd:` (the
word the user types). It's this stable id that allows rename detection: changing only the
`cmd:` of an already-registered command (keeping the same id) is treated as a rename, not as
removing one command and creating another.

`cmd` is required — without it, the command doesn't load and a warning is logged. `desc` and
`manual` are entirely optional: without `desc`, the field is simply left out of the menu; without
`manual` (when someone explicitly asks for the manual), the kernel falls back to `desc`.

A command is either `plugin:` (routes to one or more plugin functions) or `text:` (fixed response)
— or, with no function of its own, a **container command** that just groups `subcommands:` (see
its own section below). Declaring `plugin:` without `function`/`functions` and without `subcommands:`
loads nothing — the kernel logs a warning and ignores the entry.

`plugin:` accepts either the registry's full key (`owner/repo`) or just the plugin's short
name (resolved automatically against active plugins — if the short name matches more than
one `owner/repo`, the ambiguity isn't resolved automatically, and the original value is kept, producing
an "orphan" command with a log warning). It also accepts the inline form `plugin: many-media.removeBackground`
(a dot separating plugin and function) as a shortcut for `plugin: many-media` + `function: removeBackground`.

> 🚧 The `group:` field in the schema is meant to group different commands under a single
> menu entry — but it currently **has no effect at all** (it's not read anywhere in the
> menu renderer). Avoid relying on it until grouping is actually implemented.

### Long text and manual from a file

`text:`/`manual:` accept inline content or a `file:./path` reference (relative to the
config folder, `~/.manybot/`). The content is read literally, with no parser — WhatsApp's
native formatting (`*bold*`, `_italic_`, etc.) already works directly in the file. If the file isn't
found, the kernel logs a warning and uses the reference itself (`file:./...`) as the text.

```yaml
rules:
  cmd: rules
  text: "file:./texts/rules.txt"
```

### Per-language description and manual

`desc`/`manual` accept either a plain string or a per-language map, using the same i18n system as
the rest of ManyBot ([`ctx.i18n`](/docs/api/ctx-i18n/)):

```yaml
sticker:
  cmd: sticker
  plugin: many-media
  desc:
    pt: "Transforma uma imagem/vídeo em figurinha"
    en: "Turns an image/video into a sticker"
```

## Function chaining (`functions:`)

Besides a single `function: "name"`, a command can declare a **chain** of functions from the
same plugin, executed in order:

```yaml
ban:
  cmd: ban
  plugin: many-mod
  functions: [logAttempt, executeBan]
```

Each function receives the same `(ctx, { args, subcommand })` shape. By default, a function's
return value doesn't stop the chain (even `undefined`/empty lets the next one run) — to
deliberately stop it, a function can return the `STOP_CHAIN` sentinel (exported from
`commandsConfig.ts`). An empty chain (a command with no `function`/`functions` and no `subcommands`
with their own handler) means the command is metadata only — nothing is executed.

## Loading indicator (`loading:`)

While the plugin processes, the kernel can show a "processing..." indicator without the
plugin needing to handle it:

```yaml
sticker:
  cmd: sticker
  plugin: many-media
  loading:
    type: reaction
    icon: "⏳"
    onSuccess: "✅"
    onError: "❌"
```

Recognized types:

- `reaction` — reacts to the original message with an emoji (`icon`, `onSuccess`/`on_success`,
  `onError`/`on_error`).
- `typing` — WhatsApp's native "typing..." presence. Doesn't accept extra properties.
- `recording_audio` — native "recording audio..." presence. Same rule as `typing`.
- `spinner` — edits a message sent by the bot itself, cycling through a list of `frames` at
  each `intervalMs`/`interval_ms` (minimum 1000ms; smaller values are rounded up).
  Also accepts `onSuccess`/`onError`.
- `none` — explicitly disables any indicator.

An unrecognized property for the declared type invalidates the whole `loading:` (fail
closed — the kernel logs an error and treats it as if nothing had been declared there).

`loading:` accepts either an inline object or the **name of a preset** defined under
`loading_presets:` (a top-level key):

```yaml
loading_presets:
  default:
    type: reaction
    icon: "⏳"

sticker:
  cmd: sticker
  plugin: many-media
  loading: default
```

The inheritance chain (from most specific to most generic) is:

```
command/subcommand  →  category (categories.<id>.loading)  →  defaults.loading (or top-level loading:)
```

The first level that declares a `loading:` (inline or an already-resolved preset) wins; omitted
levels simply pass through to the next in the chain. A subcommand with no `loading:` of its own
inherits the parent command's already-resolved `loading:`.

## Permissions

```yaml
ban:
  cmd: ban
  plugin: many-mod
  permissions:
    admin: true          # sender must be a group admin
    botAdmin: true        # the bot must be a group admin
    scope: group          # group | dm | any (default: any)
    owner: false           # true = only the number configured as OWNER_NUMBER
    dono: "5511999999999"  # optional — specific owner for this command, takes priority over owner/OWNER_NUMBER
    allowedChats: []        # closed list of chats (groups and DMs) where the command can run
    cooldownSeconds: 5
    whitelist:
      groups: ["120363...@g.us"]
      users: ["5511999999999@c.us"]
    blacklist:
      groups: []
      users: []
  messages:
    senderNotAdmin: "Only admins can use this command."
    botNotAdmin: "I need to be a group admin to do this."
    ownerOnly: "This command is restricted to the bot's owner."
    donoOnly: "This command is restricted to a specific number."
    wrongScope: "This command only works in groups."
    allowedChats: "This command isn't enabled in this chat."
    blacklist: "You can't use this command here."
    cooldown: "Hold on — wait {{seconds}}s before trying again."
```

The check order is fixed: `dono` → `owner` → `scope` → `allowedChats` → `blacklist` →
`whitelist` → bot admin (`botAdmin`) → sender admin (`admin`) → cooldown (only
consumed if all previous checks pass). If `whitelist` and `blacklist` both match at the
same time, **`blacklist` wins**. `dono` (a specific JID/number for this command) takes priority
over the generic `owner` — if `dono` is defined, only it is checked at that step, the bot's
global `OWNER_NUMBER` doesn't even come into play.

The `cooldown` message accepts the placeholders `{{seconds}}` and `{{time}}` — both resolve to
the same value (remaining seconds).

Defaults set in `defaults.permissions`/`defaults.messages` apply to every command that doesn't
override the field; a plugin can also declare its own permission defaults, but whatever is
in `commands.yaml` always wins.

### Alternative (flat) forms of the same fields

Besides the canonical form above, the parser accepts "flattened" alternative names (the
project's own reference YAML uses this variant):

| Canonical form | Equivalent flat form |
|---|---|
| `scope: group` | `group_only: true` |
| `scope: dm` | `dm_only: true` |
| `whitelist: { groups: [...] }` | `whitelist_groups: [...]` (merges with the nested form, if both exist) |
| `blacklist: { users: [...] }` | `blacklist_users: [...]` (same) |
| (no canonical equivalent) | `allowed_chats: [...]` — same field as `allowedChats` |
| (no canonical equivalent) | `hidden_outside_scope: true` — same field as `hiddenOutsideScope`, hides the command from the menu outside its scope |

Declaring `group_only: true` and `dm_only: true` at the same time is invalid — the parser logs a warning
and ignores the scope (as if nothing had been declared).

`defaults`/root also accepts a `permission_messages:` block with its own names, which is
translated internally into the same `messages:` fields above:

```yaml
permission_messages:
  admin_only: "Admins only."        # -> senderNotAdmin AND botNotAdmin
  dono_only: "Owner only."          # -> ownerOnly AND donoOnly
  group_only: "Groups only."        # -> wrongScope
  cooldown: "Wait {{seconds}}s."
  blacklist: "Blocked here."
  allowed_chats: "Not enabled here."
```

It's also possible to override `defaults.notifyChanges`/`notifyPeriodDays`/`notifyMessage` via
equivalent top-level keys: `notify_changes`, `notify_period_days`, `deprecation_message`. If both
forms appear (a `defaults:` block and the top-level key), the top-level key wins — it's a shallow
overlay, no deep merge (same rule as `import:`).

## Required arguments

```yaml
ban:
  cmd: ban
  plugin: many-mod
  arguments:
    - name: target
      type: mention
      required: true
```

`args:` works as a synonym for `arguments:`, and `optional: true` as the inverse of
`required: true` — if neither is given, the argument is optional by default.

Recognized types today: `mention`, `url`, `media_direct`, `media_reply` (`media_direct_or_reply`
is also accepted and silently normalized to `media_reply`), `number`, `duration`,
`choice` (use `choices: [...]` alongside), `boolean`, `quoted_text`, `reply`. If a
`required: true` argument is missing, the kernel responds on its own with a usage hint, generated automatically
from the declared types — the plugin doesn't even run. Example hint generated for the `ban` above:
`!ban @<user>`. Each type has its own notation in the hint (`--name=<a|b|c>` for `choice`,
`--name[=true|false]` for `boolean`, `<n>` for `number`, etc.); optional arguments appear
in brackets. Free-text parsing remains the plugin's responsibility; the kernel only
recognizes these structured forms.

## Subcommands

Each sub-action of a command (e.g. `!sticker remove-background`) is declared as an item of a
**list** `subcommands:` — not as a map:

```yaml
sticker:
  cmd: sticker
  aliases: [f]
  plugin: many-media
  subcommands:
    - cmd: remove-background
      aliases: [rb]
      function: removeBackground   # or functions: [...] — omitted = inherits the parent's function chain
      desc: "Removes the sticker's background"
```

If `subcommands:` isn't a list, the kernel ignores all subcommands of that command (with a
log warning). Each subcommand's `id` isn't chosen by whoever writes the YAML — it's derived
automatically as `<parent-id>::<cmd>` (e.g. `sticker::remove-background`), useful for recognizing
the subcommand in logs. Two subcommands with the same `cmd` (compared case-insensitively)
under the same parent: the second is ignored with a log warning.

A subcommand inherits the parent's `functions` chain when it doesn't declare its own, and inherits the
parent's already-resolved `loading:` when it doesn't declare its own. Permissions follow the usual
logic (own > plugin's default function > `defaults.permissions`), with the parent's `scope`
as a fallback when the subcommand doesn't define its own. Aliases are **not** inherited from the
parent — a subcommand's `aliases:` list is always the one it declares itself (empty, if omitted).

Session/state that's specific to the flow (timeout, received-media history, etc.) continues living
entirely inside the plugin — it's not part of `commands.yaml`.

## Container commands (subcommands with no handler of their own)

A command can exist just to group `subcommands:`, with no `function`/`functions` of its own — useful
when the "parent" command should never be called on its own (e.g. `!todo` with nothing after it doesn't
make sense, only `!todo add`/`!todo list` do):

```yaml
todo:
  cmd: todo
  plugin: many-utils
  subcommands:
    - cmd: add
      function: addItem
    - cmd: list
      function: listItems
```

This type of command is never dispatched directly — the kernel always routes to one of the
`subcommands:`. An unknown subcommand token after `!todo` falls back to the default fallback
("unknown subcommand", with the list of valid subcommands), the same as would happen
with a normal command that has subcommands.

## Automatic deprecation

Renaming a command's `cmd:` (keeping the same id) or removing the key entirely marks the
old name as **deprecated** for a transition period (`notifyPeriodDays`, default 7 days).
The cmd-by-id history, and active deprecations, are persisted to disk (in the same
SQLite database used by the bot's settings) — so this survives process restarts; the warning
window keeps counting from the actual moment of the change, it isn't reset on every boot.
During this window:

- whoever types the old name gets a configurable warning (placeholders `{{old}}`, `{{new}}`,
  `{{days}}` — double braces, same as the rest of ManyBot's i18n system);
- registering a new command that reuses this old name is blocked, to avoid confusion.

```yaml
defaults:
  notifyChanges: true
  notifyPeriodDays: 7
  notifyMessage: "The command {{old}} became {{new}}. This notice goes away in {{days}} days."
```

`notifyChanges` can also be overridden per individual command, as can the message: a
command can declare its own `deprecatedMessage`, which takes priority over
`defaults.notifyMessage` when present. With neither defined, the kernel falls back to a
built-in default text (translated via i18n).

> 🐛 **Legacy to migrate:** the `sticker` command still implements its own deprecation
> manually (not via `commands.yaml`) — this migration hasn't been done yet.

## Menu

```yaml
menu:
  title: "🤖 My Bot — Menu"
  intro: "Use {prefix}<command> to run it or {prefix}help <command> to see the manual."
  footer: "Made with ManyBot"
  cmd: menu
  aliases: [help, man, menu, bot, "?"]
  notFoundFallback: false
  welcomeMessage: "Hi! Type {prefix}menu to see what I do."
  welcomeWindowDays: 3
  pageSize: 15

categories:
  media:
    label: "Media"
    order: 1
  moderation:
    label: "Moderation"
    order: 2
    scope: group           # commands in this category default to scope: group
    hiddenInScope: dm       # category disappears from the menu when viewed in a DM
    loading: default          # loading default for commands in this category with no loading: of their own
```

Note that `{prefix}` in `intro`/`footer`/`welcomeMessage` uses **single** braces — unlike the
`{{old}}`/`{{new}}`/`{{days}}`/`{{seconds}}`/`{{time}}` placeholders in the sections above. Don't
mix the two formats.

`welcomeMessage` is shown automatically to a user on their first message within the
`welcomeWindowDays` window — it replaces the old permanent `seen_users` flag. Two extra
checks prevent a "ghost welcome": the received message needs non-empty text, and it needs
to have arrived at most 60s ago (protects against Baileys' event buffer reclassifying
old receipts as new messages after a reconnection). `notFoundFallback` controls whether the
bot responds with something when a prefixed command doesn't match anything known (even after
legacy plugins have run); since legacy plugins can't signal that they "handled" the message,
this can produce a duplicate response — that's why it's off by default.

`hiddenInScope` (per category) and `hiddenOutsideScope`/`hidden_outside_scope` (per
command/subcommand, inside `permissions:`) are already accounted for by the real menu — a
category disappears entirely from the menu when it's rendered in the scope declared in
`hiddenInScope`, and a command disappears from the menu outside the scope declared in `hiddenOutsideScope`
(except when the resolved scope is `any`).

## Importing other files

`commands.yaml` can import other YAML files, each contributing entire root-level
sections (`menu:`, `categories:`, `manuals:`, or full commands declared at the root):

```yaml
import:
  - menu.yaml
  - manual.yaml
```

`import:` also accepts a single string (one file only), not just a list. Resolution happens
**before** unwrapping any `commands:` in the main file — so an imported
key can collide with either the root or the content inside `commands:`.

There's no deep merge: each root-level key can only be declared in **one** place (the
main file or one of the imports). If two files declare the same key, the first
declaration wins and the second is ignored with a log error — this prevents an import from
silently overwriting something already defined. An `import:` nested inside an imported
file is ignored (not recursive). A missing or malformed import file (read
error, invalid YAML, or a root that isn't an object) logs an error and is skipped, without dropping
the loading of the rest of `commands.yaml`.

## Commands declared by the plugin itself

Besides declaring everything in `commands.yaml`, a plugin can export its own default commands
directly in the code:

```js
export const commands = {
  sticker: {
    cmd: "sticker",
    aliases: ["f", "sticker"],
    desc: "Turns an image/video into a sticker",
    category: "media",
    handler: async (ctx) => { /* ... */ },
  },
};
```

Each entry accepts either a "pure" function (just `handler`, no identity of its own — only
becomes accessible if the user's `commands.yaml` provides the `cmd`) or a full object with
its own identity (`cmd`/`aliases`/`desc`/`category`/`manual`/`permissions` already defined in
the plugin). When the same id also exists in the user's `commands.yaml`, the YAML **always
overrides** what the plugin declared — the same principle already applies today to permissions, and it
applies to the command's entire identity. A `functions: [a, b]` chain declared in the YAML can
chain several entries from that same `commands` map, as long as they all belong to the same plugin.

## Querying commands from inside a plugin

`ctx.commands` lets a plugin check whether a command exists (or read its description/manual) without
needing `ctx.plugins.require()` on the owning plugin — see
[ctx.commands](/docs/api/ctx-commands/) 🧪.

## Still undecided / open

- Whether/when grouping via `group:` (see note above) will be implemented in the menu.
- Dual use/conflict warning when the same plugin function is referenced in `commands.yaml`
  **and** called directly by another plugin via `ctx.plugins.require()`/`.get()` — the approach
  being considered is a runtime check comparing the requested function's name against the set of
  names registered as `function:`/`functions:` in the command registry, but it hasn't been implemented
  or validated yet.
- Mandatory enforcement of "every command must be in `commands.yaml`" is deliberately
  postponed — today it's opt-in, and plugins with no entry at all keep working the old way
  (`ctx.msg.is(...)` inside their own `default()`). This should only become mandatory once the
  YAML actually becomes the primary path.
- Migrating the `sticker` command's manual deprecation to `commands.yaml`'s automatic mechanism
  (see note in the deprecation section above).
