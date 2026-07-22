# Referência da API

Plugins funcionam interagindo com a API do ManyBot. Ela expõe um conjunto de recursos que
permite aos plugins enviar e receber mensagens, acessar informações e utilizar funcionalidades do
WhatsApp sem precisar lidar diretamente com o socket ou sua implementação — e, pro caso raro de
precisar de algo que a API não cobre, existe um escape hatch (`ctx.wa`) com acesso direto a ele.

> **TypeScript/JSDoc:** o ManyBot publica seu próprio pacote de tipos, `@manybot/types`. Ao criar
> um plugin com `manyplug init`, ele já entra como `devDependency` no `package.json` gerado — é
> ele que dá autocomplete no seu editor, tanto em JS (via JSDoc) quanto em TS. Veja
> [como configurar](/docs/how-to-make-a-plugin/#typescript).

1. [Anatomia de um plugin & guardOptions](/docs/01-plugins-basic/)
2. [O objeto ctx (setup vs runtime)](/docs/02-ctx-overview/)
3. [ctx.send & ctx.msg](/docs/03-ctx-messaging/)
4. [ctx.chat, ctx.admin & ctx.me](/docs/04-ctx-chat-admin-me/)
5. [ctx.contacts](/docs/05-ctx-contacts/)
6. [ctx.events](/docs/06-ctx-events/)
7. [ctx.poll](/docs/07-ctx-polls/)
8. [ctx.config, i18n, utils, download, storage, plugins, scheduler, settings, log, botId, ctx.wa](/docs/08-ctx-utilities/)
9. [Padrões comuns](/docs/09-common-patterns/)
