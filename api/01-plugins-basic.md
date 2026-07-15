# Anatomia de um plugin

```js
// plugins/meu-plugin/index.js

export async function setup(ctx) {
  ctx.log.info("meu-plugin inicializado!");
}

export default async function (ctx) {
  if (ctx.msg.is("oi")) {
    await ctx.send.text("Olá, mundo!");
  }
}
```

- `default` — chamado a cada mensagem recebida num chat permitido.
- `setup` (opcional) — chamado uma vez, na inicialização do bot.
- Sem roteamento central: todo plugin ativo recebe toda mensagem e decide sozinho se age.

Veja a diferença entre `ctx` de setup e de runtime em [ctx overview](/docs/02-ctx-overview/).

---

## guardOptions

Controla o `pluginGuard` — a camada anti-detecção/anti-ban do bot. Exporte `guardOptions` pra
ajustar por plugin:

```js
export const guardOptions = {
  timeout: false,   // desativa o timeout de 2min
  typing:  false,   // desativa indicador "digitando..."
  cooldown: false,  // desativa intervalo mínimo entre envios
  jitter:  false,   // desativa delay aleatório antes do envio
};
```

| Opção      | Padrão | O que faz                                                        |
|------------|--------|--------------------------------------------------------------------|
| `timeout`  | `true` | Desativa o plugin se ele não terminar em 2 minutos.                |
| `typing`   | `true` | Mostra "digitando…"/"gravando áudio…" durante a execução.          |
| `cooldown` | `true` | Intervalo mínimo entre envios pro mesmo chat.                      |
| `jitter`   | `true` | Delay aleatório antes de enviar, pra simular comportamento humano. |

> Isso **não** impede banimento do WhatsApp — só mitiga alguns efeitos de detecção. Veja os
> [Termos de Uso](/docs/terms-and-privacy/).
