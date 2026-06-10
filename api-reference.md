# Referência da API

A API é responsável por montar o objeto `ctx` que cada plugin recebe. Plugins só podem interagir
 com o bot através do que está exposto aqui, nunca têm acesso direto ao client do whatsapp-web.js.

Para entender melhor é recomendado inspencionar o arquivo [pluginApi.js](https://github.com/many-bot/manybot/blob/master/src/kernel/pluginApi.js).

O `ctx` tem duas variantes dependendo do ciclo de vida:

---

## `buildSetupApi(client, pluginRegistry)` → ctx (setup)

Passado para `plugin.setup(ctx)` durante a inicialização do bot, antes de qualquer mensagem chegar.
Útil para registrar listeners, agendar tarefas, ou fazer setup one-time. Não tem `msg`, `chat`,
nem os métodos de `send` para o chat atual - apenas os `sendTo` para enviar para um chat por ID específico.

---

## `buildApi({ msg, chat, client, pluginRegistry })` → ctx (runtime)

Passado para `plugin.default(ctx)` toda vez que uma mensagem chega num chat permitido. Contém tudo
do setup mais o contexto completo da mensagem e os métodos de envio para o chat atual.

---

## ctx.config

Acesso ao CONFIG global do bot. Os valores vêm do arquivo `~/.manybot/manybot.conf` do usuário, com defaults aplicados em cima.

| Método | Assinatura             | Descrição                                                                                     |
|--------|------------------------|-----------------------------------------------------------------------------------------------|
| `get`  | `(key, defaultValue?)` | Retorna o valor da chave ou `defaultValue` se ausente. Default do segundo argumento é `null`. |
| `all`  | —                      | Objeto CONFIG completo, somente leitura. Evite mutá-lo.                                       |

---

## ctx.i18n

API de internacionalização. O bot tem traduções no core, mas plugins podem ter suas próprias pastas de locale.

| Método           | Assinatura          | Descrição                                                                                                       |
|------------------|---------------------|-----------------------------------------------------------------------------------------------------------------|
| `t`              | `(key, ...args)`    | Traduz uma chave dos locales do core.                                                                           |
| `createT`        | `(import.meta.url)` | Cria uma função `t()` escopada para os locales do próprio plugin. Passe `import.meta.url` do arquivo do plugin. |
| `reload`         | `()`                | Recarrega todas as traduções do disco - útil após troca de idioma em runtime.                                   |
| `getCurrentLang` | `()`                | Retorna o código do idioma atual (ex: `"pt"`, `"en"`).                                                          |

---

## ctx.utils

Utilitários de uso geral expostos para plugins.

| Método        | Assinatura | Descrição                                                                                  |
|---------------|------------|--------------------------------------------------------------------------------------------|
| `emptyFolder` | `(folder)` | Apaga o conteúdo de uma pasta sem remover a pasta em si. Útil para limpar caches de mídia. |
| `getChatId`   | `(chat)`   | Retorna o ID serializado de um objeto `chat` (ex: `"5511999999999@c.us"`).                 |

---

## ctx.download

Fila de downloads controlada pelo kernel. Plugins não devem fazer downloads pesados diretamente - enfileirem via essa API para não travar o event loop.

| Método    | Assinatura           | Descrição                                                                                  |
|-----------|----------------------|--------------------------------------------------------------------------------------------|
| `enqueue` | `(workFn, errorFn?)` | Adiciona uma função assíncrona à fila de download. `errorFn` é chamado se `workFn` lançar. |

---

## ctx.plugins

Permite que plugins se comuniquem entre si via suas APIs públicas (o que o plugin exporta em `exports`). Análogo a um sistema de DI leve.

| Método    | Assinatura | Descrição                                                                                                                    |
|-----------|------------|------------------------------------------------------------------------------------------------------------------------------|
| `get`     | `(name)`   | Retorna a API pública do plugin ou `null` se não estiver ativo. Use quando a dependência é opcional.                         |
| `require` | `(name)`   | Retorna a API pública ou lança um erro se o plugin não existir ou não estiver ativo. Use quando a dependência é obrigatória. |
| `exists`  | `(name)`   | Retorna `true` se o plugin estiver ativo. Útil para feature flags baseadas em plugins instalados.                            |

---

## ctx.log

Wrapper sobre o logger interno. Preferir isso a `console.log` para manter o formato de log consistente com o resto do bot.

| Método             | Quando usar                        |
|--------------------|------------------------------------|
| `info(...args)`    | Informação geral de fluxo.         |
| `warn(...args)`    | Algo inesperado mas não fatal.     |
| `error(...args)`   | Erros que precisam de atenção.     |
| `success(...args)` | Confirmação de operação concluída. |

---

## ctx.sendTo (setup + runtime)

Disponível em ambos os contextos. Envia para qualquer chat pelo ID serializado, independente de qual chat disparou o evento. Útil para notificações, alertas para grupos de admin, etc.

| Método          | Assinatura                     | Descrição                                                                                                 |
|-----------------|--------------------------------|-----------------------------------------------------------------------------------------------------------|
| `sendTo`        | `(chatId, text)`               | Envia texto para um chat por ID.                                                                          |
| `sendImageTo`   | `(chatId, filePath, caption?)` | Envia imagem a partir de um caminho no disco.                                                             |
| `sendVideoTo`   | `(chatId, filePath, caption?)` | Envia vídeo a partir de um caminho no disco.                                                              |
| `sendAudioTo`   | `(chatId, filePath)`           | Envia áudio como mensagem de voz.                                                                         |
| `sendStickerTo` | `(chatId, source)`             | Envia sticker. `source` pode ser um caminho (string) ou um Buffer — nesse caso usa mimetype `image/webp`. |

---

## ctx.send (runtime only)

Atalhos para enviar para o chat que originou a mensagem atual. Internamente usam o mesmo `makeSender` que os `sendTo`.

| Método        | Assinatura             | Descrição                                           |
|---------------|------------------------|-----------------------------------------------------|
| `send`        | `(text)`               | Envia texto para o chat atual.                      |
| `sendImage`   | `(filePath, caption?)` | Envia imagem para o chat atual.                     |
| `sendVideo`   | `(filePath, caption?)` | Envia vídeo para o chat atual.                      |
| `sendAudio`   | `(filePath)`           | Envia áudio como mensagem de voz para o chat atual. |
| `sendSticker` | `(source)`             | Envia sticker para o chat atual.                    |

> Para responder com quote use `ctx.msg.reply(text)` em vez de `ctx.send(text)`.

---

## ctx.msg (runtime only)

Contexto da mensagem que disparou o handler. Tudo que você precisa para inspecionar e reagir a uma mensagem.

| Propriedade/Método | Tipo                              | Descrição                                                                                           |
|--------------------|-----------------------------------|-----------------------------------------------------------------------------------------------------|
| `body`             | `string`                          | Texto da mensagem. String vazia se não houver.                                                      |
| `type`             | `string`                          | Tipo da mensagem — `"chat"`, `"image"`, `"video"`, `"audio"`, etc.                                  |
| `fromMe`           | `boolean`                         | `true` se a mensagem foi enviada pelo próprio bot.                                                  |
| `sender`           | `string`                          | ID do remetente — `msg.author` em grupos, `msg.from` em privado.                                    |
| `senderName`       | `string`                          | Nome de exibição do contato (notifyName) ou o número limpo caso não disponível.                     |
| `args`             | `string[]`                        | `body` splitado por whitespace. Útil para parsear argumentos de comando.                            |
| `is(cmd)`          | `boolean`                         | Retorna `true` se o body começa com `cmd` (case-insensitive). Principal forma de detectar comandos. |
| `hasMedia`         | `boolean`                         | `true` se a mensagem tem mídia anexada. Cheque antes de chamar `downloadMedia()`.                   |
| `isGif`            | `boolean`                         | `true` se a mídia for um GIF.                                                                       |
| `downloadMedia()`  | `Promise<{mimetype, data}\|null>` | Baixa a mídia da mensagem. Retorna objeto com `mimetype` e `data` em base64, ou `null` se falhar.   |
| `hasReply`         | `boolean`                         | `true` se a mensagem é uma resposta a outra mensagem.                                               |
| `getReply()`       | `Promise<Message\|null>`          | Retorna a mensagem citada como objeto do whatsapp-web.js, ou `null`.                                |
| `reply(text)`      | `Promise`                         | Responde à mensagem com quote. Diferente de `ctx.send()`, que manda sem quote.                      |
| `react(emoji)`     | `Promise`                         | Adiciona uma reação de emoji à mensagem.                                                            |

---

## ctx.chat (runtime only)

Informações básicas sobre o chat onde a mensagem chegou. O kernel já garantiu que esse chat está na lista de permitidos.

| Propriedade | Tipo      | Descrição                                                                  |
|-------------|-----------|----------------------------------------------------------------------------|
| `id`        | `string`  | ID serializado do chat (ex: `"5511999999999@c.us"` ou `"120363...@g.us"`). |
| `name`      | `string`  | Nome do chat ou grupo. Cai pro `id.user` se não tiver nome.                |
| `isGroup`   | `boolean` | `true` se o chat for um grupo (ID termina em `@g.us`).                     |

---

## ctx.botId

`string | null` — ID serializado do próprio bot (`client.info.wid._serialized`). Pode ser `null` se o client ainda não estiver totalmente pronto quando o plugin inicializar.
