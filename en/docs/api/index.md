---
title: API Reference
description: Overview of the API that ManyBot plugins use to send messages, react to events, and access WhatsApp data.
sidebar:
  order: 0
  label: Overview
---

Plugins work by interacting with the ManyBot API. It exposes a set of resources that
let plugins send and receive messages, access information, and use WhatsApp features
without having to deal directly with the socket or its implementation — and for the rare
case where you need something the API doesn't cover, there's an escape hatch (`ctx.wa`) with direct access to it.

> **TypeScript/JSDoc:** ManyBot publishes its own types package, `@manybot/types`. When you create
> a plugin with `manyplug init`, it's already added as a `devDependency` in the generated `package.json` — it's
> what gives you autocomplete in your editor, both in JS (via JSDoc) and TS. See
> [how to set it up](/docs/how-to-make-a-plugin/#typescript).

1. [Anatomy of a plugin & guardOptions](/docs/api/plugins-basic/)
2. [The ctx object (setup vs runtime)](/docs/api/ctx-overview/)
3. [ctx.send](/docs/api/ctx-send/)
4. [ctx.msg](/docs/api/ctx-msg/)
5. [ctx.chat](/docs/api/ctx-chat/)
6. [ctx.admin](/docs/api/ctx-admin/)
7. [ctx.me](/docs/api/ctx-me/)
8. [ctx.contacts](/docs/api/ctx-contacts/)
9. [ctx.events](/docs/api/ctx-events/)
10. [ctx.poll](/docs/api/ctx-poll/)
11. [ctx.config](/docs/api/ctx-config/)
12. [ctx.i18n](/docs/api/ctx-i18n/)
13. [ctx.utils](/docs/api/ctx-utils/)
14. [ctx.download](/docs/api/ctx-download/)
15. [ctx.scheduler](/docs/api/ctx-scheduler/)
16. [ctx.storage](/docs/api/ctx-storage/)
17. [ctx.settings](/docs/api/ctx-settings/)
18. [ctx.plugins](/docs/api/ctx-plugins/)
19. [ctx.log](/docs/api/ctx-log/)
20. [ctx.botId](/docs/api/ctx-botid/)
21. [ctx.wa](/docs/api/ctx-wa/)
22. [ctx — options and signatures](/docs/api/ctx-options/)
23. 🧪 [ctx.commands](/docs/api/ctx-commands/) — part of the new [`commands.yaml`](/docs/commands-yaml/) architecture, experimental
24. 🧪 [ctx.session](/docs/api/ctx-session/) — same
25. 🧪 [ctx.runCommand](/docs/api/ctx-runcommand/) — same
26. [Common patterns](/docs/api/common-patterns/)

*(The numbering above follows the suggested reading order of the list, not necessarily the order shown in the sidebar.)*
