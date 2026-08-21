---
title: ctx.commands
description: Consulta somente-leitura ao registro central de comandos — exists, desc, manual, list, isMenuAlias. Experimental.
sidebar:
  order: 21
  label: ctx.commands 🧪
---

> ⚠️ **Experimental — parte da [nova arquitetura `commands.yaml`](/docs/commands-yaml/) 🧪,
> ainda em testes e não 100% funcional. Sujeito a mudar sem aviso em versões `5.x`.**

Consulta somente-leitura ao registro central de comandos (setup + runtime). Deixa um plugin
checar se outro comando existe, ou ler sua descrição/manual, sem precisar de
`ctx.plugins.require()` do plugin dono — pensado principalmente pra plugins de IA/menu, que
citam comandos e não podem simplesmente inventar (alucinar) se eles existem ou não.

```js
if (ctx.commands.exists("figurinha")) {
  await ctx.send.text("Sim, esse comando existe!");
}

const descricao = ctx.commands.desc("figurinha");        // string | null, no idioma atual
const descricaoEn = ctx.commands.desc("figurinha", "en"); // string | null, forçando idioma

const manual = ctx.commands.manual("figurinha"); // cai pra desc se não houver manual

const todos = ctx.commands.list(); // todo comando de topo registrado, um item por id estável
// [{ id, cmd, aliases, category, desc }, ...]

ctx.commands.isMenuAlias("ajuda"); // true se "ajuda" é um dos aliases do próprio comando de menu
```

> As consultas usam exatamente a palavra (`cmd` ou alias) como foi declarada no
> [`commands.yaml`](/docs/commands-yaml/) — sem normalização implícita de maiúsculas/minúsculas.

> `list()` devolve só comandos de **topo** (um item por id estável) — não expande subcomandos
> individualmente.
