---
title: About plugins
description: What a ManyBot plugin is, how to discover and install them via ManyPlug, and where plugins live on disk.
sidebar:
  order: 4
---

Every ManyBot feature is an independent plugin. This lets developers build
their own bot functionality without touching the system's core — in practice, plugins
still do the vast majority of the work. (There are a few native kernel commands, such as `ping`,
`status`, `config` - as part of the new experimental
[`commands.yaml`](/docs/commands-yaml/) architecture, but they don't replace plugins; they're just a base
until that option matures.)

## Supported languages

- **JavaScript** — works directly, no extra step
- **TypeScript** — needs to be compiled first (ManyBot only loads `.js`); `manyplug init --lang ts`
  already sets up the build structure for this

The official types package is `@manybot/types`, published on npm — `manyplug init` already adds it
as a plugin `devDependency`, giving autocomplete for `ctx` in both JS (via JSDoc) and TS.
See [how to make a plugin](/docs/how-to-make-a-plugin) for details.

## Download and installation

To download and install plugins, use ManyBot's official tool for it,
[ManyPlug](https://www.npmjs.com/package/@manybot/manyplug) (`manyplug` or `mp`).

To find plugins, use `manyplug search <term>` or browse the
[official index](https://manybot.stxerr.dev/manyplug/mpindex.json).

## Default plugin directory

Once installed, you'll find them at `~/.manybot/plugins/` (or `C:\Users\YourUser\.manybot\plugins`
on Windows), organized by author — e.g. `~/.manybot/plugins/synt-xerror/sticker`.

More information about the ManyPlug CLI on the [next page of this section](/docs/manyplug-cli).
