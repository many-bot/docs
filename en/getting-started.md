---
title: Getting started
description: From zero to a ManyBot replying to messages in a few minutes — installation, connection, and first tests.
sidebar:
  order: 1
---

If you just want a simple bot for basic tasks (like downloading videos/audio,
making stickers, etc), we recommend using the official instance, by adding the number:
**+55 16 99459-1903**.

If your goal is to host your own custom bot, this page takes you from
zero to a bot replying to messages in a few minutes.

> ⚠️ Use version **5.9.0** or newer — earlier versions (5.6.x–5.8.x) had
> release management issues.

> If you have any questions, you can contact us via [manybot@pm.me](mailto:manybot@pm.me)
> or in our [WhatsApp](https://chat.whatsapp.com/KfOuIwhpQjN8fcZTMHmaGQ) and [Discord](https://discord.gg/gC7aKChXmA) communities.

## Index

- [What you'll install](#what-youll-install)
- [Linux](#linux-installation)
- [Windows](#windows-installation)
- [Android (Termux)](#android-termux-installation)
- [Connecting the bot](#connecting-the-bot)
- [Getting the bot to respond to something](#getting-the-bot-to-respond-to-something)
- [Common issues (Windows)](#common-issues-windows)

> **FreeBSD** and **macOS** haven't been tested. If you want compatibility with another
> system, open an issue on [GitHub](https://github.com/many-bot/manybot) or
> [Codeberg](https://codeberg.org/many-bot/manybot), or send a suggestion by email to
> [manybot@pm.me](mailto:manybot@pm.me).

---

## What you'll install

Two tools, both via `npm`:

- **[ManyBot](https://www.npmjs.com/package/@manybot/manybot)** — the bot itself. On its own it
  doesn't do anything besides connecting: all functionality (stickers, downloads, moderation, etc.) comes from plugins.
- **[ManyPlug](https://www.npmjs.com/package/@manybot/manyplug)** — the plugin manager, used
  to install and enable what the bot will actually do. It can be called either `manyplug`
  or the shorter alias `mp`.

**Node.js 24 or newer** is required.

---

## Linux installation

Install Node.js and NPM according to your Linux distribution (root access may be required):

| Distro               | Command                           |
| --------------------- | --------------------------------- |
| Debian / Ubuntu        | `sudo apt install nodejs npm`     |
| Fedora                 | `sudo dnf install nodejs npm`     |
| Arch Linux             | `sudo pacman -S nodejs npm`       |
| openSUSE               | `sudo zypper install nodejs npm`  |
| Alpine                 | `sudo apk add nodejs npm`         |
| Void Linux             | `sudo xbps-install -S nodejs npm` |
| Gentoo                 | `sudo emerge nodejs npm`          |

> Check which distribution your system is based on —
> for example, Linux Mint is based on Ubuntu, so the installation command is the same.
>
> If your distro's package installs a Node version below 24, use
> [NodeSource](https://github.com/nodesource/distributions) or [nvm](https://github.com/nvm-sh/nvm)
> to get a newer version.

Confirm the installed version:

```bash
node -v   # must be 24.x or newer
```

---

Check the NPM global prefix and see if you have access to it with:

```bash
npm config get prefix
```

Example output:

```
/usr
```

If the shown directory isn't accessible by your user and requires another user's permissions,
you can change the prefix to something accessible using the command:

```bash
npm config set prefix ~/.npm-global # or another directory
```

Check:

```bash
npm config get prefix
```

Then make sure your PATH includes the new directory.

In Bash:

```bash
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
```

In Zsh:

```zsh
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.zshrc
```

Afterwards, close and reopen your terminal to restart the shell.

---

Now install **ManyBot** and **ManyPlug**:

```bash
npm install -g @manybot/manybot @manybot/manyplug
```

Confirm both were installed:

```bash
npm list -g @manybot/manybot @manybot/manyplug
manyplug --version   # or: mp --version
```

> `manybot` doesn't have a `--version` flag — running it already attempts to connect (see the next section).

---

## Windows installation

Download and install Node.js from the [official site](https://nodejs.org) — get the **latest
LTS** version (24 or higher). During installation, make sure to check the **"Add to PATH"** option.

After installation, open **Command Prompt** or **PowerShell** and check that everything's fine:

```powershell
node -v
npm -v
```

Then install **ManyBot** and **ManyPlug**:

```powershell
npm install -g @manybot/manybot @manybot/manyplug
```

Confirm both were installed:

```powershell
npm list -g @manybot/manybot @manybot/manyplug
manyplug --version
```

---

## Android (Termux) installation

Use [Termux](https://termux.dev/) — install it from [F-Droid](https://f-droid.org/packages/com.termux/)
or from [GitHub](https://github.com/termux/termux-app/releases) (**do not** use the Play
Store version, it's outdated and unsupported).

We recommend using Node.js LTS on Termux, which has more support than its pinned version.

```bash
pkg update && pkg upgrade
pkg install nodejs-lts
```

Confirm the installed version:

```bash
node -v   # must be 24.x or newer
```

Install **ManyBot** and **ManyPlug** normally:

```bash
npm install -g @manybot/manybot @manybot/manyplug
```

### Keeping the bot alive in the background

Android aggressively kills background processes to save battery — this brings
the bot down if you switch apps or turn off the screen without these adjustments:

- Run `termux-wake-lock` before starting the bot, to prevent the system from suspending Termux.
  `termux-wake-unlock` undoes it.
- In Android settings, remove Termux from battery optimization (usually under
  **Settings → Apps → Termux → Battery → Unrestricted**) — the exact path varies by
  manufacturer.
- Consider a process manager like [pm2](https://pm2.keymetrics.io/) (works on
  Termux just like on any Linux) to reconnect automatically if the process dies anyway.

> **Watch out for native dependencies in plugins:** Termux runs on a different architecture and
> libc (Bionic, not glibc) than most Linux distros. npm packages with pre-built native
> binaries (`sqlite3`, `bcrypt`, `sharp`, `canvas`, etc.) often don't have a prebuilt binary
> for that combination and fall back to compiling from scratch — which requires an extra
> build toolchain (`pkg install clang make python`) and, even so, may fail or hang the
> plugin's installation on various devices. See [plugin best practices](/docs/best-practices) before
> choosing dependencies, especially if you plan to publish the plugin for others to use.

---

## Connecting the bot

With everything installed, run it for the first time:

```bash
manybot
```

On the first run, ManyBot:

1. Creates the configuration folder at `~/.manybot/` (on Windows, `C:\Users\YourUser\.manybot\`),
   with the `manybot.toml` file inside — that's where the bot's entire configuration lives (see the
   [full reference](/docs/config)).
2. Asks how you want to connect: **scan a QR Code** or **receive a pairing code**
   on your phone number. This choice is saved, so it's only asked once.
   - **QR Code**: open WhatsApp on your phone, go to **Settings → Linked devices →
     Link a device** and scan the code shown in the terminal.
   - **Pairing code**: enter the number with country code when asked, and type the code
     received on WhatsApp, on the same **Link a device** screen.
3. Once connected, the bot goes online and listens to messages from **any conversation** the
   account is part of — see [`CHATS`](/docs/config#chats--restricting-where-the-bot-responds) if
   you want to restrict it to specific chats.

> Session files are saved at `~/.manybot/sessions/manybot/` (the subfolder name follows
> `CLIENT_ID`, `"manybot"` by default). Don't share this folder with anyone — whoever has access
> to it can control your WhatsApp account.

To keep the bot running in the background (and reconnect automatically if it drops), use a
process manager like [pm2](https://pm2.keymetrics.io/) — this is optional, `manybot` alone already
keeps the connection active while the terminal is open.

---

## Getting the bot to respond to something

On its own, ManyBot has no built-in commands — **everything is a plugin**. With the bot connected (leave
that terminal open), open a second terminal and install something:

```bash
manyplug search sticker
manyplug install <author/plugin-you-found>
```

Or, if you'd rather just test the flow without installing anything third-party, create a sample plugin:

```bash
manyplug init my-test --category utility
cd my-test
manyplug install --local .
```

This installs a plugin that replies `Pong!` to the `!ping` message.

ManyBot detects the change automatically — **no restart needed** — and starts loading the
plugin within a few seconds. Send the corresponding message (e.g. `!ping`, with the default prefix `!`)
in any conversation with the connected number, and the bot should reply.

> `CMD_PREFIX`, `CHATS`, and most other keys in `~/.manybot/manybot.toml` are
> applied automatically when you edit the file, without restarting the bot. The exception is `LANGUAGE`
> (translation language), which requires a restart — see [Configuration](/docs/config) for details on
> each key.

From here:
- See [Configuration](/docs/config) for all `manybot.toml` keys, including how to
  restrict the bot to specific chats.
- See [about plugins](/docs/about-plugins) and the [ManyPlug CLI](/docs/manyplug-cli) to explore
  the manager further.
- See [how to make a plugin](/docs/how-to-make-a-plugin) if you want to develop your own.

## Common issues (Windows)

### "The file C:\Program Files\nodejs\npm.ps1 cannot be loaded because running scripts is disabled on this system."

To fix this you need to set the PowerShell execution policy. Run in PowerShell as administrator:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Type `Y` on the next prompt.
