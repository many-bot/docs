# ctx.config

```js
const prefix = ctx.config.get("CMD_PREFIX");
const lang   = ctx.config.get("LANGUAGE", "pt"); // com fallback
```

`get(key, defaultValue?)` — lê o `manybot.toml` do usuário. Padrão do 2º argumento: `null`.

---

# ctx.i18n

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

| Método | Assinatura | Descrição |
|---|---|---|
| `t` | `(key, context?)` | Traduz chave dos locales do **core**. |
| `createT` | `(import.meta.url)` | Retorna `{ t, lang }` — `t()` escopado aos locales do plugin, `lang` é o idioma ativo no momento. |
| `reload` | `()` | Recarrega traduções do disco. |
| `getCurrentLang` | `()` | `"pt"`, `"en"`, etc. |

> Sem tradução pro idioma configurado → cai pro `en.json` automaticamente.
>
> `ctx.t` é um atalho direto pro mesmo `t` do core (equivalente a `ctx.i18n.t`) — útil se seu
> plugin só precisa das traduções do core e não tem locale próprio.
>
> **`LANGUAGE` não recarrega sozinho:** diferente de outras chaves do `manybot.toml`, o idioma é
> carregado uma única vez por processo. Mudar `LANGUAGE` no arquivo não muda o que `t()`/`ctx.t`
> traduzem em runtime — é preciso chamar `ctx.i18n.reload()` (de algum plugin) ou reiniciar o bot.

---

# ctx.utils

```js
ctx.utils.emptyFolder(DOWNLOADS_DIR); // apaga conteúdo sem remover a pasta
```

> Só apaga **arquivos diretamente dentro** da pasta — não é recursivo, subpastas (e o conteúdo
> delas) ficam intactas. Lança erro se a pasta não existir; garanta que ela existe antes (ou
> envolva em `try/catch`).

---

# ctx.download

Fila serializada pra downloads pesados — **não** baixe direto no handler, isso trava o event
loop e atrasa outras mensagens.

```js
export default async function (ctx) {
  if (!ctx.msg.is("video")) return;
  const url = ctx.msg.args[0];
  if (!url) return void await ctx.msg.reply.text("Informe uma URL.");

  await ctx.msg.reply.text("Baixando, aguarde...");

  ctx.download.enqueue(
    async () => {
      const filePath = await baixarVideo(url);
      await ctx.send.video(filePath);
    },
    async (err) => {
      ctx.log.error(`Download falhou: ${err.message}`);
      await ctx.msg.reply.text("Falha no download.");
    }
  );
}
```

Só um job roda por vez. `errorFn` (segundo argumento) não é opcional — sempre passe os dois.

---

# ctx.scheduler

Agenda tarefas recorrentes via cron, escopadas ao seu plugin (setup + runtime, mas normalmente
usado no `setup()` — chamar dentro do handler de mensagem registraria uma tarefa nova a cada
mensagem).

```js
export async function setup(ctx) {
  ctx.scheduler.schedule("0 9 * * 1", async () => {
    await ctx.send.to("5511999999999@c.us").text("Bom dia! Relatório semanal:");
  });
}
```

| Método | Assinatura | Descrição |
|---|---|---|
| `schedule(expression, fn)` | `(string, () => Promise<void>) => { stop(): void }` | Registra uma tarefa cron; retorna um handle pra cancelar. |

`expression` segue a sintaxe padrão de cron (minuto, hora, dia do mês, mês, dia da semana). `fn`
roda sem receber `ctx` — feche sobre as variáveis que precisar, como no exemplo acima.

```js
const tarefa = ctx.scheduler.schedule("*/5 * * * *", async () => { /* ... */ });
tarefa.stop(); // cancela — útil se a condição de agendar for dinâmica
```

> Chamar `schedule()` de novo com a **mesma** expressão cron no mesmo plugin **substitui** a
> tarefa anterior, em vez de acumular duas rodando em paralelo — seguro chamar de novo a cada
> hot-reload do plugin. Expressão cron inválida não lança erro: loga um aviso e retorna um handle
> que não faz nada.

---

# ctx.storage

Diretório de dados persistentes do plugin (`~/.manybot/data/<key>/`), criado automaticamente.

```js
import { readFileSync, writeFileSync, existsSync } from "fs";

const dbPath = ctx.storage.resolve("dados.json"); // cria subpastas se precisar
const dados = existsSync(dbPath) ? JSON.parse(readFileSync(dbPath, "utf-8")) : {};
dados[ctx.msg.sender] = Date.now();
writeFileSync(dbPath, JSON.stringify(dados, null, 2));
```

| Prop/Método | Descrição |
|---|---|
| `dir` | Caminho absoluto do diretório de dados. |
| `resolve(relativePath)` | Resolve caminho dentro de `dir`, criando subpastas. |

> Sobrevive a reinstalações. `manyplug remove` pergunta antes de apagar (`-Y` pula tudo).
> `resolve()` rejeita tentativas de escapar do diretório (`../`, caminhos absolutos) — sempre
> retorna um caminho dentro de `dir`.

---

# ctx.settings

