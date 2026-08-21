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
3. [ctx.send](/docs/api/ctx-send/)
4. [ctx.msg](/docs/api/ctx-msg/)
5. [ctx.chat](/docs/api/ctx-chat/)
6. [ctx.admin](/docs/api/ctx-admin/)
7. [ctx.me](/docs/api/ctx-me/)
8. [ctx.contacts](/docs/api/ctx-contacts/)
9. [ctx.events](/docs/api/ctx-events/)
10. [ctx.poll](/docs/api/ctx-polls/)
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
22. [ctx — opções e assinaturas](/docs/api/ctx-options/)
23. 🧪 [ctx.commands](/docs/api/ctx-commands/) — parte da nova arquitetura [`commands.yaml`](/docs/commands-yaml/), experimental
24. 🧪 [ctx.session](/docs/api/ctx-session/) — idem
25. 🧪 [ctx.runCommand](/docs/api/ctx-runcommand/) — idem
26. [Padrões comuns](/docs/api/common-patterns/)
