---
title: ctx.plugins
description: Comunicação entre plugins via API pública
sidebar:
  order: 16
---

Comunicação entre plugins via API pública.

```js
// plugins/meu-banco/index.js
export const api = {
  async buscarUsuario(id) { /* ... */ },
};
```

```js
// consumindo
const banco = ctx.plugins.require("meu-banco");   // lança erro se não existir OU não estiver ativo
const stats = ctx.plugins.get("many-stats");       // null apenas se não existir; ignora status
if (ctx.plugins.exists("many-ai")) { /* feature flag */ }
```

## Diferença entre `get` e `require`

`get` e `require` **não se comportam da mesma forma em relação ao status do plugin**:

| Método    | Plugin não registrado | Plugin registrado, mas não `active`           |
|-----------|-----------------------|-----------------------------------------------|
| `get`     | retorna `null`        | retorna `exports` normalmente (ignora status) |
| `require` | lança erro            | lança erro                                    |
| `exists`  | retorna `false`       | retorna `false`                               |

Ou seja: `get()` só verifica se o plugin existe no registro — se ele estiver com erro, desabilitado, ou qualquer status diferente de `active`, `get()` ainda devolve o `exports` normalmente. Só `require()` e `exists()` exigem `status === "active"`.

Na prática:

- Use `ctx.plugins.exists(name)` antes de `get()` se seu plugin precisa que a dependência esteja realmente ativa (não só presente no registro).
- Prefira `require()` quando a dependência é obrigatória: além de garantir o status `active`, a mensagem de erro já identifica a dependência que falhou.

## Exemplos

### Dependência obrigatória

Se seu plugin não funciona sem outro, use `require()` e deixe o erro propagar, ele já identifica a dependência que falhou.

```js
export default async function (ctx) {
  const banco = ctx.plugins.require("eu/meu-banco");
  const usuario = await banco.buscarUsuario(ctx.msg.senderId);
  ctx.reply(`Olá, ${usuario.nome}`);
}
```

### Dependência opcional, com fallback

Quando o plugin funciona de qualquer forma, mas usa outro se disponível, combine `exists()` + `get()` para garantir que está realmente ativo (não só registrado).

```js
export default async function (ctx) {
  const stats = ctx.plugins.exists("user/stats")
    ? ctx.plugins.get("user/stats")
    : null;

  if (stats) {
    await stats.registrarEvento("comando_usado");
  }

  ctx.reply("Comando executado.");
}
```

### Feature flag

Pra só ligar/desligar um trecho de código conforme a presença de outro plugin, sem precisar do `exports` dele:

```js
export default async function (ctx) {
  if (ctx.plugins.exists("many-ai")) {
    ctx.reply("🤖 resposta gerada por IA...");
  } else {
    ctx.reply("resposta padrão");
  }
}
```

### Cuidado com `get()` sozinho

Sem checar `exists()` antes, `get()` pode devolver o `exports` de um plugin que está em erro. O objeto existe, mas o plugin por trás dele pode não estar funcionando como esperado.

```js
// arriscado: não garante que "meu-banco" está active
const banco = ctx.plugins.get("meu-banco");
if (banco) {
  await banco.buscarUsuario(id); // pode falhar mesmo com banco !== null
}
```

Veja a tabela de métodos (`get`, `require`, `exists`) em
[ctx - opções e assinaturas](/docs/api/ctx-options/#ctxplugins).