Armazenamento de configurações por chat (ou global), persistido em disco — pense em "preferências
que o usuário ajusta pelo próprio WhatsApp", diferente de `ctx.storage` (dados livres do plugin).

```js
// no chat atual
ctx.settings.set("boasVindas", true);
const ativo = ctx.settings.get("boasVindas", false); // com fallback
ctx.settings.getAll();      // todas as chaves deste chat
ctx.settings.delete("boasVindas");
ctx.settings.deleteAll();

// configuração global do bot, não ligada a nenhum chat
ctx.settings.global.set("modoManutencao", true);
ctx.settings.global.get("modoManutencao", false);

// configuração de outro chat específico
ctx.settings.forChat(outroChatId).set("idioma", "en");
```

### Comunidades

Grupos que fazem parte da mesma comunidade do WhatsApp podem compartilhar configurações:

```js
ctx.settings.link(communityId);          // associa o chat atual a uma comunidade
ctx.settings.unlink();                   // remove a associação do chat atual
ctx.settings.getCommunityId();           // string | null
ctx.settings.getCommunityChats();        // string[] — chats associados à mesma comunidade
```

| Método | Descrição |
|---|---|
| `get(key, default?)` / `getAll()` | Lê uma chave (ou todas) do chat atual. Sem 2º argumento, retorna `undefined` (não `null` — diferente de `ctx.config.get()`). |
| `set(key, value)` / `delete(key)` / `deleteAll()` | Escreve/remove no chat atual. `value` precisa ser serializável em JSON. |
| `global` | Mesmos métodos acima (`get`/`set`/`delete`/`getAll`/`deleteAll`), mas escopados ao bot inteiro, sem chat associado. |
| `forChat(chatId)` | Mesmos métodos acima, escopados a outro chat específico. |
| `link(communityId)` / `unlink()` | Associa/desassocia o chat atual a uma comunidade. |
| `getCommunityId()` / `getCommunityChats()` | Consulta a associação atual. |

> No `setup()`, só `ctx.settings.global` está disponível — sem chat atual, os métodos que operam
> no "chat atual" (incluindo `forChat`/`link`/`unlink`) não fazem sentido ali.

---

# ctx.plugins

Comunicação entre plugins via API pública.

```js
// plugins/meu-banco/index.js
export const api = {
  async buscarUsuario(id) { /* ... */ },
};
```

```js
// consumindo
const banco = ctx.plugins.require("meu-banco");   // lança erro se não existir
const stats = ctx.plugins.get("many-stats");       // null se não existir
if (ctx.plugins.exists("many-ai")) { /* feature flag */ }
```

| Método | Descrição |
|---|---|
| `get(name)` | API pública ou `null`. Dependência opcional. |
| `require(name)` | API pública ou lança erro. Dependência obrigatória. |
| `exists(name)` | `true` se o plugin está ativo. |

---

# ctx.log

```js
ctx.log.info("Iniciando processamento...");
ctx.log.warn("API key não configurada");
ctx.log.error(`Falha: ${err.message}`);
ctx.log.success("Sticker enviado!");
```

Prefira `ctx.log` a `console.log` — mantém formato consistente com o resto do bot.

---

# ctx.botId

```js
ctx.log.info(`Bot rodando como: ${ctx.botId}`);
```

`string | null` — pode ser `null` se o client ainda não tiver terminado de inicializar
(emite warning automático no log nesse caso).

---

# ctx.wa (runtime only)

Escape hatch pro socket cru do Baileys (a biblioteca por trás da conexão com o WhatsApp) — pra
quando algo que você precisa não está coberto pelo resto da API.

```js
ctx.wa.sock;    // instância do socket Baileys (@whiskeysockets/baileys) — API completa da lib
ctx.wa.store;   // store interno do ManyBot (cache de contatos/chats)
ctx.wa.msg;     // objeto de mensagem bruto do Baileys que disparou esse handler
await ctx.wa.downloadMedia(); // helper de download que não passa pelas checagens de ctx.msg
```

```js
// exemplo: método do Baileys sem equivalente em ctx.admin/ctx.chat
export default async function (ctx) {
  if (!ctx.msg.is("perfil-de-negocio")) return;
  const perfil = await ctx.wa.sock.getBusinessProfile(ctx.msg.sender);
  await ctx.msg.reply.text(JSON.stringify(perfil ?? "não é conta business"));
}
```

> **Use com cautela.** Diferente do resto do `ctx`, isso não passa pelas normalizações do
> ManyBot (formato de JID, `guardOptions`, tratamento de erro/reload) — bugs aqui não são
> pega pelo mesmo retry/disable de 3 tentativas de um `default()` normal quebrando, e IDs que
> saem direto do `sock`/`msg` vêm no formato nativo do Baileys (`@s.whatsapp.net`, não `@c.us`).
> Prefira sempre a API normal (`ctx.msg`, `ctx.chat`, `ctx.admin`, etc.) quando ela cobrir o que
> você precisa.

`ctx.tg` e `ctx.dc` também existem na interface (reservados pra Telegram e Discord), mas hoje são
sempre `null` — o ManyBot só tem driver de WhatsApp implementado.
