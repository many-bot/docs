---
title: ManyPlug CLI
description: Reference for ManyBot's plugin manager — init, install, link, unlink, search, validate, update, list, enable/disable, remove, info, version.
sidebar:
  order: 5
---

ManyBot's official plugin manager.

```bash
manyplug <command> [options]
# or, shorter:
mp <command> [options]
```

`manyplug` and `mp` are exactly the same command — use whichever you prefer.

---

## init

Creates the skeleton of a new plugin, pluginpack, or profile.

```bash
manyplug init <name>
```

`init` always interactively asks for the **author's name** (used to build the plugin's `key`,
`author/name`) — there's no flag for this. If the `<name>` directory already exists, it asks before
overwriting it.

### Options

- `-c`, `--category <cat>` — plugin category: `integration`, `games`, `media`, `utility`,
  `admin`, `fun`, `moderation`, `ai`, `education`, `social`, `economy`, `automation`, `tools`
  (default: `utility`; an invalid category triggers a warning and falls back to the default)
- `-t`, `--type <type>` — `plugin`, `pluginpack`, or `profile` (default: `plugin`)
- `--lang <lang>` — plugin language: `js` or `ts`. If omitted (or invalid), you're asked
  interactively — pressing Enter without typing anything defaults to `js`. Doesn't apply to `--type profile`
  (profiles have no code).

### Generated structure (plugin, JavaScript)

```
<name>/
├── index.js
├── manyplug.json
├── package.json        # devDependencies: @manybot/types
├── README.md
├── .gitignore
└── locale/
    ├── pt.json
    └── en.json
```

### Generated structure (plugin, TypeScript)

With `--lang ts` (or choosing `ts` at the prompt), the structure changes a bit:

```
<name>/
├── src/
│   └── index.ts
├── manyplug.json      # "main": "dist/index.js"
├── package.json       # includes "scripts.build" and devDependencies: typescript, @manybot/types
├── tsconfig.json
├── README.md
├── .gitignore         # already ignores dist/
└── locale/
    ├── pt.json
    └── en.json
```

ManyBot only loads already-compiled `.js` files — it doesn't transpile TypeScript. Before
running `manyplug install --local .`, run:

```bash
npm install
npm run build
```

This generates `dist/index.js`, which is the file `manyplug.json` points to in `main`. Repeat the build
every time you edit `src/index.ts`.

In both cases (JS or TS), the `@manybot/types` package (added as a `devDependency` by
`init`) describes the shape of `ctx` — import from it (`import('@manybot/types').PluginContext` via
JSDoc, or `import type { PluginContext } from "@manybot/types"` in TS) to get autocomplete in
your editor. See the [API Reference](/docs/api/) to learn what each part of `ctx` does.

### `--type pluginpack` / `--type profile`

With `--type pluginpack`, `init` also generates an `example-plugin/` subfolder with a complete
example plugin (same structure as a regular plugin), showing how to organize the rest. With
`--type profile`, it generates only a `manyplug.json` with `"plugins": []` for you to fill in — no
`index.js`/`package.json`, since profiles have no code. Both come with a `README.md` already
explaining the format.

---

## install

