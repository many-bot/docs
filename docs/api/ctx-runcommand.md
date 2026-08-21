---
title: ctx.runCommand
description: Invoca outro comando registrado passando pelo mesmo pipeline do kernel (permissão, argumentos, subcomando, alerta de crash). Experimental.
sidebar:
  order: 23
  label: ctx.runCommand 🧪
---

> ⚠️ **Experimental — parte da [nova arquitetura `commands.yaml`](/docs/commands-yaml/) 🧪,
> ainda em testes e não 100% funcional. Sujeito a mudar sem aviso em versões `5.x`.**

Invoca outro comando registrado passando pelo **mesmo pipeline** usado pra mensagens reais:
checagem de permissão → roteamento de subcomando → validação de argumentos obrigatórios →
despacho do handler → captura de crash. Runtime only.

```js
const resultado = await ctx.runCommand("figurinha", "https://exemplo.com/imagem.jpg");

switch (resultado.status) {
  case "executed":
    // rodou normalmente
    break;
  case "permission_denied":
  case "argument_missing":
  case "unknown_sub":
    if (resultado.suggestedReply) await ctx.send.text(resultado.suggestedReply);
    break;
  case "no_dispatch":
    // invocation não existe, ou é um comando de texto fixo (sem plugin) — nada foi despachado
    break;
}
```

- `invocation` — o comando ou alias, sem prefixo (ex: `"sticker"`, não `"!sticker"`).
- `rawArgs` — o restante da linha, sem parsing (opcional).

Roda com um `ctx` escopado ao plugin **dono** do comando alvo (`storage`, `plugins`,
`guardOptions` próprios) — não o contexto de quem chamou. Mesmo princípio de
[`ctx.plugins.require()`](/docs/api/ctx-plugins/), mas pra comandos em vez da API pública
exportada por um plugin.

| Campo | Descrição |
|---|---|
| `status` | `"executed"` \| `"permission_denied"` \| `"argument_missing"` \| `"unknown_sub"` \| `"no_dispatch"` |
| `sentReply` | Texto que o próprio kernel já enviou ao chat atual durante o pipeline (ex: aviso de permissão), ou `null`. |
| `suggestedReply` | Texto sugerido pra quem chamou decidir se envia, quando o kernel não enviou nada sozinho, ou `null`. |

> Comandos de texto fixo (`text:`, sem `plugin:`) e invocações desconhecidas resolvem com
> `status: "no_dispatch"` em vez de lançar erro.
