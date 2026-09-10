---
title: Introduction
description: Official ManyBot documentation — install, configure, develop, maintain, and contribute.
sidebar:
  order: 0
---

This is the official ManyBot documentation, here you will find everything you want to know about the project,
 how to install, how to configure, how to develop, maintain, and how to contribute to the development.

## Collaboration

ManyBot is open-source, under the [GNU General Public License 3.0](https://www.gnu.org/licenses/gpl-3.0.en.html) (GPLv3).
 This means anyone can inspect the code, modify, redistribute, and collaborate.

If you're a developer interested in helping out, below you'll learn how to help with the development of ManyBot and
related projects.

**ManyBot repositories:**

- **ManyBot**: the bot's source code itself (distributed under GPLv3)
    - GitHub: https://github.com/many-bot/manybot
    - Codeberg: https://codeberg.org/many-bot/manybot
    - CGit: https://git.stxerr.dev/manybot.git

- **ManyBot Docs**: all of ManyBot's documentation
    - GitHub: https://github.com/many-bot/docs
    - Codeberg: https://codeberg.org/many-bot/docs
    - CGit: https://git.stxerr.dev/manybot-docs.git

- **ManyBot Website**: the project's official website
    - GitHub: https://github.com/many-bot/website
    - Codeberg: https://codeberg.org/many-bot/website
    - CGit: https://git.stxerr.dev/manybot-website.git

- **ManyPlug CLI**: command-line plugin manager (distributed under MIT)
    - GitHub: https://github.com/many-bot/manyplug
    - Codeberg: https://codeberg.org/many-bot/manyplug
    - CGit: https://git.stxerr.dev/manyplug.git

### How to contribute

It depends on how you like to contribute. Below we'll demonstrate two methods: **pull requests** and **git patches**.

#### Pull requests

The most common way nowadays.

1. Go to our repository on [GitHub](https://github.com/many-bot) or [Codeberg](https://codeberg.org/many-bot).
2. Fork the repository
3. Clone your fork:
```
git clone https://...
```
4. Create a branch:
```
git checkout -b my-fix
```
5. Make your changes.
6. Commit:
```
git add .
git commit -m "Fix issue X"
```
7. Push to your fork:
```
git push origin my-fix
```
8. Open your fork's page on GitHub/Codeberg.
9. Click "Create Pull Request".
10. Choose:
- Base: `manybot/main`
- Compare: `your-fork/my-fix`
11. Write a description and submit.

#### Git patches

Patches are the classic way, still widely used in projects like the *Linux Kernel Organization*.

1. Clone the repository:
```
git clone https://...
```
2. Create a branch:
```
git checkout -b my-fix
```
3. Make your changes and commit:
```
git add .
git commit -m "Fix issue X"
```
4. Generate the patch:
```
git format-patch main --stdout > my-fix.patch
```
5. Send the patch by email to [devel@stxerr.dev](mailto:devel@stxerr.dev).

You can do this directly from the terminal with `git send-email`:
```
git send-email --to=devel@stxerr.dev my-fix.patch
```

> To configure `git send-email`, check your email provider's documentation or use a local SMTP server like `msmtp`.

Received patches will be reviewed and applied with `git am`:
```
git am my-fix.patch
```

## Guides and tutorials

Check out our [YouTube channel](https://youtube.com/@manybotyt).

# Questions?

Get in touch via email (manybot@pm.me) or join our community on [WhatsApp](https://wa.manybot.org) or [Discord](https://discord.gg/gC7aKChXmA).