Installs plugins, pluginpacks, or profiles from the [mpindex](https://manybot.stxerr.dev/manyplug/mpindex.json)
or from a local path. Alias: `i`.

```bash
manyplug install <author/plugin> [plugin2...]
manyplug i <author/plugin>
```

```bash
# local
manyplug install --local <path>
```

### Options

- `-l`, `--local <path>` — installs from a local directory with a valid `manyplug.json`
- `-w`, `--watch` — watches for changes and reinstalls automatically (requires `--local`)
- `-b`, `--branch <branch>` — installs from a specific branch
- `-y`, `--yes` — skips confirmation

> Plugins with no `key` in `manyplug.json` are installed under `manydev/<name>`. Add `"key": "author/name"` to avoid this.

Accepts the short name (`manyplug install sticker`) if it's unique in the registry — if more than one
author has a plugin with the same name, the command lists the options and asks for the full key
(`author/sticker`). Installing a plugin whose short name already belongs to a different plugin
(different author) that's already installed is refused, to avoid ambiguity.

The download happens in a temporary directory and is only moved to its final location after being validated —
a canceled or mid-failure installation doesn't leave a broken plugin behind. If the
registry has more than one mirror for the same plugin (e.g. Codeberg and GitHub), `install` automatically tries the
next one if one fails.

### Pluginpacks and profiles

Besides individual plugins, `install` also understands two other package types, identified
by the `"type"` field in `manyplug.json`:

- **pluginpack** — a repository with several plugins inside, each in its own subfolder with
  its own `manyplug.json`. Installing the pack installs each plugin individually — once
  installed, each one works exactly as if it had been installed on its own.
- **profile** — just a list of plugins (`"plugins": ["author/name", ...]`) for ManyPlug to download,
  with no code of its own. Installing a profile fetches each listed plugin from the registry.

Both are created with `manyplug init --type pluginpack` or `--type profile`.

---

## link

Creates a symlink for a local plugin (or pluginpack) inside the plugins folder — like `npm link`.
Edits to the source code take effect immediately, with no need to reinstall. Alias: `ln`.

```bash
manyplug link [path]   # default: .
manyplug ln
```

Pluginpacks link each sub-plugin individually. Profiles have no code of their own to link and
aren't supported by this command — use `install` in that case.

---

## unlink

Undoes a `link`: removes the plugin from the plugins folder, but **doesn't touch** the original
source directory. Alias: `unln`.

```bash
manyplug unlink <plugin> [plugin2...]
manyplug unln my-plugin
```

### Options

- `-y`, `--yes` — skips confirmation

---

## search

Searches the registry for plugins by name, key, category, or description. Alias: `s`.

```bash
manyplug search <query>
manyplug s sticker
```

### Options

- `-c`, `--category <cat>` — filters by category

Plugins already installed show up marked as `[installed]` in the results; pluginpack or profile
entries show up marked as `[pluginpack]`/`[profile]`.

---

## update

Reinstalls plugins whose version in the registry has changed since the one installed — **not** an
unconditional reinstall. With no arguments, checks all installed plugins; accepts specific names.
Alias: `up`.

```bash
manyplug update
manyplug up my-plugin other-plugin
```

### Options

- `-y`, `--yes` — skips confirmation
- `-f`, `--force` — reinstalls even when the local version is already up to date

> Local plugins (installed with `--local`) with no `key` are skipped (warned about separately).

---

## list

Lists installed plugins. By default, shows only the active ones. Alias: `ls`.

```bash
manyplug list
manyplug ls --all
```

### Options

- `-a`, `--all` — includes disabled plugins

The listing shows name, version, category, a **linked** column (`Yes`/`No` — whether the plugin was
installed via `link`), and status — which can be `enabled`, `disabled`, or `incomplete` (the entry
file declared in `main` no longer exists on disk). A `!` before the name flags a corrupted/unreadable
manifest.

---

## enable / disable

Activates or deactivates installed plugins. Accepts multiple names at once. Aliases: `en` and `dis`.

```bash
manyplug enable <plugin> [plugin2...]
manyplug en <plugin>

manyplug disable <plugin> [plugin2...]
manyplug dis <plugin>
```

```bash
# by profile
manyplug enable -p myprofile
manyplug disable --profile myprofile
```

### Options

- `-a`, `--all` — activates/deactivates all installed plugins
- `-p`, `--profile <profile>` — activates/deactivates all plugins installed through this profile.
  Accepts the short name or the full key (`author/profile`); if the short name is ambiguous between
  profiles from different authors, the command lists the options and asks for the full key. Takes priority
  over plugin names and over `--all` — if used together, the other arguments are ignored.

> Only plugins installed **through** the profile (`manyplug install author/profile`) are
> marked as belonging to it — installing the plugin separately afterward doesn't create that marker.

ManyBot detects the change on its own and applies it within seconds — no need to restart the bot.

---

## remove

Removes installed plugins. Offers the option to also delete the plugin's data. Alias: `rm`.

```bash
manyplug remove <plugin> [plugin2...]
manyplug rm <plugin>
```

### Options

- `-y`, `--yes` — skips confirmation for removing the plugin (confirmation for removing the data
  is still asked)
- `-Y` — skips **both** confirmations (plugin and data) on its own, without needing `-y` alongside it

---

## validate

Validates a local plugin, pluginpack, or profile. Alias: `val`.

```bash
manyplug validate [path]   # default: .
```

Checks on `manyplug.json`:
- Required fields (`name`, `version`, `category`) and types of all known fields.
- Unrecognized field in the manifest → warning.
- `name`/`key` follow the format described in [manyplug.json](/docs/how-to-make-a-plugin/#manyplugjson); if `key` is present, its part after the `/` must match `name`.
- `manybotVersion`, if present, is compared against the installed ManyBot version.
- Entry point (`main`) exists on disk.
- Pluginpacks: each subfolder needs its own valid `manyplug.json` — a pack with no
  sub-plugin is an error. Profiles: `plugins` must be a non-empty list of keys.

Dependency checks:
- `package.json` → each declared dependency has a matching folder in `node_modules`
  (warning if missing — run `npm install`).
- `manyplug.json`'s `dependencies` (other plugins) → compared against what the code actually uses via
  `ctx.plugins.require()`: automatically fills in a dependency detected in the code but
  missing from the manifest, and warns about a dependency declared but never used.
- `externalDependencies` → each `command` is looked up in `PATH`. Missing: error if `optional`
  is `false`/omitted (blocks, exits with code 1); warning if `optional: true`.
- The code is also scanned for calls to `exec`/`execSync`/`execFile`/`execFileSync`/`spawn`/
  `spawnSync` with a literal binary (e.g. `execSync("ffmpeg ...")`) — even if that binary isn't
  in `externalDependencies`, `validate` warns if it isn't found in `PATH`.

Checks on `locale/`:
- All language files exist and are valid JSON.
- The same keys exist across all languages — a key present in one file and missing in another
  triggers a warning, to catch forgotten translations.

Checks on `ctx` usage in the code:
- Destructuring (`const { x } = ctx`) and direct access (`ctx.x`, `ctx.x.y`) are compared against the
  API's real keys and methods — a wrong or misspelled name triggers a warning.
- Calling `ctx.send(...)` or `ctx.msg.reply(...)` directly (instead of `ctx.send.text(...)`,
  `ctx.msg.reply.text(...)`, etc.) is detected and flagged.

Any **error** (not a warning) makes the command exit with code 1 — useful for gating CI/publish
hooks.

---

## info

Shows details of an installed plugin: name, key, version, category, author, license, repo, type,
status, entry file (`main`), path on disk, size, data directory (and its size, or
"none"), description, dependencies (other plugins, with version), and external dependencies.

```bash
manyplug info <plugin>
```

Accepts the short name (`my-plugin`) or the full key (`author/my-plugin`) — the same resolution used
by `enable`/`disable`/`remove`/`install`: if the short name is ambiguous between plugins from
different authors, the full key is requested.

---

## version

Shows or updates the version in the current plugin's `manyplug.json`.

```bash
manyplug version           # shows the current version
manyplug version 1.2.0     # updates it to 1.2.0
```

> Can be any string, doesn't need to follow semver. Unlike `validate`, it doesn't accept a
> path — it always operates on the `manyplug.json` of the current directory (`cd` into the plugin's
> folder before running it).

---

## help

```bash
manyplug help
manyplug help <command>
```

Running `manyplug` with no arguments also shows this help. Note that `-h`/`--help` **don't**
work (unlike most CLIs) — use `help` instead.

### `-v`, `--version`

Shows the installed ManyPlug version and exits — works anywhere, doesn't need to be run from
inside a plugin folder (unlike `manyplug version`, which is about the current plugin's
`manyplug.json`).

```bash
manyplug --version
manyplug -v
```

---

## Configuration

On the first run of any command, ManyPlug creates `~/.manybot/manyplug.toml` — this holds the
list of active plugins (managed by `enable`/`disable`) and a few preferences:

| Key         | Default                                               | Description                                              |
|-------------|--------------------------------------------------------|------------------------------------------------------------|
| `LANGUAGE`  | `"auto"`                                              | Interface language. `"auto"` detects it from the system; can be pinned (e.g. `"pt"`). |
| `REGISTRY`  | `https://manybot.stxerr.dev/manyplug/mpindex.json`    | Registry URL used by `install`/`search`/`update`. |
| `CONFIRM`   | `true`                                                | If `false`, skips confirmations that normally require `-y`. |

The language can also be overridden per-command with the `MANYPLUG_LANG` environment variable
(useful for scripts and CI), without needing to edit the file.

> ManyPlug also keeps an internal cache at `~/.manybot/registry.json` (installation
> metadata — what was installed local/linked/via which profile). Not meant to be hand-edited; if it
> looks corrupted or out of date, it's safe to delete — it gets rebuilt on the next operation.
