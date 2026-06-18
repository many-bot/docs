# Como fazer um plugin

Aqui você vai aprender tudo que precisa para começar a desenvolver plugins para o ManyBot.

Caso ainda não leu [sobre os plugins](/docs/about-plugins), é recomendado ler antes de continuar.

## Índice

- [Exemplo mínimo](#exemplo-minimo)
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
  const prefix = ctx.config.get("CMD_PREFIX");

  // ignora tudo que não for o comando
  if (!ctx.msg.is(prefix + "oi")) return;

  await ctx.msg.reply(`Olá, ${ctx.msg.senderName}!`);
}
```

É só isso. Um arquivo exportando uma função `default` que recebe `ctx` — o objeto com toda a
API do bot. Para entender o que mais está disponível no `ctx`, veja a
[Referência da API](/docs/api-reference).

---

## Estrutura padrão de um plugin

```
meu-plugin/
├── index.js
├── manyplug.json
├── package.json
├── README.md
└── locale/
    ├── en.json
    ├── es.json
    └── pt.json
```

> O comando `manyplug init` monta essa estrutura automaticamente. Veja [manyplug init](/docs/manyplug-cli/#init).

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
  const prefix = ctx.config.get("CMD_PREFIX");

  if (!ctx.msg.is(prefix + "hello")) return;

  await ctx.msg.reply("Hello!");
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
  "author": {
    "name": "eu",
    "email": "meu@email.com",
    "website": "www.meusite.com"
  },
  "description": "Meu plugin legal pro ManyBot.",
  "version": "1.0.0",
  "license": "MIT",
  "category": "utility",
  "service": false,
  "main": "index.js",
  "dependencies": {
    "dependencia-npm": ">=10",
    "synt-xerror/manymedia": ">=3.2.0"
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

#### `name`
Nome do plugin. Usado como identificador legível e no registro. Deve ser único por autor.

#### `key`
Chave global no formato `autor/nome`. Usada para referenciar o plugin no `mpindex` e em
dependências de outros plugins.

#### `author`
Informações do autor. Somente `name` é obrigatório.

#### `description`
Descrição curta do que o plugin faz. Exibida na listagem do registro.

#### `version`
Versão atual do plugin. Use o formato que preferir (SemVer, CalVer, etc).

#### `license`
[Licença de código aberto](https://www.freecodecamp.org/portuguese/news/como-funcionam-as-licencas-de-codigo-aberto-e-como-adiciona-las-a-seus-projetos-2/)
do plugin — define como o código pode ser distribuído e modificado por outros.

#### `category`
Categoria do plugin. Valores possíveis:

| Valor         | Quando usar                                              |
|---------------|----------------------------------------------------------|
| `utility`     | Ferramentas de uso geral                                 |
| `media`       | Download, conversão ou envio de mídia                    |
| `games`       | Jogos e interações por turnos                            |
| `integration` | Integrações com APIs e serviços externos                 |
| `service`     | Plugins que rodam em background sem comandos diretos     |
| `admin`       | Ferramentas de administração de grupos ou do próprio bot |
| `fun`         | Entretenimento sem categoria específica                  |

#### `service`
`true` se o plugin roda em background como um serviço (sem comandos diretos, ex: um monitor
que envia alertas periódicos). `false` se é acionado por comandos do usuário.

#### `main`
Nome do arquivo de entrada. Normalmente `"index.js"`.

#### `dependencies`
Pacotes npm necessários ou plugins do ManyBot. O ManyPlug os instala automaticamente. Formato:
```json
{
  "dependencies": {
    "nome-do-pacote": ">=1.0.0"
  }
}
```

#### `externalDependencies`
Programas externos que precisam estar instalados no sistema (ex: `yt-dlp`, `ffmpeg`).

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
- `optional` — se `false`, o plugin não carrega sem esse programa; se `true`, degrada
  graciosamente (pode funcionar parcialmente sem ele)

---

## package.json

Arquivo mínimo. O único campo obrigatório é `"type": "module"` — necessário para que o
Node.js trate seus arquivos como ESM (o formato que o ManyBot usa):

```json
{
  "type": "module"
}
```

Se seu plugin tiver dependências npm, elas também aparecem aqui — mas o ManyPlug gerencia isso
para você automaticamente a partir do `manyplug.json`.

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

Add to `manybot.conf`:

| Key                | Default | Description                                                    |
|--------------------|---------|----------------------------------------------------------------|
| `UPL_MEDIA_TO_SRV` | `no`    | Set to `yes` to upload to a server and reply with a link       |
| `MEDIA_SRV_API_KEY`| —       | API key for the storage server (required when `UPL_MEDIA_TO_SRV=yes`) |
```

---

## locale

Diretório com as traduções do seu plugin. O ManyBot carrega automaticamente o arquivo
correspondente ao idioma configurado no `.conf`.

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
  await ctx.msg.reply(t("hello", { name: ctx.msg.senderName }));
}
```

> Locales **não** são obrigatórios, servem apenas para a tradução do seu plugin. Porém, são incentivados.

Para mais detalhes sobre a API de i18n (internacionalization), veja [ctx.i18n](/docs/api-reference#ctxi18n).

---

## Publicando seu plugin

Aqui é somente se quiser que outras pessoas usem seu plugin além de você, a partir do índice oficial.
Totalmente dispensável.

### 1. Teste localmente

Antes de qualquer coisa, instale e teste:

```bash
manyplug install --local /caminho/do/seu/plugin
```

O ManyPlug instala o plugin no lugar correto. Execute o bot e confirme que tudo funciona
como esperado antes de prosseguir.

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

### 4. Revisão e publicação

Após a revisão, você receberá um email com:

- Se o plugin foi aceito ou não, e o motivo
- Feedback sobre o código ou documentação
- Qualquer outra informação relevante

O atendimento é 100% humano. Sem medo de perguntar.

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
