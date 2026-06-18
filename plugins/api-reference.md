# Referência da API — ManyBot

A API é responsável por montar o objeto `ctx` que cada plugin recebe. Plugins só podem interagir
com o bot através do que está exposto aqui — nunca têm acesso direto ao client do WhatsApp.

Se quiser entender como funciona por baixo, veja
[`pluginApi.js`](https://github.com/many-bot/manybot/blob/master/src/kernel/pluginApi.js).

---

## Anatomia de um plugin

Um plugin é um arquivo `index.js` que exporta uma função `default`. Essa função é chamada
**toda vez que uma mensagem chega** num chat permitido. Opcionalmente, você pode exportar uma
função `setup` para rodar código uma única vez na inicialização do bot.

```js
// plugins/meu-plugin/index.js

// setup: executado uma vez quando o bot conecta (opcional)
export async function setup(ctx) {
  ctx.log.info("meu-plugin inicializado!");
}

// default: executado em cada mensagem recebida
export default async function (ctx) {
  if (ctx.msg.is("/oi")) {
    await ctx.send("Olá, mundo!");
  }
}
```

O bot passa por todos os plugins ativos para cada mensagem. Cada plugin decide por conta
própria se age ou ignora — não há roteamento central.

---

## O objeto `ctx`

O `ctx` tem duas variantes dependendo do ciclo de vida:

### `ctx` no setup

Passado para `plugin.setup(ctx)` durante a inicialização, antes de qualquer mensagem chegar.
Útil para registrar listeners, agendar tarefas periódicas, ou qualquer setup one-time.

**Não contém:** `ctx.msg`, `ctx.chat`, `ctx.send`, `ctx.sendImage`, etc. (esses dependem de uma
mensagem ter chegado). Contém os métodos `sendTo` para enviar para chats por ID.

### `ctx` no runtime

Passado para `plugin.default(ctx)` em cada mensagem. Contém tudo do setup mais o contexto
completo da mensagem e os atalhos de envio para o chat atual.

---

## ctx.config

Acesso ao CONFIG global do bot. Os valores vêm do arquivo `~/.manybot/manybot.conf` do usuário,
com defaults aplicados em cima.

| Método | Assinatura             | Descrição                                                                                     |
|--------|------------------------|-----------------------------------------------------------------------------------------------|
| `get`  | `(key, defaultValue?)` | Retorna o valor da chave ou `defaultValue` se ausente. Default do segundo argumento é `null`. |
| `all`  | —                      | Objeto CONFIG completo, somente leitura. Evite mutá-lo.                                       |

**Exemplos:**

```js
// Lê o prefixo de comandos definido pelo usuário (ex: "/", "!", ".")
const prefix = ctx.config.get("CMD_PREFIX");

// Lê uma chave opcional com fallback
const lang = ctx.config.get("LANGUAGE", "pt");

// Lê uma API key de serviço externo
const groqKey = ctx.config.get("GROQ_API_KEY");

// Acessa o config inteiro (somente leitura)
const tudo = ctx.config.all;
```

> **Dica:** prefira sempre `ctx.config.get("CHAVE", valorPadrao)` em vez de acessar
> `ctx.config.all.CHAVE` diretamente — assim você garante um fallback se a chave não estiver
> definida no `.conf` do usuário.

---

## ctx.i18n

API de internacionalização. O bot tem traduções no core, mas plugins podem (e devem) ter suas
próprias pastas de locale se exibirem texto para o usuário.

| Método           | Assinatura          | Descrição                                                                                                        |
|------------------|---------------------|------------------------------------------------------------------------------------------------------------------|
| `t`              | `(key, context?)`   | Traduz uma chave dos locales do core.                                                                            |
| `createT`        | `(import.meta.url)` | Cria uma função `t()` escopada para os locales do próprio plugin. Passe `import.meta.url` do arquivo do plugin. |
| `reload`         | `()`                | Recarrega todas as traduções do disco — útil após troca de idioma em runtime.                                    |
| `getCurrentLang` | `()`                | Retorna o código do idioma atual (`"pt"`, `"en"`, etc.).                                                         |

### Usando traduções no seu plugin

O fluxo recomendado é criar uma função `t()` escopada para o seu plugin via `createT`. Assim
suas strings ficam isoladas das do core e de outros plugins.

Estrutura de pastas esperada:

```
plugins/
  meu-plugin/
    index.js
    locale/
      pt.json
      en.json
```

`locale/pt.json`:
```json
{
  "bemVindo": "Bem-vindo, {{nome}}!",
  "erro": {
    "generico": "Algo deu errado. Tente novamente."
  }
}
```

`index.js`:
```js
export default async function (ctx) {
  // Cria um t() isolado para este plugin
  const { t } = ctx.i18n.createT(import.meta.url);

  if (ctx.msg.is("/oi")) {
    // Interpolação de variáveis com {{chave}}
    await ctx.send(t("bemVindo", { nome: ctx.msg.senderName }));
  }
}
```

> O bot já cuida de carregar o idioma correto baseado no `LANGUAGE` do `.conf`. Se a tradução
> pro idioma configurado não existir, cai automaticamente para o inglês (`en.json`).

---

## ctx.utils

Utilitários de uso geral expostos para plugins.

| Método        | Assinatura | Descrição                                                                                  |
|---------------|------------|--------------------------------------------------------------------------------------------|
| `emptyFolder` | `(folder)` | Apaga o conteúdo de uma pasta sem remover a pasta em si. Útil para limpar caches de mídia. |
| `getChatId`   | `(chat)`   | Retorna o ID serializado de um objeto `chat` (ex: `"5511999999999@c.us"`).                 |

**Exemplos:**

```js
import path from "path";

const DOWNLOADS_DIR = path.resolve("downloads");

export default async function (ctx) {
  // ... faz algum processamento que gera arquivos em DOWNLOADS_DIR ...

  // Limpa a pasta de downloads ao final, sem removê-la
  ctx.utils.emptyFolder(DOWNLOADS_DIR);
}
```

---

## ctx.download

Fila de downloads controlada pelo kernel. Plugins **não devem** fazer downloads pesados
diretamente no handler — isso bloqueia o event loop enquanto o download não terminar, atrasando
todas as outras mensagens. Use essa fila para serializar o trabalho pesado.

| Método    | Assinatura           | Descrição                                                                                  |
|-----------|----------------------|--------------------------------------------------------------------------------------------|
| `enqueue` | `(workFn, errorFn?)` | Adiciona uma função assíncrona à fila. `errorFn` é chamado se `workFn` lançar uma exceção. |

A fila garante que só um job roda por vez, sem travar o bot.

**Exemplo:**

```js
export default async function (ctx) {
  const { msg } = ctx;
  const prefix  = ctx.config.get("CMD_PREFIX");

  if (!msg.is(prefix + "video")) return;

  const url = msg.args[1];
  if (!url) {
    await msg.reply("Informe uma URL.");
    return;
  }

  // Responde imediatamente para o usuário não ficar no vácuo
  await msg.reply("Baixando, aguarde...");

  // O download pesado vai pra fila — o bot continua respondendo outras mensagens enquanto isso
  ctx.download.enqueue(
    async () => {
      // todo o trabalho pesado aqui: download, conversão, envio
      const filePath = await baixarVideo(url);
      await ctx.sendVideo(filePath);
    },
    async (err) => {
      // chamado se o workFn lançar
      ctx.log.error(`Download falhou: ${err.message}`);
      await msg.reply("Falha no download. Tente novamente.");
    }
  );
}
```

---

## ctx.plugins

Permite que plugins se comuniquem entre si via APIs públicas. Um plugin pode expor uma API
exportando uma constante chamada `api`. Outros plugins acessam essa API via `ctx.plugins`.

| Método    | Assinatura | Descrição                                                                                                                    |
|-----------|------------|------------------------------------------------------------------------------------------------------------------------------|
| `get`     | `(name)`   | Retorna a API pública do plugin ou `null` se não estiver ativo. Use quando a dependência é opcional.                         |
| `require` | `(name)`   | Retorna a API pública ou lança um erro se o plugin não existir ou não estiver ativo. Use quando a dependência é obrigatória. |
| `exists`  | `(name)`   | Retorna `true` se o plugin estiver ativo. Útil para feature flags baseadas em plugins instalados.                            |

### Expondo uma API pública

Para outros plugins poderem usar o seu, exporte uma constante `api`:

```js
// plugins/meu-banco/index.js

// API pública que outros plugins podem acessar
export const api = {
  async buscarUsuario(id) {
    // ...
  },
  async salvarDado(id, valor) {
    // ...
  },
};

export default async function (ctx) {
  // lógica normal do plugin
}
```

### Consumindo a API de outro plugin

```js
// plugins/meu-plugin/index.js

export default async function (ctx) {
  // Dependência obrigatória — lança erro se não existir
  const banco = ctx.plugins.require("meu-banco");
  const usuario = await banco.buscarUsuario("123");

  // Dependência opcional — retorna null se não existir
  const stats = ctx.plugins.get("many-stats");
  if (stats) {
    await stats.registrarUso("meu-plugin");
  }

  // Só verifica se está ativo, sem pegar a API
  if (ctx.plugins.exists("many-ai")) {
    await ctx.send("O plugin de IA está ativo!");
  }
}
```

---

## ctx.log

Wrapper sobre o logger interno. Prefira isso a `console.log` — mantém o formato consistente
com o resto do bot e facilita filtrar logs por nível.

| Método             | Quando usar                        |
|--------------------|------------------------------------|
| `info(...args)`    | Informação geral de fluxo.         |
| `warn(...args)`    | Algo inesperado mas não fatal.     |
| `error(...args)`   | Erros que precisam de atenção.     |
| `success(...args)` | Confirmação de operação concluída. |

**Exemplos:**

```js
ctx.log.info("Iniciando processamento...");
ctx.log.warn("API key não configurada, usando modo offline");
ctx.log.error(`Falha ao conectar: ${err.message}`);
ctx.log.success("Sticker enviado com sucesso!");
```

---

## ctx.sendTo (setup + runtime)

Disponível em ambos os contextos. Envia para qualquer chat pelo ID serializado, sem depender
de qual chat disparou o evento. Útil para notificações, alertas para grupos de admin, etc.

| Método          | Assinatura                     | Descrição                                                                                                 |
|-----------------|--------------------------------|-----------------------------------------------------------------------------------------------------------|
| `sendTo`        | `(chatId, text)`               | Envia texto para um chat por ID.                                                                          |
| `sendImageTo`   | `(chatId, filePath, caption?)` | Envia imagem a partir de um caminho no disco.                                                             |
| `sendVideoTo`   | `(chatId, filePath, caption?)` | Envia vídeo a partir de um caminho no disco.                                                              |
| `sendAudioTo`   | `(chatId, filePath)`           | Envia áudio como mensagem de voz.                                                                         |
| `sendStickerTo` | `(chatId, source)`             | Envia sticker. `source` pode ser um caminho (string) ou um Buffer — nesse caso usa mimetype `image/webp`. |

**Onde encontrar o chatId?** Em runtime é `ctx.chat.id`. Para grupos de admin ou outros
destinos fixos, você pode guardar o ID num `.conf` customizado e ler via `ctx.config.get`.

**Exemplo — notificação no setup:**

```js
const ADMIN_CHAT = "5511999999999@c.us";

export async function setup(ctx) {
  // Avisa o admin quando o bot sobe
  await ctx.sendTo(ADMIN_CHAT, "✅ Bot online!");
}

export default async function (ctx) {
  // Em runtime, envia para um chat diferente do atual
  if (ctx.msg.is("/alertar")) {
    await ctx.sendTo(ADMIN_CHAT, `Alerta enviado por ${ctx.msg.senderName}!`);
    await ctx.send("Alerta enviado para o admin.");
  }
}
```

---

## ctx.send (runtime only)

Atalhos para enviar para o chat que originou a mensagem atual. Internamente usam o mesmo
mecanismo dos `sendTo` — são só conveniências para não precisar passar `ctx.chat.id` toda vez.

| Método        | Assinatura             | Descrição                                           |
|---------------|------------------------|-----------------------------------------------------|
| `send`        | `(text, opts?)`        | Envia texto para o chat atual.                      |
| `sendImage`   | `(filePath, caption?)` | Envia imagem para o chat atual.                     |
| `sendVideo`   | `(filePath, caption?)` | Envia vídeo para o chat atual.                      |
| `sendAudio`   | `(filePath)`           | Envia áudio como mensagem de voz para o chat atual. |
| `sendSticker` | `(source)`             | Envia sticker para o chat atual.                    |

> **send vs reply:** `ctx.send(text)` manda sem quote. `ctx.msg.reply(text)` manda com quote
> (citando a mensagem que disparou o handler). Em grupos, prefira `reply` para deixar claro a
> quem o bot está respondendo.

**Exemplos:**

```js
// Texto simples
await ctx.send("Olá!");

// Com quote na mensagem original
await ctx.msg.reply("Recebi sua mensagem.");

// Imagem com legenda
await ctx.sendImage("/tmp/foto.jpg", "Aqui está sua imagem.");

// Áudio como mensagem de voz
await ctx.sendAudio("/tmp/audio.ogg");

// Sticker a partir de um caminho no disco
await ctx.sendSticker("/tmp/sticker.webp");

// Sticker a partir de um Buffer (ex: gerado em memória)
const buf = await gerarSticker();
await ctx.sendSticker(buf);
```

### Opções do ctx.send

Passe um segundo argumento com opções da biblioteca whatsapp-web.js:

```js
ctx.send(texto, { opção: valor })
```

| Opção                 | Tipo             | Padrão  | Descrição                                                                         |
|-----------------------|------------------|---------|-----------------------------------------------------------------------------------|
| `linkPreview`         | `boolean`        | `true`  | Exibe preview de links. Sem efeito em contas multi-device.                        |
| `sendSeen`            | `boolean`        | `true`  | Marca a conversa como lida ao enviar.                                             |
| `waitUntilMsgSent`    | `boolean`        | `false` | Aguarda confirmação de envio antes de continuar.                                  |
| `ignoreQuoteErrors`   | `boolean`        | `true`  | Se a mensagem citada não for encontrada, envia sem o quote em vez de lançar erro. |
| `parseVCards`         | `boolean`        | `true`  | Detecta e envia contatos em formato vCard automaticamente.                        |
| `quotedMessageId`     | `string`         | —       | ID da mensagem a ser citada.                                                      |
| `mentions`            | `string[]`       | —       | IDs de usuários a mencionar.                                                      |
| `sendMediaAsSticker`  | `boolean`        | `false` | Envia mídia como figurinha. Prefira `ctx.sendSticker`.                            |
| `sendAudioAsVoice`    | `boolean`        | `false` | Envia áudio como mensagem de voz. Prefira `ctx.sendAudio`.                        |
| `sendVideoAsGif`      | `boolean`        | `false` | Envia vídeo como GIF.                                                             |
| `sendMediaAsDocument` | `boolean`        | `false` | Envia mídia como documento, sem compressão.                                       |
| `sendMediaAsHd`       | `boolean`        | `false` | Envia imagem em qualidade HD.                                                     |
| `isViewOnce`          | `boolean`        | `false` | Envia foto ou vídeo como visualização única.                                      |
| `caption`             | `string`         | —       | Legenda da imagem ou vídeo.                                                       |

**Exemplo com opções:**

```js
// Envia sem marcar como lido e sem preview de link
await ctx.send("https://exemplo.com", { sendSeen: false, linkPreview: false });

// Envia como documento (sem compressão)
await ctx.send("", { media: minhaMedia, sendMediaAsDocument: true });
```

---

## ctx.msg (runtime only)

Contexto da mensagem que disparou o handler. Tudo que você precisa para inspecionar e reagir.

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
| `reply(text)`      | `Promise`                         | Responde à mensagem com quote.                                                                      |
| `react(emoji)`     | `Promise`                         | Adiciona uma reação de emoji à mensagem.                                                            |

### Detectando comandos

`msg.is(cmd)` é a forma idiomática de checar comandos. A comparação é case-insensitive e
verifica a primeira palavra do body.

```js
const prefix = ctx.config.get("CMD_PREFIX"); // ex: "/"

if (ctx.msg.is(prefix + "ping")) {
  await ctx.send("pong!");
}
```

### Lendo argumentos

`msg.args` é o body splitado por espaços — `args[0]` é o comando, `args[1]` em diante são
os argumentos.

```js
// Mensagem: "/video https://youtube.com/watch?v=..."
const url = ctx.msg.args[1]; // "https://youtube.com/watch?v=..."
if (!url) {
  await ctx.msg.reply("Informe uma URL.");
  return;
}
```

### Baixando mídia

```js
if (ctx.msg.hasMedia) {
  const media = await ctx.msg.downloadMedia();
  if (!media) {
    await ctx.msg.reply("Não consegui baixar a mídia.");
    return;
  }

  // media.mimetype — ex: "image/jpeg", "video/mp4"
  // media.data     — conteúdo em base64
  const buf = Buffer.from(media.data, "base64");
}
```

### Acessando a mensagem citada

```js
if (ctx.msg.hasReply) {
  const quoted = await ctx.msg.getReply();
  if (quoted?.hasMedia) {
    const media = await quoted.downloadMedia();
    // processa a mídia da mensagem citada
  }
}
```

### Evitando loops com `fromMe`

O bot recebe suas **próprias mensagens** também. Se o seu plugin responde a qualquer mensagem
(não só comandos), filtre para não entrar em loop:

```js
export default async function (ctx) {
  if (ctx.msg.fromMe) return; // ignora mensagens do próprio bot

  // lógica do plugin...
}
```

---

## ctx.chat (runtime only)

Informações básicas sobre o chat onde a mensagem chegou. O kernel já garantiu que esse chat
está na lista de permitidos do `.conf`.

| Propriedade | Tipo      | Descrição                                                                  |
|-------------|-----------|----------------------------------------------------------------------------|
| `id`        | `string`  | ID serializado do chat (`"5511999999999@c.us"` ou `"120363...@g.us"`).     |
| `name`      | `string`  | Nome do chat ou grupo. Cai para `id.user` se não tiver nome.               |
| `isGroup`   | `boolean` | `true` se o chat for um grupo (ID termina em `@g.us`).                     |

**Exemplos:**

```js
// Comportamento diferente em grupo vs privado
if (ctx.chat.isGroup) {
  await ctx.send(`Olá, grupo *${ctx.chat.name}*!`);
} else {
  await ctx.send(`Olá, ${ctx.msg.senderName}!`);
}

// Guardar estado por chat (ex: sessões, histórico)
const sessoes = new Map();

export default async function (ctx) {
  const chatId = ctx.chat.id;

  if (!sessoes.has(chatId)) {
    sessoes.set(chatId, { historico: [] });
  }

  sessoes.get(chatId).historico.push(ctx.msg.body);
}
```

---

## ctx.botId

`string | null` — ID serializado do próprio bot (`client.info.wid._serialized`). Pode ser
`null` se o client ainda não estiver totalmente pronto quando o plugin inicializar.

```js
export default async function (ctx) {
  ctx.log.info(`Bot rodando como: ${ctx.botId}`);
}
```

---

## Padrões comuns

### Plugin de comando simples

```js
export default async function (ctx) {
  const prefix = ctx.config.get("CMD_PREFIX");

  if (!ctx.msg.is(prefix + "oi")) return;

  await ctx.msg.reply(`Olá, ${ctx.msg.senderName}!`);
}
```

### Plugin que ignora o bot e mídia

```js
export default async function (ctx) {
  const { msg } = ctx;

  if (msg.fromMe) return;
  if (msg.type !== "chat") return; // só texto

  // processa mensagens de texto de outros usuários
}
```

### Plugin com estado por chat (sessões)

```js
const sessoes = new Map();
const TIMEOUT = 2 * 60 * 1000; // 2 minutos

export default async function (ctx) {
  const { msg } = ctx;
  const prefix  = ctx.config.get("CMD_PREFIX");
  const chatId  = ctx.chat.id;

  if (msg.is(prefix + "iniciar")) {
    const timeout = setTimeout(() => {
      sessoes.delete(chatId);
    }, TIMEOUT);

    sessoes.set(chatId, { autor: msg.sender, timeout });
    await msg.reply("Sessão iniciada! Você tem 2 minutos.");
    return;
  }

  if (msg.is(prefix + "finalizar")) {
    const sessao = sessoes.get(chatId);
    if (!sessao) {
      await msg.reply("Nenhuma sessão ativa.");
      return;
    }
    clearTimeout(sessao.timeout);
    sessoes.delete(chatId);
    await msg.reply("Sessão encerrada.");
  }
}
```

### Plugin com setup e notificação

```js
const ADMIN = "5511999999999@c.us";

export async function setup(ctx) {
  ctx.log.success("meu-plugin carregado.");
  await ctx.sendTo(ADMIN, "Bot online! 🟢");
}

export default async function (ctx) {
  if (ctx.msg.is(ctx.config.get("CMD_PREFIX") + "status")) {
    await ctx.send("Tudo funcionando.");
  }
}
```

### Plugin com i18n própria

```js
export default async function (ctx) {
  const { t }  = ctx.i18n.createT(import.meta.url);
  const prefix = ctx.config.get("CMD_PREFIX");

  if (!ctx.msg.is(prefix + "ajuda")) return;

  await ctx.send(t("ajuda.texto"));
}
```

`locale/pt.json`:
```json
{
  "ajuda": {
    "texto": "Este bot faz X, Y e Z. Use /comando para começar."
  }
}
```

`locale/en.json`:
```json
{
  "ajuda": {
    "texto": "This bot does X, Y and Z. Use /command to start."
  }
}
```
