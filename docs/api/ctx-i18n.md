---
title: ctx.i18n
description: Traduções — t()/createT() escopado ao plugin, locales por idioma e o atalho ctx.t.
sidebar:
  order: 10
---

```js
export default async function (ctx) {
  const { t } = ctx.i18n.createT(import.meta.url); // t() escopado pro seu plugin

  if (ctx.msg.is("oi")) {
    await ctx.send.text(t("bemVindo", { nome: ctx.msg.senderName }));
  }
}
```

Estrutura esperada:

```
plugins/meu-plugin/
  index.js
  locale/
    pt.json   → { "bemVindo": "Bem-vindo, {{nome}}!" }
    en.json
```

Veja os métodos disponíveis (`t`, `createT`, `reload`, `getCurrentLang`) em
[ctx — opções e assinaturas](/docs/api/ctx-options/#ctxi18n).

> Sem tradução pro idioma configurado → cai pro `en.json` automaticamente.
>
> `ctx.t` é um atalho direto pro mesmo `t` do core (equivalente a `ctx.i18n.t`) — útil se seu
> plugin só precisa das traduções do core e não tem locale próprio.
>
> **`LANGUAGE` não recarrega sozinho:** diferente de outras chaves do `manybot.toml`, o idioma é
> carregado uma única vez por processo. Mudar `LANGUAGE` no arquivo não muda o que `t()`/`ctx.t`
> traduzem em runtime — é preciso chamar `ctx.i18n.reload()` (de algum plugin) ou reiniciar o bot.
