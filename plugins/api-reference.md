# Referência da API — ManyBot

A API é responsável por montar o objeto `ctx` que cada plugin recebe. Plugins só podem interagir
com o bot através do que está exposto aqui — nunca têm acesso direto ao client do WhatsApp.

Se quiser entender como funciona por baixo, veja
[`pluginApi.js`](https://github.com/many-bot/manybot/blob/master/src/kernel/pluginApi.js).

# Índice

- [Anatomia de um plugin](#anatomia-de-um-plugin)
- [guardOptions](#guardoptions-opcoes-do-pluginguard)
- [O objeto ctx](#o-objeto-ctx)
    - [ctx.config](#ctxconfig)
    - [ctx.i18n](#ctxi18n)
    - [ctx.utils](#ctxutils)
    - [ctx.download](#ctxdownload)
    - [ctx.storage](#ctxstorage)
    - [ctx.plugins](#ctxplugins)
    - [ctx.log](#ctxlog)
    - [ctx.contacts](#ctxcontacts-setup-runtime)
    - [ctx.events](#ctxevents-setup-only)
    - [ctx.send](#ctxsend-setup-parcial-runtime)
    - [ctx.msg](#ctxmsg-runtime-only)
    - [ctx.chat](#ctxchat-runtime-only)
    - [ctx.botId](#ctxbotid)
- [Padrões comuns](#padroes-comuns)

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
  if (ctx.msg.is("oi")) {
    await ctx.send.text("Olá, mundo!");
  }
}
```

O bot passa por todos os plugins ativos para cada mensagem. Cada plugin decide por conta
própria se age ou ignora — não há roteamento central.

---

## guardOptions (opções do pluginGuard)

O módulo do pluginGuard é responsável pela segurança da execução dos plugins, ele tem um
papel importante em fazer o ManyBot não parecer tão suspeito para o WhatsApp e evitar
banimentos precoces.

É possível ativar ou desativar algumas opções do guard para seu plugin, caso ache que seja
rígido demais e sua lógica pode ser executada sem muitas restrições tranquilamente.

Para fazer isso, exporte o objeto `guardOptions` no arquivo principal do seu plugin (ex.
`index.js`):

```js
export const guardOptions = {
    // ...opções
};
```

### Opções:

| Opção      | Tipo      | Padrão | Descrição                                                                                        |
|------------|-----------|--------|--------------------------------------------------------------------------------------------------|
| `timeout`  | `boolean` | `true` | Ativa o timeout de 2 minutos. Se o plugin não encerrar no tempo, é desativado.                   |
| `typing`   | `boolean` | `true` | Ativa o indicador de digitação ("digitando…" ou "gravando audio…") durante a execução do plugin. |
| `cooldown` | `boolean` | `true` | Ativa o intervalo mínimo entre envios para o mesmo chat (anti-detecção).                         |
| `jitter`   | `boolean` | `true` | Ativa o delay aleatório antes do envio para simular comportamento humano.                        |

## Exemplo

```js
export const guardOptions = {
  timeout: false,
  typing:  false,
  cooldown: false,
  jitter:  false,
};
```

> IMPORTANTE: O pluginGuard não impede banimentos no WhatsApp. Ele apenas pode ajudar a mitigar alguns efeitos e reduzir a detecção de
atividades automatizadas em usos casuais. O ManyBot não apoia qualquer uso ilegal ou que infrinja leis ou regras de segurança digital. 
Leia os [Termos de Uso e Política de Privacidade](/docs/terms-and-privacy/) para saber mais.

---

## O objeto `ctx`

O `ctx` tem duas variantes dependendo do ciclo de vida:

### `ctx` no setup

Passado para `plugin.setup(ctx)` durante a inicialização, antes de qualquer mensagem chegar.
Útil para registrar listeners, agendar tarefas periódicas, ou qualquer setup one-time.

**Não contém:** `ctx.msg`, `ctx.chat`, nem os atalhos de envio para o chat atual
(`ctx.send.text`, `ctx.send.image`, etc.). Contém apenas `ctx.send.to()` para enviar a chats
por ID.

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

**Exemplos:**

```js
// Lê o prefixo de comandos definido pelo usuário (ex: "/", "!", ".")
const prefix = ctx.config.get("CMD_PREFIX");

// Lê uma chave opcional com fallback
const lang = ctx.config.get("LANGUAGE", "pt");

// Lê uma API key de serviço externo
const groqKey = ctx.config.get("GROQ_API_KEY");
```

> **Dica:** prefira sempre `ctx.config.get("CHAVE", valorPadrao)` em vez de acessar o CONFIG
> diretamente — assim você garante um fallback se a chave não estiver definida no `.conf` do
> usuário.

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

  if (ctx.msg.is("oi")) {
    // Interpolação de variáveis com {{chave}}
    await ctx.send.text(t("bemVindo", { nome: ctx.msg.senderName }));
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
  if (!ctx.msg.is("video")) return;

  const url = ctx.msg.args[0];
  if (!url) {
    await ctx.msg.reply.text("Informe uma URL.");
    return;
  }

  // Responde imediatamente para o usuário não ficar no vácuo
  await ctx.msg.reply.text("Baixando, aguarde...");

  // O download pesado vai pra fila — o bot continua respondendo outras mensagens enquanto isso
  ctx.download.enqueue(
    async () => {
      const filePath = await baixarVideo(url);
      await ctx.send.video(filePath);
    },
    async (err) => {
      ctx.log.error(`Download falhou: ${err.message}`);
      await ctx.msg.reply.text("Falha no download. Tente novamente.");
    }
  );
}
```

---

## ctx.storage

Acesso ao diretório de dados persistentes do plugin, em `~/.manybot/data/<key>/`. O diretório
é criado automaticamente na primeira vez que o plugin é carregado.

| Propriedade/Método | Assinatura        | Descrição                                                                                                               |
|--------------------|-------------------|-------------------------------------------------------------------------------------------------------------------------|
| `dir`              | `string`          | Caminho absoluto para o diretório de dados do plugin.                                                                   |
| `resolve`          | `(relativePath)`  | Resolve um caminho relativo dentro de `dir`, criando subdiretórios intermediários automaticamente. Retorna o caminho absoluto. |

Use `ctx.storage` para guardar qualquer arquivo que precise persistir entre sessões — banco de
dados SQLite, JSON, cache de mídia, etc.

**Exemplos:**

```js
import { readFileSync, writeFileSync, existsSync } from "fs";

export default async function (ctx) {
  if (!ctx.msg.is("salvar")) return;

  // Resolve o caminho do arquivo dentro do diretório de dados do plugin
  const dbPath = ctx.storage.resolve("dados.json");

  const dados = existsSync(dbPath)
    ? JSON.parse(readFileSync(dbPath, "utf-8"))
    : {};

  dados[ctx.msg.sender] = Date.now();
  writeFileSync(dbPath, JSON.stringify(dados, null, 2));

  await ctx.msg.reply.text("Salvo!");
}
```

```js
// Subdiretórios são criados automaticamente
const cachePath = ctx.storage.resolve("cache/imagens/foto.jpg");
// Equivale a: ~/.manybot/data/autor/meu-plugin/cache/imagens/foto.jpg
// O diretório cache/imagens/ é criado se não existir
```

> O diretório de dados sobrevive a reinstalações do plugin. O `manyplug remove` pergunta antes
> de apagá-lo — e com `-Y` ele remove tudo sem perguntar.

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
    await ctx.send.text("O plugin de IA está ativo!");
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

## ctx.contacts (setup + runtime)

Acesso a informações de contatos do WhatsApp. Disponível em ambos os contextos.

> **Nota sobre `@lid`:** em grupos criados recentemente, o WhatsApp pode retornar IDs no formato
> `@lid` em vez de `@c.us` — um identificador opaco introduzido para preservar a privacidade
> dos usuários. Se você estiver no contexto de uma mensagem, prefira `ctx.msg.getContact()` —
> ele resolve o `@lid` internamente sem chamadas extras.

| Método             | Assinatura    | Descrição                                                                                          |
|--------------------|---------------|----------------------------------------------------------------------------------------------------|
| `get`              | `(contactId)` | Retorna um objeto normalizado com as propriedades do contato, ou `null` se não encontrado.         |
| `getProfilePicUrl` | `(contactId)` | Retorna a URL da foto de perfil, ou `null` se não disponível (privacidade ou contato inexistente). |
| `getAbout`         | `(contactId)` | Retorna o texto "sobre" do contato, ou `null` se não acessível pelas configurações de privacidade. |

### Objeto de contato normalizado

Retornado por `ctx.contacts.get()` e `ctx.msg.getContact()` — sempre a mesma shape.

| Propriedade   | Tipo               | Descrição                                              |
|---------------|--------------------|--------------------------------------------------------|
| `id`          | `string`           | ID serializado (`"5511999999999@c.us"`).               |
| `number`      | `string`           | Número de telefone.                                    |
| `pushname`    | `string \| null`   | Nome público configurado pelo contato no WhatsApp.     |
| `name`        | `string \| null`   | Nome salvo na sua agenda de contatos.                  |
| `shortName`   | `string \| null`   | Versão abreviada do nome salvo.                        |
| `isBusiness`  | `boolean`          | `true` se for uma conta business.                      |
| `isEnterprise`| `boolean`          | `true` se for uma conta enterprise.                    |
| `isBlocked`   | `boolean`          | `true` se você bloqueou esse contato.                  |
| `isMe`        | `boolean`          | `true` se for o próprio bot.                           |
| `isMyContact` | `boolean`          | `true` se o número estiver salvo na sua agenda.        |
| `isWAContact` | `boolean`          | `true` se o número estiver registrado no WhatsApp.     |
| `isUser`      | `boolean`          | `true` se for um contato de usuário (não grupo).       |
| `isGroup`     | `boolean`          | `true` se for um contato de grupo.                     |

**Exemplos:**

```js
const contact = await ctx.contacts.get("5511999999999@c.us");
if (contact) {
  ctx.log.info(`Pushname: ${contact.pushname}`);
  ctx.log.info(`Business: ${contact.isBusiness}`);
}

const picUrl = await ctx.contacts.getProfilePicUrl("5511999999999@c.us");
if (picUrl) await ctx.send.text(`Foto: ${picUrl}`);

const about = await ctx.contacts.getAbout("5511999999999@c.us");
await ctx.send.text(about ?? "Sem descrição.");
```

---

## ctx.events (setup only)

API de eventos do Client do WhatsApp. Disponível **apenas no setup** — registrar listeners
dentro do handler de mensagem criaria um novo listener a cada mensagem recebida.

| Método    | Assinatura         | Descrição                                                                                                                                                    |
|-----------|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `on`      | `(event, handler)` | Registra um listener persistente. Retorna uma função `off()` para cancelar quando quiser.                                                                    |
| `once`    | `(event)`          | Retorna uma `Promise` que resolve na próxima ocorrência do evento.                                                                                           |
| `cleanup` | `()`               | Remove todos os listeners registrados por este plugin. Chamado automaticamente pelo kernel quando o plugin é descarregado ou entra em erro. Raramente chamado diretamente pelo plugin. |

### Eventos disponíveis

| Evento                    | Payload                                                                 | Descrição                                                            |
|---------------------------|-------------------------------------------------------------------------|----------------------------------------------------------------------|
| `auth_failure`            | `string`                                                                | Erro ao restaurar uma sessão existente.                              |
| `authenticated`           | —                                                                       | Autenticação bem-sucedida.                                           |
| `change_battery`          | `{ battery: number, plugged: boolean }`                                 | Bateria do dispositivo mudou. **Deprecated** — não funciona em multi-device. |
| `change_state`            | `WAState`                                                               | Estado da conexão mudou.                                             |
| `chat_archived`           | `(chat, currState: boolean, prevState: boolean)`                        | Um chat foi arquivado ou desarquivado.                               |
| `chat_removed`            | `Chat`                                                                  | Um chat foi removido.                                                |
| `code`                    | `string`                                                                | Pairing code recebido.                                               |
| `contact_changed`         | `(message, oldId: string, newId: string, isContact: boolean)`           | Contato ou participante de grupo mudou de número.                    |
| `disconnected`            | `WAState \| "LOGOUT"`                                                   | Client desconectou.                                                  |
| `group_admin_changed`     | `GroupNotification`                                                     | Alguém foi promovido a admin ou rebaixado.                           |
| `group_join`              | `GroupNotification`                                                     | Alguém entrou num grupo via link ou foi adicionado por admin.        |
| `group_leave`             | `GroupNotification`                                                     | Alguém saiu ou foi removido de um grupo.                             |
| `group_membership_request`| `GroupNotification { chatId, author, timestamp }`                       | Alguém solicitou entrada num grupo com aprovação obrigatória.        |
| `group_update`            | `GroupNotification`                                                     | Assunto, descrição ou foto do grupo foram alterados.                 |
| `incoming_call`           | `{ id, peerJid, isVideo, isGroup, canHandleLocally, outgoing, webClientShouldHandle, participants }` | Chamada recebida.               |
| `media_uploaded`          | `Message`                                                               | Upload de mídia de uma mensagem enviada pelo bot foi concluído.      |
| `message`                 | `Message`                                                               | Nova mensagem recebida. ⚠️ Veja nota abaixo.                         |
| `message_ack`             | `(message: Message, ack: MessageAck)`                                   | Status de entrega/leitura de uma mensagem mudou.                     |
| `message_ciphertext`      | `Message`                                                               | Mensagem recebida ainda não decriptada.                              |
| `message_ciphertext_failed`| `Message`                                                              | Mensagem não conseguiu ser decriptada após tentativa de recuperação. |
| `message_create`          | `Message`                                                               | Mensagem criada, incluindo as do próprio bot. ⚠️ Veja nota abaixo.  |
| `message_edit`            | `(message: Message, newBody: string, prevBody: string)`                 | Uma mensagem foi editada.                                            |
| `message_reaction`        | `{ id, orphan, orphanReason, timestamp, reaction, read, msgId, senderId, ack }` | Reação enviada, recebida, atualizada ou removida.        |
| `message_revoke_everyone` | `(message: Message, revokedMsg: Message \| null)`                       | Uma mensagem foi apagada para todos.                                 |
| `message_revoke_me`       | `Message`                                                               | Uma mensagem foi apagada pelo usuário que a enviou.                  |
| `qr`                      | `string`                                                                | QR code recebido para autenticação.                                  |
| `ready`                   | —                                                                       | Client inicializado e pronto para receber mensagens.                 |
| `vote_update`             | `PollVote`                                                              | Uma opção de enquete foi selecionada ou desmarcada.                  |

> ⚠️ **`message` e `message_create`:** o kernel já consome o evento `message` internamente para
> despachar os plugins. Se você ouvir `message` via `ctx.events.on`, receberá **todas** as
> mensagens — incluindo chats que não estão na lista de permitidos do `.conf`. Use com cuidado
> e filtre manualmente se necessário.

### Exemplos

**Listener persistente com `on`:**

```js
export async function setup(ctx) {
  // Avisa no grupo quando alguém entra
  ctx.events.on("group_join", async (notification) => {
    await ctx.send.to(notification.chatId).text("Bem-vindo! 👋");
  });

  // Para de ouvir depois de 1 hora
  const off = ctx.events.on("message_reaction", (reaction) => {
    ctx.log.info(`Reação de ${reaction.senderId}: ${reaction.reaction}`);
  });
  setTimeout(off, 60 * 60 * 1000);
}
```

**Aguardar um evento com `once`:**

```js
export async function setup(ctx) {
  // Espera o client estar totalmente pronto antes de configurar o resto
  await ctx.events.once("ready");
  ctx.log.success("Client pronto!");

  ctx.events.on("disconnected", (reason) => {
    ctx.log.warn(`Bot desconectou: ${reason}`);
  });
}
```

**Repassar QR code para um canal externo:**

```js
export async function setup(ctx) {
  const ADMIN = ctx.config.get("ADMIN_CHAT_ID");

  ctx.events.on("qr", async (qr) => {
    // qr é uma string — converta para imagem no seu plugin com qrcode ou similar
    ctx.log.info(`Novo QR gerado: ${qr}`);
    await ctx.send.to(ADMIN).text(`Novo QR Code gerado. Acesse o terminal para escanear.`);
  });
}
```

---

## ctx.send (setup parcial + runtime)

API unificada de envio. Sempre namespaced sob `ctx.send` — sem métodos soltos no root do ctx.

> **Envio com comportamento humano:** todos os métodos de envio passam por uma camada de
> anti-detecção automática antes de cada mensagem — rate limit global (máx. 3 msg/s), cooldown
> por chat (mín. 900ms entre envios), jitter aleatório (400–1400ms), e indicador "digitando…"
> ou "gravando áudio…" proporcional ao conteúdo. Esse comportamento é transparente para o
> plugin e não requer configuração. Leve-o em conta ao projetar interações que dependem de
> timing: múltiplos envios em sequência serão espaçados automaticamente.

### ctx.send.to() — setup + runtime

Disponível em ambos os contextos. Retorna um sender bound a qualquer chat por ID, sem depender
de qual chat disparou o evento. Útil para notificações, alertas para grupos de admin, etc.

```js
ctx.send.to(chatId).text(text, opts?)
ctx.send.to(chatId).image(filePath, caption?)
ctx.send.to(chatId).video(filePath, caption?)
ctx.send.to(chatId).audio(filePath, opts?)
ctx.send.to(chatId).sticker(source)
ctx.send.to(chatId).file(filePath, filename?)
```

**Onde encontrar o chatId?** Em runtime é `ctx.chat.id`. Para destinos fixos (grupo de admin,
etc.), guarde o ID num config customizado e leia via `ctx.config.get`.

**Exemplo — notificação no setup:**

```js
const ADMIN_CHAT = "5511999999999@c.us";

export async function setup(ctx) {
  await ctx.send.to(ADMIN_CHAT).text("✅ Bot online!");
}

export default async function (ctx) {
  if (ctx.msg.is("alertar")) {
    await ctx.send.to(ADMIN_CHAT).text(`Alerta de ${ctx.msg.senderName}!`);
    await ctx.send.text("Alerta enviado para o admin.");
  }
}
```

### ctx.send.\* — runtime only

Atalhos para o chat que originou a mensagem atual.

| Método    | Assinatura              | Descrição                                            |
|-----------|-------------------------|------------------------------------------------------|
| `text`    | `(text, opts?)`         | Envia texto para o chat atual.                       |
| `image`   | `(filePath, caption?)`  | Envia imagem para o chat atual.                      |
| `video`   | `(filePath, caption?)`  | Envia vídeo para o chat atual.                       |
| `audio`   | `(filePath, opts?)`     | Envia áudio como mensagem de voz para o chat atual.  |
| `sticker` | `(source)`              | Envia sticker para o chat atual.                     |
| `file`    | `(filePath, filename?)` | Envia arquivo como documento para o chat atual.      |

> **send vs reply:** `ctx.send.text(text)` manda sem quote. `ctx.msg.reply.text(text)` manda
> com quote (citando a mensagem que disparou o handler). Em grupos, prefira `reply` para deixar
> claro a quem o bot está respondendo.

**Exemplos:**

```js
await ctx.send.text("Olá!");
await ctx.send.image("/tmp/foto.jpg", "Aqui está sua imagem.");
await ctx.send.audio("/tmp/audio.ogg");
await ctx.send.sticker("/tmp/sticker.webp");
await ctx.send.file("/tmp/relatorio.pdf");
await ctx.send.file("/tmp/relatorio.pdf", "relatorio-2025.pdf"); // com nome customizado

// Sticker a partir de um Buffer (ex: gerado em memória)
const buf = await gerarSticker();
await ctx.send.sticker(buf);

// Com opções da whatsapp-web.js
await ctx.send.text("https://exemplo.com", { linkPreview: false });
```

### Opções de envio

Aceitas pelos métodos `text` e `to().text`. Correspondem às opções nativas da whatsapp-web.js.

| Opção                 | Tipo       | Padrão  | Descrição                                                                         |
|-----------------------|------------|---------|-----------------------------------------------------------------------------------|
| `linkPreview`         | `boolean`  | `true`  | Exibe preview de links. Sem efeito em contas multi-device.                        |
| `sendSeen`            | `boolean`  | `true`  | Marca a conversa como lida ao enviar.                                             |
| `waitUntilMsgSent`    | `boolean`  | `false` | Aguarda confirmação de envio antes de continuar.                                  |
| `ignoreQuoteErrors`   | `boolean`  | `true`  | Se a mensagem citada não for encontrada, envia sem o quote em vez de lançar erro. |
| `parseVCards`         | `boolean`  | `true`  | Detecta e envia contatos em formato vCard automaticamente.                        |
| `quotedMessageId`     | `string`   | —       | ID da mensagem a ser citada.                                                      |
| `mentions`            | `string[]` | —       | IDs de usuários a mencionar.                                                      |
| `sendMediaAsSticker`  | `boolean`  | `false` | Envia mídia como figurinha. Prefira `ctx.send.sticker`.                           |
| `sendAudioAsVoice`    | `boolean`  | `false` | Envia áudio como mensagem de voz. Prefira `ctx.send.audio`.                       |
| `sendVideoAsGif`      | `boolean`  | `false` | Envia vídeo como GIF.                                                             |
| `sendMediaAsDocument` | `boolean`  | `false` | Envia mídia como documento, sem compressão. Prefira `ctx.send.file`.              |
| `sendMediaAsHd`       | `boolean`  | `false` | Envia imagem em qualidade HD.                                                     |
| `isViewOnce`          | `boolean`  | `false` | Envia foto ou vídeo como visualização única.                                      |
| `caption`             | `string`   | —       | Legenda da imagem ou vídeo.                                                       |

---

## ctx.msg (runtime only)

Contexto da mensagem que disparou o handler.

| Propriedade/Método | Tipo                                    | Descrição                                                                                                                              |
|--------------------|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `body`             | `string`                                | Texto completo da mensagem. String vazia se não houver.                                                                                |
| `type`             | `string`                                | Tipo da mensagem — `"chat"`, `"image"`, `"video"`, `"audio"`, etc.                                                                     |
| `fromMe`           | `boolean`                               | `true` se a mensagem foi enviada pelo próprio bot.                                                                                     |
| `sender`           | `string`                                | ID do remetente — `msg.author` em grupos, `msg.from` em privado.                                                                       |
| `senderName`       | `string`                                | Nome de exibição do contato (notifyName) ou o número limpo caso não disponível.                                                        |
| `command`          | `string`                                | Primeira palavra do body, sem o prefixo configurado em `CMD_PREFIX`. Já em minúsculas.                                                 |
| `args`             | `string[]`                              | Palavras após o comando. `args[0]` é o primeiro argumento.                                                                             |
| `is(cmd)`          | `boolean`                               | Retorna `true` se `command === cmd` (case-insensitive). Principal forma de detectar comandos.                                          |
| `hasMedia`         | `boolean`                               | `true` se a mensagem tem mídia anexada. Cheque antes de chamar `downloadMedia()`.                                                      |
| `isGif`            | `boolean`                               | `true` se a mídia for um GIF.                                                                                                          |
| `downloadMedia()`  | `Promise<{mimetype, data} \| null>`     | Baixa a mídia da mensagem. Retorna objeto com `mimetype` e `data` em base64, ou `null` se falhar.                                      |
| `hasReply`         | `boolean`                               | `true` se a mensagem é uma resposta a outra mensagem.                                                                                  |
| `getReply()`       | `Promise<Message \| null>`              | Retorna a mensagem citada como objeto do whatsapp-web.js, ou `null`.                                                                   |
| `getContact()`     | `Promise<object \| null>`               | Retorna o contato do remetente como objeto normalizado (mesma shape de `ctx.contacts.get()`), resolvendo `@lid` automaticamente.       |
| `reply`            | `sender`                                | Sender com quote na mensagem atual. Mesmos métodos de `ctx.send`: `.text()`, `.image()`, `.video()`, `.audio()`, `.sticker()`, `.file()`. |
| `react(emoji)`     | `Promise`                               | Adiciona uma reação de emoji à mensagem.                                                                                               |
| `hasPrefix`        | `boolean`                               | `true` se a mensagem começa com o CMD_PREFIX configurado no arquivo de configuração                                                    |

### Detectando comandos

`msg.is(cmd)` é a forma idiomática de checar comandos. O prefixo configurado em `CMD_PREFIX`
é descontado automaticamente — você só passa o nome do comando.

```js
// Mensagem recebida: "!ping"  (CMD_PREFIX = "!")
if (ctx.msg.is("ping")) {
  await ctx.send.text("pong!");
}
```

### Lendo argumentos

`msg.args` contém as palavras após o comando — `args[0]` é o primeiro argumento, sem o
comando em si.

```js
// Mensagem: "!video https://youtube.com/watch?v=..."
//   msg.command → "video"
//   msg.args    → ["https://youtube.com/watch?v=..."]

const url = ctx.msg.args[0];
if (!url) {
  await ctx.msg.reply.text("Informe uma URL.");
  return;
}
```

### Respondendo com quote

`ctx.msg.reply` é um sender completo — não é uma função, é um objeto com os mesmos métodos
de `ctx.send`, mas toda mensagem enviada por ele cita a mensagem original.

```js
await ctx.msg.reply.text("Aqui está sua resposta!");
await ctx.msg.reply.image("/tmp/foto.jpg", "Sua imagem.");
await ctx.msg.reply.file("/tmp/doc.pdf");
```

### Baixando mídia

```js
if (ctx.msg.hasMedia) {
  const media = await ctx.msg.downloadMedia();
  if (!media) {
    await ctx.msg.reply.text("Não consegui baixar a mídia.");
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

### Acessando o contato do remetente

`ctx.msg.getContact()` retorna o contato normalizado (mesma shape de `ctx.contacts.get()`).
É a forma recomendada no contexto de uma mensagem porque resolve IDs `@lid` automaticamente,
sem precisar chamar `ctx.contacts.get(ctx.msg.sender)`.

```js
if (ctx.msg.is("perfil")) {
  const contact = await ctx.msg.getContact();
  if (!contact) {
    await ctx.send.text("Não foi possível obter o contato.");
    return;
  }

  const picUrl = await ctx.contacts.getProfilePicUrl(contact.id);
  const about  = await ctx.contacts.getAbout(contact.id);

  await ctx.send.text([
    `*${contact.pushname ?? contact.name ?? "Sem nome"}*`,
    `Número: ${contact.number}`,
    `Sobre: ${about ?? "—"}`,
    `Foto: ${picUrl ?? "—"}`,
  ].join("\n"));
}
```

### Evitando loops com `fromMe`

O bot recebe suas **próprias mensagens** também. Se o seu plugin responde a qualquer mensagem
(não só comandos), filtre para não entrar em loop:

```js
export default async function (ctx) {
  if (ctx.msg.fromMe) return;

  // lógica do plugin...
}
```

---

## ctx.chat (runtime only)

Informações sobre o chat onde a mensagem chegou. O kernel já garantiu que esse chat está na
lista de permitidos do `.conf`.

| Propriedade/Método      | Tipo                                                          | Descrição                                                              |
|-------------------------|---------------------------------------------------------------|------------------------------------------------------------------------|
| `id`                    | `string`                                                      | ID serializado do chat.                                                |
| `name`                  | `string`                                                      | Nome do chat ou grupo. Cai para `id.user` se não tiver nome.           |
| `isGroup`               | `boolean`                                                     | `true` se o chat for um grupo (ID termina em `@g.us`).                 |
| `getParticipants()`     | `Promise<Array<{id, isAdmin, isSuperAdmin}>>`                 | Lista os participantes do grupo. Retorna `[]` em chats privados.       |
| `isAdmin(contactId)`    | `Promise<boolean>`                                            | `true` se o contato for admin ou superadmin. Sempre `false` em privado.|

**Exemplos:**

```js
// Comportamento diferente em grupo vs privado
if (ctx.chat.isGroup) {
  await ctx.send.text(`Olá, grupo *${ctx.chat.name}*!`);
} else {
  await ctx.send.text(`Olá, ${ctx.msg.senderName}!`);
}

// Verificar se quem mandou é admin antes de executar um comando
if (ctx.msg.is("banir")) {
  const admin = await ctx.chat.isAdmin(ctx.msg.sender);
  if (!admin) {
    await ctx.msg.reply.text("Só admins podem usar esse comando.");
    return;
  }
  // ...
}

// Listar participantes
if (ctx.msg.is("participantes")) {
  const lista = await ctx.chat.getParticipants();
  await ctx.send.text(`${lista.length} participantes no grupo.`);
}
```

---

## ctx.botId

`string | null` — ID serializado do próprio bot (`client.info.wid._serialized`). Pode ser
`null` se o client ainda não estiver totalmente pronto quando o plugin inicializar — nesse
caso um warning é emitido no log automaticamente.

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
  if (!ctx.msg.is("oi")) return;

  await ctx.msg.reply.text(`Olá, ${ctx.msg.senderName}!`);
}
```

### Plugin que ignora o bot e mídia

```js
export default async function (ctx) {
  if (ctx.msg.fromMe) return;
  if (ctx.msg.type !== "chat") return; // só texto

  // processa mensagens de texto de outros usuários
}
```

### Plugin com estado por chat (sessões)

```js
const sessoes = new Map();
const TIMEOUT = 2 * 60 * 1000; // 2 minutos

export default async function (ctx) {
  const { msg, chat } = ctx;

  if (msg.is("iniciar")) {
    const timeout = setTimeout(() => sessoes.delete(chat.id), TIMEOUT);
    sessoes.set(chat.id, { autor: msg.sender, timeout });
    await msg.reply.text("Sessão iniciada! Você tem 2 minutos.");
    return;
  }

  if (msg.is("finalizar")) {
    const sessao = sessoes.get(chat.id);
    if (!sessao) {
      await msg.reply.text("Nenhuma sessão ativa.");
      return;
    }
    clearTimeout(sessao.timeout);
    sessoes.delete(chat.id);
    await msg.reply.text("Sessão encerrada.");
  }
}
```

### Plugin com setup e notificação

```js
const ADMIN = "5511999999999@c.us";

export async function setup(ctx) {
  ctx.log.success("meu-plugin carregado.");
  await ctx.send.to(ADMIN).text("Bot online! 🟢");
}

export default async function (ctx) {
  if (ctx.msg.is("status")) {
    await ctx.send.text("Tudo funcionando.");
  }
}
```

### Plugin com dados persistentes

```js
import { readFileSync, writeFileSync, existsSync } from "fs";

export default async function (ctx) {
  if (!ctx.msg.is("pontos")) return;

  const dbPath = ctx.storage.resolve("pontos.json");
  const db = existsSync(dbPath)
    ? JSON.parse(readFileSync(dbPath, "utf-8"))
    : {};

  db[ctx.msg.sender] = (db[ctx.msg.sender] ?? 0) + 1;
  writeFileSync(dbPath, JSON.stringify(db, null, 2));

  await ctx.msg.reply.text(`Você tem ${db[ctx.msg.sender]} ponto(s)!`);
}
```

### Plugin com i18n própria

```js
export default async function (ctx) {
  const { t } = ctx.i18n.createT(import.meta.url);

  if (!ctx.msg.is("ajuda")) return;

  await ctx.send.text(t("ajuda.texto"));
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

### Plugin que restringe a admins

```js
export default async function (ctx) {
  if (!ctx.chat.isGroup) return;
  if (!ctx.msg.is("limpar")) return;

  const admin = await ctx.chat.isAdmin(ctx.msg.sender);
  if (!admin) {
    await ctx.msg.reply.text("Só admins podem usar esse comando.");
    return;
  }

  ctx.utils.emptyFolder("./downloads");
  await ctx.msg.reply.text("Cache limpo.");
}
```
