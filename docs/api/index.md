---
title: Referência da API
description: Visão geral da API que plugins do ManyBot usam pra enviar mensagens, reagir a eventos e acessar dados do WhatsApp.
sidebar:
  order: 0
  label: Visão geral
---

Plugins funcionam interagindo com a API do ManyBot. Ela expõe um conjunto de recursos que
permite aos plugins enviar e receber mensagens, acessar informações e utilizar funcionalidades do
WhatsApp sem precisar lidar diretamente com o socket ou sua implementação — e, pro caso raro de
precisar de algo que a API não cobre, existe um escape hatch (`ctx.wa`) com acesso direto a ele.

> **TypeScript/JSDoc:** o ManyBot publica seu próprio pacote de tipos, `@manybot/types`. Ao criar
> um plugin com `manyplug init`, ele já entra como `devDependency` no `package.json` gerado — é
> ele que dá autocomplete no seu editor, tanto em JS (via JSDoc) quanto em TS. Veja
> [como configurar](/docs/how-to-make-a-plugin/#typescript).

1. [Anatomia de um plugin & guardOptions](/docs/api/plugins-basic/)
2. [O objeto ctx (setup vs runtime)](/docs/api/ctx-overview/)
3. [ctx.send & ctx.msg](/docs/api/ctx-messaging/)
4. [ctx.chat, ctx.admin & ctx.me](/docs/api/ctx-chat-admin-me/)
5. [ctx.contacts](/docs/api/ctx-contacts/)
6. [ctx.events](/docs/api/ctx-events/)
7. [ctx.poll](/docs/api/ctx-polls/)
8. [ctx.config, i18n, utils, download, storage, plugins, scheduler, settings, log, botId, ctx.wa](/docs/api/ctx-utilities/)
9. [Padrões comuns](/docs/api/common-patterns/)
