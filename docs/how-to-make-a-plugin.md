---
title: Como fazer um plugin
description: Como criar um plugin pro ManyBot do zero — index.js, manyplug.json, package.json, locale, TypeScript, e como publicar no registry oficial.
sidebar:
  order: 6
---

Aqui você vai aprender tudo que precisa para começar a desenvolver plugins para o ManyBot.

Caso ainda não leu [sobre os plugins](/docs/about-plugins), é recomendado ler antes de continuar.

> Depois de passar por essa página, vale a pena ler também
> [boas práticas em plugins](/docs/best-practices) — principalmente a parte sobre dependências
> nativas, que trava a instalação em bastante gente no Android/Termux se não for evitada.

## Índice

- [Exemplo mínimo](#exemplo-minimo)
- [TypeScript & Language Server Protocol (LSP)](#typescript)
- [Estrutura padrão de um plugin](#estrutura-padrao-de-um-plugin)
    - [index.js](#indexjs)
    - [manyplug.json](#manyplugjson)
    - [package.json](#packagejson)
    - [README.md](#readmemd)
    - [locale](#locale)
- [Publicando seu plugin](#publicando-seu-plugin)

---

## Exemplo mínimo

Antes de entrar nos detalhes, veja como é um plugin funcional do começo ao fim:

```js
// plugins/meu-plugin/index.js

export default async function (ctx) {
  const { msg } = ctx;

  // ignora tudo que não for o comando
  if (!msg.is("oi")) return;

  await msg.reply.text(`Olá, ${ctx.msg.senderName}!`);
}
```

É só isso. Um arquivo exportando uma função `default` que recebe `ctx` — o objeto com toda a
API do bot. Para entender o que mais está disponível no `ctx`, veja a
[Referência da API](/docs/api/).

---

## TypeScript

Você pode desenvolver plugins com **autocomplete completo** usando TypeScript ou JSDoc — o
ManyBot publica seu próprio pacote de tipos, `@manybot/types`, e o `manyplug init` já deixa
tudo configurado pra você.

### JavaScript com JSDoc (zero setup)

```js
/**
 * @param {import('@manybot/types/en').PluginContext} ctx
 */
export default async function (ctx) {
  // autocomplete funciona aqui ✅
  if (ctx.msg.is("oi")) {
    await ctx.msg.reply.text(`Olá, ${ctx.msg.senderName}!`);
  }
}
```

`manyplug init` já adiciona `@manybot/types` como `devDependency` no `package.json` gerado —
rode `npm install` no diretório do plugin e o autocomplete funciona direto, sem nenhum arquivo
de tipos local.

O types tem dois idiomas, use entre `@manybot/types/en` e `@manybot/types/pt`.

### TypeScript (recomendado para projetos maiores)

Peça pro `manyplug init` já montar a estrutura de TypeScript:

```bash
manyplug init meu-plugin --lang ts
```

Isso gera:

```
meu-plugin/
├── src/
│   └── index.ts
├── manyplug.json      # "main": "dist/index.js"
├── package.json       # devDependencies: typescript, @manybot/types
├── tsconfig.json
└── locale/
    ├── pt.json
    └── en.json
```

`src/index.ts` já vem com o tipo importado do pacote publicado:

```typescript
import type { PluginContext } from "@manybot/types";

export default async function (ctx: PluginContext) {
  if (!ctx.msg.is("oi")) return;

  // autocomplete completo do ctx e de todos os métodos ✅
  await ctx.msg.reply.text(`Olá, ${ctx.msg.senderName}!`);
}
```

> **Importante:** o ManyBot só carrega arquivos `.js` já compilados — ele **não** transpila
> TypeScript sozinho. Antes de instalar ou publicar, rode:
>
> ```bash
> npm install
> npm run build
> ```
>
> Isso compila `src/index.ts` para `dist/index.js`, que é o arquivo que `main` no
> `manyplug.json` aponta. Repita sempre que editar o código-fonte — e lembre de rodar o build
> antes de cada `manyplug install --local .` durante o desenvolvimento.

---

```
meu-plugin/
├── index.js
├── manyplug.json
├── package.json       # devDependencies: @manybot/types
├── README.md
├── .gitignore
└── locale/
    ├── en.json
    └── pt.json
```

> O comando `manyplug init` monta essa estrutura automaticamente — e sempre **pergunta o nome do
> autor** interativamente (usado pra montar `key`), além do idioma (`--lang`) se você não passar
> essa flag. Veja [manyplug init](/docs/manyplug-cli/#init).

---

## index.js

Arquivo principal do plugin — é aqui onde começa a execução.

Você exporta a função `default`, que o bot chama em cada mensagem recebida. Opcionalmente,
pode exportar também uma função `setup`, que roda uma única vez quando o bot conecta.

```js
// setup: executado uma vez quando o bot conecta (opcional)
export async function setup(ctx) {
  ctx.log.info("meu-plugin inicializado!");
}

// default: executado em cada mensagem recebida (obrigatório)
export default async function (ctx) {
  if (!ctx.msg.is("hello")) return;

  await ctx.msg.reply.text("Hello!");
}
```

Alguns pontos importantes:

- **Todos os plugins recebem todas as mensagens.** O bot não filtra por comando antes de
  chamar seu plugin — você mesmo decide se age ou ignora, geralmente com um `if (!msg.is(...)) return` no topo.
- **Plugins rodam em sequência.** Cada mensagem passa por todos os plugins ativos, um por um.
  Se o seu plugin lançar um erro não tratado, o kernel o desativa automaticamente para não
  quebrar os outros.
- **O bot também recebe as próprias mensagens.** Se o seu plugin responde a qualquer coisa
  (não só comandos), filtre `ctx.msg.fromMe` para não entrar em loop:

```js
export default async function (ctx) {
  if (ctx.msg.fromMe) return;

  // lógica do plugin...
}
```

---

## manyplug.json

Arquivo de metadados do plugin. Obrigatório para publicar no registro oficial, recomendado
mesmo em plugins privados.

```json
{
  "name": "meu-plugin",
  "key": "eu/meu-plugin",
  "version": "1.0.0",
  "description": "Meu plugin legal pro ManyBot.",
  "category": "utility",
  "manybotVersion": ">=5.0.0",
  "author": {
    "name": "eu",
    "email": "meu@email.com",
    "website": "www.meusite.com"
  },
  "license": "MIT",
  "repo": "https://github.com/eu/meu-plugin.many",
  "main": "index.js",
  "dependencies": {
    "outroautor/outro-plugin": "*"
  },
  "externalDependencies": {
    "ffmpeg": {
      "command": "ffmpeg",
      "optional": false
    }
  }
}
```

### Campos

#### `name` *(obrigatório)*
Nome do plugin. Letras minúsculas, números, pontos, hífens e underscores — precisa começar e
terminar com letra ou número. Deve ser único por autor. `manyplug init` já valida isso ao
perguntar o nome; `manyplug validate` aplica a mesma regra depois.

#### `version` *(obrigatório)*
Versão atual do plugin. Use o formato que preferir — SemVer, CalVer, tanto faz.

#### `category` *(obrigatório)*
Categoria do plugin. Valores possíveis:

| Valor         | Quando usar                                              |
|---------------|----------------------------------------------------------|
| `utility`     | Ferramentas de uso geral                                 |
| `media`       | Download, conversão ou envio de mídia                    |
| `games`       | Jogos e interações por turnos                            |
| `integration` | Integrações com APIs e serviços externos                 |
| `admin`       | Ferramentas de administração de grupos ou do próprio bot |
| `fun`         | Entretenimento sem categoria específica                  |
| `moderation`  | Anti-spam, filtros de conteúdo, moderação automática     |
| `ai`          | Integrações com modelos de IA/LLM                        |
| `education`   | Ferramentas educacionais, dicionários, tradutores        |
| `social`      | Interação social, perfis, rankings entre usuários        |
| `economy`     | Moedas virtuais, lojas, sistemas de pontos                |
| `automation`  | Automatizações e fluxos disparados por eventos            |
| `tools`       | Utilitários técnicos (ex: conversores, geradores)         |

#### `key`
Chave global no formato `autor/nome`, onde `nome` segue a mesma regra do campo `name` acima (e
precisa ser idêntico a ele). Usada para referenciar o plugin no `mpindex` e em dependências de
outros plugins. Sem ela, o ManyPlug instala em `manydev/<nome>` e o `validate` vai reclamar.

#### `manybotVersion`
String livre indicando a(s) versão(ões) do ManyBot com que o plugin é compatível (ex:
`">=5.0.0"`). Opcional — se ausente, `manyplug validate` não faz essa checagem. Se presente, o
`validate` compara com a versão do ManyBot instalada e avisa em caso de incompatibilidade ou se
não conseguir detectar uma instalação.

#### `description`
Descrição curta do que o plugin faz. Exibida na listagem do registro.

#### `author`
Informações do autor. Somente `name` é obrigatório. Aceita também uma string simples para compatibilidade.

#### `license`
[Licença de código aberto](https://www.freecodecamp.org/portuguese/news/como-funcionam-as-licencas-de-codigo-aberto-e-como-adiciona-las-a-seus-projetos-2/)
do plugin — define como o código pode ser distribuído e modificado por outros.

#### `repo`
Repositório Git do seu plugin. Deve ser apenas um e pode ser de qualquer forja e serviço (ex. GitHub, GitLab).
Não é obrigatório, serve apenas de identificação quando for publicado.

#### `main`
Nome do arquivo de entrada. Normalmente `"index.js"`. Se omitido, o ManyBot procura por `index.js`.

#### `dependencies`
**Não é para pacotes npm** — isso é o `package.json` (próxima seção). Esse campo lista **outros
plugins do ManyBot** que o seu usa via [`ctx.plugins.require()`](/docs/api/ctx-utilities/#ctxplugins):

```json
{
  "dependencies": {
    "outroautor/outro-plugin": "*"
  }
}
```

O `manyplug validate` escaneia seu código à procura de chamadas `ctx.plugins.require("chave")` e:
- Se encontrar uma chamada pra uma chave que não está listada aqui, **adiciona ela sozinho**
  nesse campo do seu `manyplug.json`.
- Se uma chave estiver listada aqui mas nunca for usada em `ctx.plugins.require()`, avisa que a
  dependência parece não utilizada.

O ManyPlug **não instala essas dependências automaticamente** — ele só avisa, no `install` e no
`validate`, se algum plugin listado aqui não estiver instalado. Cabe a quem instala rodar
`manyplug install` pra cada um.

> Antes de escolher uma dependência **npm** (que vai no `package.json`, não aqui), especialmente
> algo que compila código nativo (`sqlite3`, `bcrypt`, `sharp`, etc.), vale conferir as
> [boas práticas](/docs/best-practices#dependências-nativas--cuidado-especialmente-pensando-em-android) —
> essas costumam falhar pra instalar em quem roda o bot num Android via Termux.

#### `externalDependencies`
Programas externos que precisam estar instalados no sistema (ex: `yt-dlp`, `ffmpeg`):

```json
{
  "externalDependencies": {
    "yt-dlp": {
      "command": "yt-dlp",
      "optional": false
    },
    "ffmpeg": {
      "command": "ffmpeg",
      "optional": true
    }
  }
}
```

- `command` — comando usado para verificar se o programa está disponível no `PATH`
- `optional` — se `false` e o programa não for encontrado: `manyplug install` mostra um aviso mas
  instala mesmo assim; já `manyplug validate` trata como **erro** (bloqueia, encerra com código
  de saída 1). Se `true`, é sempre só um aviso em ambos os comandos.

---

## package.json

Arquivo mínimo. O único campo obrigatório é `"type": "module"` — necessário para que o
Node.js trate seus arquivos como ESM (o formato que o ManyBot usa):

```json
{
  "type": "module"
}
```

Se seu plugin tiver dependências **npm** (bibliotecas de verdade, não outros plugins — isso é o
`dependencies` do `manyplug.json`, seção anterior), declare elas aqui, do jeito normal do npm:

```json
{
  "type": "module",
  "dependencies": {
    "nome-do-pacote": "^1.0.0"
  }
}
```

O ManyPlug roda `npm install` nesse diretório automaticamente ao instalar seu plugin (e também
ao usar `manyplug link`) — quem instala não precisa fazer nada manualmente.

---

## README.md

Arquivo Markdown explicando seu plugin. Não é obrigatório, mas é altamente recomendado —
especialmente se você pretende publicar. É o que outros usuários vão ler para entender o que
seu plugin faz e como configurar.

Prefira escrever em inglês para alcançar mais pessoas, mas português também é aceito.

Exemplo real (plugin `manymedia`):

```markdown
# ManyMedia

Download videos and audio from YouTube, Reddit, Instagram, and other yt-dlp supported
sites — either sending the file directly to chat or uploading to a storage server and
sharing the link.

## Features

- **Multi-site support**: YouTube, Reddit, Instagram, SoundCloud, TikTok, and any
  other yt-dlp compatible site
- **Audio extraction**: Downloads and extracts MP3 at best quality
- **Flexible delivery**: Send file directly to chat, or upload to a storage server
  and reply with the link
- **Upload retry**: Failed uploads are retried up to 4 times with a 3-second delay
- **Queued processing**: Downloads run in a queue to prevent resource contention
- **Automatic cleanup**: Temporary files removed after delivery

## Requirements

- `yt-dlp` installed and available in `PATH`
- `ffmpeg` for converting filetypes (e.g. mp4 to mp3) — optional for download only
- `cookies.txt` in the project root (required for YouTube, Reddit, and sites that
  need authentication)

## Usage

    /video https://youtube.com/watch?v=...
    /audio https://youtube.com/watch?v=...

## Configuration

Add to `manybot.toml`:

| Key                | Default | Description                                                    |
|--------------------|---------|----------------------------------------------------------------|
| `UPL_MEDIA_TO_SRV` | `no`    | Set to `yes` to upload to a server and reply with a link       |
| `MEDIA_SRV_API_KEY`| —       | API key for the storage server (required when `UPL_MEDIA_TO_SRV=yes`) |
```

---

## locale

Diretório com as traduções do seu plugin. O ManyBot carrega automaticamente o arquivo
correspondente ao idioma configurado no `manybot.toml`.

Estrutura:

```
locale/
├── en.json
├── pt.json
└── es.json
```

`locale/en.json`:
```json
{
  "hello": "Hello, {{name}}!",
  "error": {
    "generic": "Something went wrong. Try again."
  }
}
```

`locale/pt.json`:
```json
{
  "hello": "Olá, {{name}}!",
  "error": {
    "generic": "Algo deu errado. Tente novamente."
  }
}
```

E no código:

```js
export default async function (ctx) {
  const prefix = ctx.config.get("CMD_PREFIX");
  const { t }  = ctx.i18n.createT(import.meta.url);

  if (!ctx.msg.is(prefix + "hello")) return;

  // {{name}} é substituído pelo valor passado no segundo argumento
  await ctx.msg.reply.text(t("hello", { name: ctx.msg.senderName }));
}
```

> Locales não são obrigatórios, mas são incentivados. Sem locale, o plugin simplesmente não oferece suporte a múltiplos idiomas.

Para mais detalhes sobre a API de i18n, veja [ctx.i18n](/docs/api/ctx-utilities/#ctxi18n).

---

## Publicando seu plugin

Aqui é somente se quiser que outras pessoas usem seu plugin a partir do índice oficial.
Totalmente dispensável.

### 1. Valide e teste localmente

Antes de qualquer coisa, rode o validador e instale localmente:

```bash
manyplug validate .
manyplug install --local .
```

O `validate` checa bem mais do que campos obrigatórios: tipos, entry point, `manybotVersion`,
locale, dependências npm e de outros plugins (e chega a **completar sozinho** o campo
`dependencies` se detectar `ctx.plugins.require()` no seu código), dependências externas, e até
uso incorreto do `ctx` no seu código. Veja a lista completa em
[manyplug validate](/docs/manyplug-cli/#validate). Corrija tudo que ele apontar como erro antes
de continuar — avisos não bloqueiam, mas vale revisar. Depois confirme que o plugin funciona
rodando o bot normalmente.

### 2. Crie um repositório Git

Suba o código para o GitHub, Codeberg, GitLab ou outra forja de sua preferência.

A convenção (não obrigatória, mas incentivada) é adicionar o sufixo `.many` ao nome do
repositório — ex: `https://codeberg.org/usuario/meu-plugin.many`. Serve só para identificação.

### 3. Envie uma solicitação

Mande um email para [manybot@pm.me](mailto:manybot@pm.me) com o seguinte formato:

```
Assunto: [PLUGIN-REQUEST] Nome do seu plugin

# O que ele faz?

Descrição resumida do plugin.

# Repositório(s)

GitHub:   https://github.com/.../...
Codeberg: https://codeberg.org/.../...
```

O email pode ser em inglês ou português. Certifique-se de ter um README explicativo no
repositório — é o que vai ser lido durante a revisão.

> Caso não queira enviar via email, é possível enviar na nossa comunidade do
[Discord](https://discord.com/invite/gC7aKChXmA) ou [WhatsApp](https://chat.whatsapp.com/KfOuIwhpQjN8fcZTMHmaGQ).

### 4. Revisão e publicação

Após a revisão, você receberá um email com:

- Se o plugin foi aceito ou não, e o motivo
- Feedback sobre o código ou documentação
- Qualquer outra informação relevante

O atendimento é 100% humano e anônimo. Sem medo de perguntar.

Se aceito, o plugin entra no índice oficial (`mpindex`) e fica disponível para instalação
via `manyplug install autor/nome`:

```json
"eu/meu-plugin": {
  "repos": {
    "codeberg": {
      "master": "https://codeberg.org/eu/meu-plugin.many",
      "dev":    "https://codeberg.org/eu/meu-plugin.many"
    },
    "github": {
      "master": "https://github.com/eu/meu-plugin.many",
      "dev":    "https://github.com/eu/meu-plugin.many"
    }
  },
  "manifest": "https://raw.githubusercontent.com/eu/meu-plugin.many/refs/heads/master/manyplug.json",
  "readme": "https://raw.githubusercontent.com/eu/meu-plugin.many/refs/heads/master/README.md"
}
```
