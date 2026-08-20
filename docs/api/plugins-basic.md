---
title: Anatomia de um plugin
description: A função default, a função setup, e a camada anti-detecção/anti-ban (guardOptions) que envolve cada plugin.
sidebar:
  order: 1
---

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

Veja a diferença entre `ctx` de setup e de runtime em [ctx overview](/docs/api/ctx-overview/).

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
| `timeout`  | `true` | Interrompe o plugin se ele não terminar em 2 minutos — tratado como o erro abaixo. |
| `typing`   | `true` | Mostra "digitando…" enquanto o plugin processa a mensagem.         |
| `cooldown` | `true` | Intervalo mínimo entre envios pro mesmo chat.                      |
| `jitter`   | `true` | Delay aleatório antes de enviar, pra simular comportamento humano. |

> **Erro não tratado (ou `timeout: true` estourando):** o kernel captura, loga um aviso e
> **recarrega o plugin** — ele continua ativo e tenta de novo na próxima mensagem. Só depois de
> **3 falhas seguidas** o plugin é desativado de verdade. Ou seja, um erro isolado não tira seu
> plugin do ar; falhar toda vez, sim.

> `typing: false` só desliga o indicador contínuo acima — um "digitando…" breve e proporcional
> ao tamanho da mensagem ainda aparece em cada envio (`ctx.send`/`ctx.msg.reply`), independente
> dessa opção. Hoje esse indicador é sempre "digitando…", mesmo enviando áudio — não existe um
> indicador distinto de "gravando áudio…" implementado ainda.

> Isso **não** impede banimento do WhatsApp — só mitiga alguns efeitos de detecção. Veja os
> [Termos de Uso](/docs/terms-and-privacy/).
