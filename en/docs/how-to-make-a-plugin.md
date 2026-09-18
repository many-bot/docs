---
title: How to make a plugin
description: How to create a ManyBot plugin from scratch — index.js, manyplug.json, package.json, locale, TypeScript, and how to publish it to the official registry.
sidebar:
  order: 6
---

Here you'll learn everything you need to start developing plugins for ManyBot.

If you haven't read [about plugins](/docs/about-plugins) yet, it's recommended you read it before continuing.

> After going through this page, it's also worth reading
> [best practices for plugins](/docs/best-practices) — especially the part about native
> dependencies, which blocks installation for a lot of people on Android/Termux if not avoided.
>
> 🧪 There's also a new **experimental** (still being tested) way to declare a command's identity,
> permissions, and arguments outside the plugin's code: see
> [`commands.yaml`](/docs/commands-yaml/).

## Table of contents

- [Minimal example](#minimal-example)
- [TypeScript & Language Server Protocol (LSP)](#typescript)
- [Standard plugin structure](#standard-plugin-structure)
    - [index.js](#indexjs)
    - [manyplug.json](#manyplugjson)
    - [package.json](#packagejson)
    - [README.md](#readmemd)
    - [locale](#locale)
- [Publishing your plugin](#publishing-your-plugin)

---

## Minimal example

Before getting into the details, here's what a working plugin looks like from start to finish:

```js
// plugins/my-plugin/index.js

export default async function (ctx) {
  const { msg } = ctx;

  // ignore anything that isn't the command
  if (!msg.is("hi")) return;

  await msg.reply.text(`Hello, ${ctx.msg.senderName}!`);
}
```

That's it. A file exporting a `default` function that receives `ctx` — the object with the bot's
entire API. To understand what else is available on `ctx`, see the
[API Reference](/docs/api/).

---

## TypeScript

You can develop plugins with **full autocomplete** using TypeScript or JSDoc — ManyBot
publishes its own types package, `@manybot/types`, and `manyplug init` already sets
everything up for you.

### JavaScript with JSDoc (zero setup)

```js
/**
 * @param {import('@manybot/types/en').PluginContext} ctx
 */
export default async function (ctx) {
  // autocomplete works here ✅
  if (ctx.msg.is("hi")) {
    await ctx.msg.reply.text(`Hello, ${ctx.msg.senderName}!`);
  }
}
```

`manyplug init` already adds `@manybot/types` as a `devDependency` in the generated `package.json` —
run `npm install` in the plugin's directory and autocomplete works right away, with no local
type file needed.

The types package has two languages, choose between `@manybot/types/en` and `@manybot/types/pt`.

### TypeScript (recommended for larger projects)

Have `manyplug init` set up the TypeScript structure for you:

```bash
manyplug init my-plugin --lang ts
```

This generates:

```
my-plugin/
├── src/
│   └── index.ts
├── manyplug.json      # "main": "dist/index.js"
├── package.json       # devDependencies: typescript, @manybot/types
├── tsconfig.json
└── locale/
    ├── pt.json
    └── en.json
```

`src/index.ts` already comes with the type imported from the published package:

```typescript
import type { PluginContext } from "@manybot/types";

export default async function (ctx: PluginContext) {
  if (!ctx.msg.is("hi")) return;

  // full autocomplete for ctx and all its methods ✅
  await ctx.msg.reply.text(`Hello, ${ctx.msg.senderName}!`);
}
```

> **Important:** ManyBot only loads already-compiled `.js` files — it does **not** transpile
> TypeScript on its own. Before installing or publishing, run:
>
> ```bash
> npm install
> npm run build
> ```
>
> This compiles `src/index.ts` into `dist/index.js`, which is the file `main` in
> `manyplug.json` points to. Repeat this every time you edit the source code — and remember to run the build
> before every `manyplug install --local .` during development.

---

```
my-plugin/
├── index.js
├── manyplug.json
├── package.json       # devDependencies: @manybot/types
├── README.md
├── .gitignore
└── locale/
    ├── en.json
    └── pt.json
```

> The `manyplug init` command sets up this structure automatically — and always **asks for the
> author's name** interactively (used to build `key`), as well as the language (`--lang`) if you don't pass
> that flag. See [manyplug init](/docs/manyplug-cli/#init).

---

## index.js

The plugin's main file — this is where execution starts.

You export the `default` function, which the bot calls on every message received. Optionally,
you can also export a `setup` function, which runs once when the bot connects.

```js
// setup: runs once when the bot connects (optional)
export async function setup(ctx) {
  ctx.log.info("my-plugin initialized!");
}

// default: runs on every message received (required)
export default async function (ctx) {
  if (!ctx.msg.is("hello")) return;

  await ctx.msg.reply.text("Hello!");
}
```

Some important points:

- **Every plugin receives every message.** The bot doesn't filter by command before
  calling your plugin — you decide yourself whether to act or ignore it, usually with an `if (!msg.is(...)) return` at the top.
- **Plugins run in sequence.** Every message goes through every active plugin, one by one.
  If your plugin throws an unhandled error, the kernel catches it, logs a warning, and reloads the
  plugin — it stays active and tries again on the next message. Only after **3 consecutive
  failures** is the plugin actually disabled.
- **The bot also receives its own messages.** If your plugin responds to anything
  (not just commands), filter `ctx.msg.fromMe` so you don't end up in a loop:

```js
export default async function (ctx) {
  if (ctx.msg.fromMe) return;

  // plugin logic...
}
```

---

## manyplug.json

The plugin's metadata file. Required to publish to the official registry, recommended
even for private plugins.

```json
{
  "name": "my-plugin",
  "key": "me/my-plugin",
  "version": "1.0.0",
  "description": "My cool plugin for ManyBot.",
  "category": "utility",
  "manybotVersion": ">=5.0.0",
  "author": {
    "name": "me",
    "email": "my@email.com",
    "website": "www.mywebsite.com"
  },
  "license": "MIT",
  "repo": "https://github.com/me/my-plugin.many",
  "main": "index.js",
  "dependencies": {
    "otherauthor/other-plugin": "*"
  },
  "externalDependencies": {
    "ffmpeg": {
      "command": "ffmpeg",
      "optional": false
    }
  }
}
```

### Fields

#### `name` *(required)*
The plugin's name. Only lowercase letters, numbers, and hyphens (`[a-z0-9-]+` — no periods, no
underscores, no start/end rule), between 2 and 50 characters. Must be unique per author.
`manyplug init` already validates this when it asks for the name; `manyplug validate` applies the same
rule afterward.

#### `version` *(required)*
The plugin's current version. Use whatever format you prefer — SemVer, CalVer, doesn't matter.

#### `category` *(required)*
The plugin's category. Possible values:

| Value         | When to use it                                            |
|---------------|-------------------------------------------------------------|
| `utility`     | General-purpose tools                                       |
| `media`       | Downloading, converting, or sending media                   |
| `games`       | Games and turn-based interactions                           |
| `integration` | Integrations with external APIs and services                |
| `admin`       | Group or bot administration tools                           |
| `fun`         | Entertainment with no specific category                     |
| `moderation`  | Anti-spam, content filters, automatic moderation             |
| `ai`          | Integrations with AI/LLM models                              |
| `education`   | Educational tools, dictionaries, translators                 |
| `social`      | Social interaction, profiles, rankings among users            |
| `economy`     | Virtual currencies, stores, points systems                   |
| `automation`  | Automations and event-triggered flows                        |
| `tools`       | Technical utilities (e.g. converters, generators)             |

#### `key`
Global key in the format `author/name`, where `name` follows the same rule as the `name`
field above (and must be identical to it). Used to reference the plugin in `mpindex` and in
other plugins' dependencies. Without it, ManyPlug installs under `manydev/<name>` and
`validate` will complain.

#### `manybotVersion`
Free-form string indicating which ManyBot version(s) the plugin is compatible with (e.g.
`">=5.0.0"`). Optional — if absent, `manyplug validate` skips this check. If present,
`validate` compares it against the installed ManyBot version and warns on incompatibility or if it
can't detect an installation.

#### `description`
Short description of what the plugin does. Shown in the registry listing.

#### `author`
Author information. Only `name` is required. A plain string is also accepted for compatibility.

#### `license`
The plugin's [open-source license](https://opensource.org/licenses) — defines how the code
can be distributed and modified by others.

#### `repo`
Your plugin's Git repository. Should be just one and can be on any forge or service (e.g. GitHub, GitLab).
Not required, only used for identification when published.

#### `main`
Name of the entry-point file. Normally `"index.js"`. If omitted, ManyBot looks for `index.js`.

#### `dependencies`
**Not for npm packages** — that's `package.json` (next section). This field lists **other
ManyBot plugins** that yours uses via [`ctx.plugins.require()`](/docs/api/ctx-plugins/):

```json
{
  "dependencies": {
    "otherauthor/other-plugin": "*"
  }
}
```

`manyplug validate` scans your code for `ctx.plugins.require("key")` calls and:
- If it finds a call to a key that isn't listed here, it **adds it automatically**
  to this field in your `manyplug.json`.
- If a key is listed here but never used in `ctx.plugins.require()`, it warns that the
  dependency looks unused.

ManyPlug **doesn't install these dependencies automatically** — it only warns, during `install` and
`validate`, if a plugin listed here isn't installed. It's up to whoever installs it to run
`manyplug install` for each one.

> Before choosing an **npm** dependency (which goes in `package.json`, not here), especially
> anything that compiles native code (`sqlite3`, `bcrypt`, `sharp`, etc.), it's worth checking the
> [best practices](/docs/best-practices#native-dependencies--be-especially-careful-with-android) —
> those tend to fail to install for people running the bot on Android via Termux.

#### `externalDependencies`
External programs that need to be installed on the system (e.g. `yt-dlp`, `ffmpeg`):

```json
{
  "externalDependencies": {
    "yt-dlp": {
      "command": "yt-dlp",
      "optional": false
    },
    "ffmpeg": {
      "command": "ffmpeg",
      "optional": true
    }
  }
}
```

- `command` — command used to check whether the program is available in `PATH`
- `optional` — if `false` and the program isn't found: `manyplug install` shows a warning but
  installs anyway; `manyplug validate`, on the other hand, treats it as an **error** (blocks, exits with exit
  code 1). If `true`, it's always just a warning in both commands.

---

## package.json

A minimal file. The only required field is `"type": "module"` — needed so that
Node.js treats your files as ESM (the format ManyBot uses):

```json
{
  "type": "module"
}
```

If your plugin has **npm** dependencies (real libraries, not other plugins — that's the
`dependencies` field in `manyplug.json`, previous section), declare them here, the normal npm way:

```json
{
  "type": "module",
  "dependencies": {
    "package-name": "^1.0.0"
  }
}
```

ManyPlug automatically runs `npm install` in this directory when installing your plugin (and also
when using `manyplug link`) — whoever installs it doesn't need to do anything manually.

---

## README.md

A Markdown file explaining your plugin. Not required, but highly recommended —
especially if you plan to publish it. It's what other users will read to understand what
your plugin does and how to configure it.

Prefer writing it in English to reach more people, but Portuguese is also accepted.

Real example (the `manymedia` plugin):

```markdown
# ManyMedia

Download videos and audio from YouTube, Reddit, Instagram, and other yt-dlp supported
sites — either sending the file directly to chat or uploading to a storage server and
sharing the link.

## Features

- **Multi-site support**: YouTube, Reddit, Instagram, SoundCloud, TikTok, and any
  other yt-dlp compatible site
- **Audio extraction**: Downloads and extracts MP3 at best quality
- **Flexible delivery**: Send file directly to chat, or upload to a storage server
  and reply with the link
- **Upload retry**: Failed uploads are retried up to 4 times with a 3-second delay
- **Queued processing**: Downloads run in a queue to prevent resource contention
- **Automatic cleanup**: Temporary files removed after delivery

## Requirements

- `yt-dlp` installed and available in `PATH`
- `ffmpeg` for converting filetypes (e.g. mp4 to mp3) — optional for download only
- `cookies.txt` in the project root (required for YouTube, Reddit, and sites that
  need authentication)

## Usage

    /video https://youtube.com/watch?v=...
    /audio https://youtube.com/watch?v=...

## Configuration

Add to `manybot.toml`:

| Key                | Default | Description                                                    |
|--------------------|---------|------------------------------------------------------------------|
| `UPL_MEDIA_TO_SRV` | `no`    | Set to `yes` to upload to a server and reply with a link       |
| `MEDIA_SRV_API_KEY`| —       | API key for the storage server (required when `UPL_MEDIA_TO_SRV=yes`) |
```

---

## locale

Directory holding your plugin's translations. ManyBot automatically loads the file
matching the language configured in `manybot.toml`.

Structure:

```
locale/
├── en.json
├── pt.json
└── es.json
```

`locale/en.json`:
```json
{
  "hello": "Hello, {{name}}!",
  "error": {
    "generic": "Something went wrong. Try again."
  }
}
```

`locale/pt.json`:
```json
{
  "hello": "Olá, {{name}}!",
  "error": {
    "generic": "Algo deu errado. Tente novamente."
  }
}
```

And in the code:

```js
export default async function (ctx) {
  const prefix = ctx.config.get("CMD_PREFIX");
  const { t }  = ctx.i18n.createT(import.meta.url);

  if (!ctx.msg.is(prefix + "hello")) return;

  // {{name}} is replaced with the value passed in the second argument
  await ctx.msg.reply.text(t("hello", { name: ctx.msg.senderName }));
}
```

> Locales aren't required, but are encouraged. Without a locale, the plugin simply doesn't support multiple languages.

For more details on the i18n API, see [ctx.i18n](/docs/api/ctx-i18n/).

---

## Publishing your plugin

This section only matters if you want other people to use your plugin from the official index.
Entirely optional.

### 1. Validate and test locally

Before anything else, run the validator and install it locally:

```bash
manyplug validate .
manyplug install --local .
```

`validate` checks far more than required fields: types, entry point, `manybotVersion`,
locale, npm and other-plugin dependencies (and can even **fill in** the `dependencies`
field on its own if it detects `ctx.plugins.require()` in your code), external dependencies, and even
incorrect use of `ctx` in your code. See the full list in
[manyplug validate](/docs/manyplug-cli/#validate). Fix everything it flags as an error before
continuing — warnings don't block you, but are worth reviewing. Then confirm the plugin works
by running the bot normally.

### 2. Create a Git repository

Push the code to GitHub, Codeberg, GitLab, or another forge of your choice.

The convention (not required, but encouraged) is to add the `.many` suffix to the
repository name — e.g. `https://codeberg.org/user/my-plugin.many`. It's just for identification.

### 3. Send a request

Send an email to [manybot@pm.me](mailto:manybot@pm.me) with the following format:

```
Subject: [PLUGIN-REQUEST] Your plugin's name

# What does it do?

Brief description of the plugin.

# Repository/repositories

GitHub:   https://github.com/.../...
Codeberg: https://codeberg.org/.../...
```

The email can be in English or Portuguese. Make sure there's an explanatory README in the
repository — that's what will be read during the review.

> If you'd rather not send it by email, you can send it in our
[Discord](https://discord.com/invite/gC7aKChXmA) or [WhatsApp](https://chat.whatsapp.com/KfOuIwhpQjN8fcZTMHmaGQ) community.

### 4. Review and publication

After the review, you'll receive an email with:

- Whether the plugin was accepted or not, and why
- Feedback on the code or documentation
- Any other relevant information

The review is 100% human and anonymous. Don't be afraid to ask questions.

If accepted, the plugin is added to the official index (`mpindex`) and becomes available for installation
via `manyplug install author/name`:

```json
"me/my-plugin": {
  "repos": {
    "codeberg": {
      "master": "https://codeberg.org/me/my-plugin.many",
      "dev":    "https://codeberg.org/me/my-plugin.many"
    },
    "github": {
      "master": "https://github.com/me/my-plugin.many",
      "dev":    "https://github.com/me/my-plugin.many"
    }
  },
  "manifest": "https://raw.githubusercontent.com/me/my-plugin.many/refs/heads/master/manyplug.json",
  "readme": "https://raw.githubusercontent.com/me/my-plugin.many/refs/heads/master/README.md"
}
```
